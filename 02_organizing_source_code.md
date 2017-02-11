# Organizing your source code

There are 2 stages to database design:  pre-production and maintenance mode.  In pre-production, everything is ephemeral.  You're free to drop the database whenever you please so you can recreate it and populate it with a fresh set of test data with no real consequences.  In maintenance mode, the schema must be modified and data migrated without losing any.  So the goal here is to write your pre-production code in such a way that the reusable parts can be reused during the migration process.

There are 2 categories of objects we're concerned about:  replaceable and irreplaceable.

## Irreplaceable (or Data) objects

Irreplaceable objects are objects that result in data loss when you drop them and cannot be easily restored.  Such objects include:

* Tables
* Types (domains, composite types, etc.)

Watch what happens when we drop a domain.  No amount of `CREATE DOMAIN label AS TEXT` is going to restore the database to the state it was in before we ran our command.

```
CREATE DOMAIN label AS TEXT;

CREATE TEMPORARY TABLE foo (
	foo_id INT NOT NULL,
	title LABEL NOT NULL,
	PRIMARY KEY (foo_id)
);

/*
test=> \d foo
      Table "pg_temp_2.foo"
 Column |  Type   | Modifiers
--------+---------+-----------
 foo_id | integer | not null
 title  | label   | not null
Indexes:
    "foo_pkey" PRIMARY KEY, btree (foo_id)
*/

DROP DOMAIN label CASCADE;

/*
NOTICE:  drop cascades to table foo column title
DROP DOMAIN
test=> \d foo
      Table "pg_temp_2.foo"
 Column |  Type   | Modifiers
--------+---------+-----------
 foo_id | integer | not null
Indexes:
    "foo_pkey" PRIMARY KEY, btree (foo_id)
*/
```

Once your database is in production, modifications will almost always be done using the ALTER command.  You'll never run your setup code for again for these objects outside of a testing environment.

## Replaceable (or Derived) objects

Replaceable objects are objects that can be casually replaced by the same code that created them in the first place.  Many of them can be replaced via `CREATE OR REPLACE...`.  Such objects include:

* Triggers
* Functions
* Views
* Materialized Views
* Indexes
* Constraints

Some objects like functions and views require being dropped first, in cases where the names or types of columns/arguments has changed.

## Setup vs. Migration

The basic idea here is that you want to organize your source code in such a way that you can reuse the code for replaceable objects in both the setup and migration.

I have one project that relies heavily on views, triggers, and functions.  Out of the 13 migration files I've written so far, 9 of them drop the same view, which cascades to at least 3 other views.  Rather than include the code for creating the same objects in every single migration script, include them at the end.

Setup:

```sql
\set ON_ERROR_STOP true

BEGIN;

-- irreplaceable
\ir schema/tables.sql

-- replaceable
\ir schema/triggers.sql
\ir schema/functions.sql
\ir schema/views.sql
```

Migration:

```sql
\set ON_ERROR_STOP true

BEGIN;

-- irreplaceable
\ir migration/01_fix_column_type.sql
\ir migration/02_convert_view_to_table.sql

-- replaceable
\ir schema/triggers.sql
\ir schema/functions.sql
\ir schema/views.sql
```
