# Writing a migration

There are a few basic rules that should be kept in mind when writing a migration.

* Use feature detection to control whether the migration should be run
* Only modify components of irreplaceable objects and drop replaceable objects that cannot be overwritten by a `CREATE OR REPLACE` command
* Avoid depending on views or other replaceable objects, they might not exist at the time of the migration (due to being dropped by a previous migration script)

## Feature detection vs. versioning

A lot of people advocate some method of indicating what version of the database is being run in order to determine what migration scripts need to be run.  This seems like a good idea on the surface, but it is something that can quickly fall out of synch with reality.  If it's data stored in a table, someone could modify it outside of a migration or accidentally forget to update the version information during a migration.  If it's data stored in a flat file, someone could delete it.

Before I started working on databases, I was a web designer.  The golden rule of web design is that you always use feature detection rather than version checking.  Browsers regularly lie about what version they are or what their name is and it ends up being not very future proof at all.

In most databases, there's a way to examine the structure of all of your objects.  Want to add a new table?  Check to see if it exists first.  Changing a column type?  Check to see if it has already been changed.

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

## Drop incompatible irreplacable objects

Explain more...

## Don't depend on irreplaceable objects

