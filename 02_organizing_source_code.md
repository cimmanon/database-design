# Organizing your source code

There are 2 stages to database design:  pre-production and maintenance mode.  In pre-production, everything is ephemeral.  You're free to drop the database whenever you please so you can recreate it and populate it with a fresh set of test data with no real consequences.  In maintenance mode, the schema must be modified and data migrated without losing any.  So the goal here is to write your pre-production code in such a way that the reusable parts can be reused during the migration process.

There are 2 categories of objects we're concerned about:  replaceable and irreplaceable.

## Irreplaceable objects

Irreplaceable objects are objects that result in data loss when you drop them and cannot be easily restored.  Such objects include:

* Tables
* Types (domains, composite types, etc.)

Once your database is in production, these objects will primarily be modified using the ALTER command.  You'll never your setup code again outside of a testing environment.

## Replaceable objects

Replaceable objects are objects that can be casually replaced whenever.  Many of them can be replaced via `CREATE OR REPLACE...`.  Such objects include:

* Triggers
* Functions
* Views
* Materialized Views
* Indexes
* Constraints

Some objects like functions and views require being dropped first in cases where the names or types of columns/arguments has changed.

## Setup vs. Migration

The basic idea here is that you want to organize your source code in such a way that you can reuse the code for replaceable objects in both the setup and migration.  Trying to include code to recreate views and all objects that depend on it every time something small changes in each migration file becomes quite unwieldy over time.

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
