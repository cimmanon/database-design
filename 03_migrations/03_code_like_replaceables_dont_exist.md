## Migration: Code Like Replaceable Objects Don't Exist

Consider the following scenario:

1. Migration #1 is written and deployed on database version #0
2. Migration #2 is written and deployed on database version #1
3. Migration #1 and #2 is run back to back on database version #0

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

If you really need to use replaceable objects as part of your migration, add them as part of your migration file.
