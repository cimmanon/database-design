# Organizing your source code

There are 2 stages to database development:  pre-production and maintenance mode.  In pre-production, everything is ephemeral.  You're free to drop the database whenever you please so you can recreate it and populate it with a fresh set of test data with no real consequences.  In maintenance mode, the schema must be modified and data migrated without losing any.  So the goal here is to write your pre-production code in such a way that we maximize the amount of code we can reuse during the migration process once we go into maintenance mode.

There are 2 categories of objects we're concerned about:  replaceable and irreplaceable.

## Irreplaceable (or Data) objects

Irreplaceable objects are objects that result in data loss when you drop them and cannot be easily restored.  Such objects include:

* Tables
* Types (domains, composite types, etc.)

Watch what happens when we drop a domain.  No amount of `CREATE DOMAIN label AS TEXT` is going to restore the database to the state it was in before we ran our command.

```sql
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
