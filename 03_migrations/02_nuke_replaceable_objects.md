## Migrations: Nuke Replaceable Objects Without Regret

Replaceable objects are exactly that:  objects that can be dropped and recreated at any time without resulting in lost data.  There are plenty of cases, such as renaming a column in a view, where modifying a replaceable object cannot be done via `CREATE OR REPLACE`.

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

In these cases, you must drop the old object before creating a new one.  You can either do this by dropping only the objects that need to be replaced or by dropping them all.  Either way, all of the replaceable objects are being rebuilt at the end.  Personally, I like the self-documenting nature of dropping exactly the ones you want, but dropping them all up front is pretty easy to script and make part of your regular migration process.

These objects will be restored at the end of the migration process, so they're not gone forever.
