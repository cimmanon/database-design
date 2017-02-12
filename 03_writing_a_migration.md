# Writing a migration

There are a few basic rules that should be kept in mind when writing a migration.

* Use feature detection to control whether the migration should be run
* Modify irreplaceable objects and drop replaceable objects that cannot be overwritten by a `CREATE OR REPLACE` command
* Avoid depending on views or other replaceable objects, they might not exist at the time of the migration (due to being dropped by a previous migration script)

## Feature detection vs. versioning

A lot of people advocate some method of indicating what version of the database is being run in order to determine what migration scripts need to be run.  This seems like a good idea on the surface, but it is something that can quickly fall out of synch with reality.  If it's data stored in a table, someone could modify it outside of a migration or accidentally forget to update the version information during a migration.  If it's data stored in a flat file, someone could delete it.

Before I started working on databases, I was a web designer.  One of the biggest sins you can commit is sniffing the user agent string for version or browser name.  Browsers regularly lie about what version they are or what their name is and it ends up being not very future proof at all.  Instead, you should use feature detection.  Does the feature you want to use exist?  If so, continue with your code, otherwise provide a fallback or helpful error.

Every database should have a way to check the structure of your objects using SQL.  Want to add a new table?  Check to see if it exists first.  Changing a column type?  Check to see if it has already been changed.

```sql
CREATE TEMPORARY TABLE foo (
	foo_id INT NOT NULL,
	title VARCHAR(150),
	PRIMARY KEY (foo_id)
);

DO $DO$
BEGIN
	IF EXISTS (
		SELECT 1 FROM information_schema.columns
		WHERE
			table_name = 'foo'
			AND column_name = 'title'
			AND data_type = 'character varying'
		)
	THEN

		RAISE NOTICE 'Changing foo.title from a VARCHAR to a TEXT';
		ALTER TABLE foo ALTER title TYPE TEXT;

		-- perform other changes related to altering the column type

	ELSE
		RAISE NOTICE 'The foo.title column has already been converted to a TEXT, skipping...';
	END IF;
END;
$DO$ LANGUAGE plpgsql;
```

In the above code, running the DO block, you get the following result:

```
NOTICE:  Changing foo.title from a VARCHAR to a TEXT
DO
```

If you run the DO block a second time, you get this instead:

```
NOTICE:  The foo.title column has already been converted to a TEXT, skipping...
DO
```

All of the information necessary for the migration to run exists in a single location:  your migration script.

## Drop incompatible replaceable objects

Sometimes, modifying a replaceable object cannot be done via `CREATE OR REPLACE`.

```sql
CREATE OR REPLACE TEMPORARY VIEW foo AS
SELECT
	'foo' :: text as foo,
	'bar' :: text as bar
;

CREATE OR REPLACE TEMPORARY VIEW foo AS
SELECT
	'foo' :: text as foo,
	'bar' :: text as baz
;

/*
ERROR:  cannot change name of view column "bar" to "baz"
*/
```

In these cases, you must drop the old object before creating a new one.  You can either do this by dropping only the objects that need to be replaced or by dropping them all.  Either way, all of the replaceable objects are being rebuilt at the end.  Personally, I like the self-documenting nature of dropping exactly the ones you want, but dropping them all up front is pretty easy to script.

## Avoid depending on replaceable objects

Consider the following scenario:

1. Migration #1 is written and deployed
2. Migration #2 is written and deployed on a database that is already on version #1
3. Migration #1 and #2 is run back to back on a database that was still on version #0

Here's what our migration file looks like at the time Migration #1 is deployed:

```sql
BEGIN;

-- drop a view so we can rename it
\ir migration/01_rename_column_in_view.sql

-- rebuild our replaceable objects
\ir schema/views.sql
```

Here's what our migration file looks like at the time Migration #2 is deployed:

```sql
BEGIN;

-- drop a view so we can rename it
\ir migration/01_rename_column_in_view.sql
-- migrate data using a query that selects from a dropped query
\ir migration/02_stuff_that_relies_on_view.sql

-- rebuild our replaceable objects
\ir schema/views.sql
```

By using feature detection, the changes in #1 are skipped (ie. the view is not dropped) on databases running version #1.  On databases still on version #0, Migration #1 succeeds in dropping the view and Migration #2 fails because the object it needs doesn't exist.

If you really need to use replaceable objects as part of your migration, make sure they exist before you do so.
