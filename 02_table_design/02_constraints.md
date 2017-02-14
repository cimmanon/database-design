# Table Design: Writing Constraints

Give your constraints explicit names.  If you allow your database to generate the names automatically, you have 3 potential problems:

* The error message is not very helpful, especially in the case of check constraint violations
* The constraint names might not be consistent across database instances, especially in cases where tables or columns are renamed after deployment
* It becomes difficult to target the correct constraint when it needs to be modified later

```sql
CREATE TEMPORARY TABLE foo (
	foo TEXT NOT NULL,

	PRIMARY KEY (foo),
	CHECK (length(foo) > 1),
	CHECK (length(foo) < 100)
);

/*
test=> \d foo
   Table "pg_temp_2.foo"
 Column | Type | Modifiers
--------+------+-----------
 foo    | text | not null
Indexes:
    "foo_pkey" PRIMARY KEY, btree (foo)
Check constraints:
    "foo_foo_check" CHECK (length(foo) > 1)
    "foo_foo_check1" CHECK (length(foo) < 100)

test=> insert into foo values ('x');
ERROR:  new row for relation "foo" violates check constraint "foo_foo_check"
DETAIL:  Failing row contains (x).
*/
```

Compare that to explicitly named constraints:

```sql
CREATE TEMPORARY TABLE foo (
	foo TEXT NOT NULL,

	PRIMARY KEY (foo),
	CONSTRAINT foo_foo_min_length_ck CHECK (length(foo) > 1),
	-- ^ Sublime Text snipppet: ${1:table name}_${2:column name}_${3:description}_ck
	CONSTRAINT foo_foo_max_length_ck CHECK (length(foo) < 100)
);

/*
test=> \d foo
   Table "pg_temp_2.foo"
 Column | Type | Modifiers
--------+------+-----------
 foo    | text | not null
Indexes:
    "foo_pkey" PRIMARY KEY, btree (foo)
Check constraints:
    "foo_foo_max_length_ck" CHECK (length(foo) < 100)
    "foo_foo_min_length_ck" CHECK (length(foo) > 1)

test=> insert into foo values ('x');
ERROR:  new row for relation "foo" violates check constraint "foo_foo_min_length_ck"
DETAIL:  Failing row contains (x).
*/
```
