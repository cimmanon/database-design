## Setup vs. Migration

The basic idea here is that you want to organize your source code in such a way that you can reuse the code for replaceable objects in both the setup and migration.

I have one project that relies heavily on views, triggers, and functions.  Out of the 13 migration files I've written so far, 9 of them drop the same view, which cascades to at least 3 other views.  Rather than include the code for creating the same objects in every single migration script, create the final versions of them at the very end.

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
