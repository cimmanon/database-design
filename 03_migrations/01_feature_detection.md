# Migrations: Feature detection

A lot of people advocate some method of indicating what version of the database is being run in order to determine what migration scripts need to be run.  This seems like a good idea on the surface, but it is something that can quickly fall out of synch with reality.  If it's data stored in a table, someone could modify it outside of a migration or accidentally forget to update the version information during a migration.  If it's data stored in a flat file, someone could delete it.

Before I started working on databases, I was a web designer.  One of the biggest sins you can commit is sniffing the user agent string for version or browser name in order to work around bugs or API differences.  Browsers regularly lie about what version they are or what their name is and it ends up being not very future proof at all.  Instead, you should use feature detection.  Does the feature you want to use exist?  If so, continue with your code, otherwise provide a fallback or helpful error.

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

/*
NOTICE:  Changing foo.title from a VARCHAR to a TEXT
DO
*/
```

If you run the DO block a second time, you get this instead:

```
NOTICE:  The foo.title column has already been converted to a TEXT, skipping...
DO
```

All of the information necessary for the migration to run exists in a single location:  your migration script.

## Feature Detection Isn't Perfectly Future Proof

One thing that you must keep in mind is that a condition that effectively gated a migration script from running in the past is not guaranteed to be effective in the future.

* Migration #1 converts view foo_the_bar to a table
* Migration #2 renames table foo_the_bar to bar_the_foo

If Migration #1 is only looking for the existence of the table foo_the_bar, and not bar_the_foo, you'll end up accidentally running Migration #1 again once it comes time to deploy the not-yet-written Migration #3.  Looking for the existance of the view foo_the_bar is not considered reliable for reasons we'll get to later.  If Migration #2 is looking for the existence of the table bar_the_foo, it will not run during Migration #3 which causes you to have an extra table hanging around for no reason.

Test your migrations diligently to ensure only the ones that are supposed to run *do* run.  Running a migration extra times might seem harmless if it's just extra tables or columns hanging around, but data migrations can cause the script to take more time to run than necessary (and may result in data loss or duplication!).
