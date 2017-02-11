# Table naming convetions

There are 2 basic camps when it comes to naming tables:  singular or plural.  Which should you use?  Whichever one you like best!  There are compelling reasons for both choices.

## A case for Singular table names

When PostgreSQL creates a table, it also creates a composite type of the same name.  It seems silly to have a type of FOOS that represents a singular item.

```sql
CREATE TEMPORARY TABLE foo (
	foo TEXT NOT NULL,
	bar TEXT,
	PRIMARY KEY (foo)
);

SELECT (('one', 'two') :: FOO).foo;

/*
 foo
-----
 one
(1 row)
*/
```

## A case for Plural table names

A table represents a collection of objects, so it seems natural for the collection (ie. the table) to be plural.  By using the plural version for the table name, a type that uses the singular version can be created later to use as a function's argument type.

```sql
-- TODO: add example
```

# Column naming conventions

I've changed my opinion about column names throughout the years.  I used to hold the opinion that tables were islands and each column should be named with the assumption that it belongs to the table.  I guess you could call it "object oriented naming".  While this is more attractive for programmers who are used to OOP, it ends up being quite verbose and unnatural once it comes time to write queries.

```sql
CREATE TEMPORARY TABLE foo (
	id SERIAL NOT NULL,
	title TEXT NOT NULL,
	PRIMARY KEY (id),
	UNIQUE (title)
);

CREATE TEMPORARY TABLE bar (
	foo_id INT NOT NULL,
	category TEXT NOT NULL,
	PRIMARY KEY (foo_id, category),
	FOREIGN KEY (foo_id) REFERENCES foo (id)
);

SELECT
	foo.id AS foo_id,
	foo.title,
	bar.category
FROM
	bar
	JOIN foo ON foo.id = bar.foo_id;
```

Instead, I follow the convention that columns used for FOREIGN KEYs should be unique across all tables that could potentially be used in the same join.  This makes it a little bit easier to figure out which columns can be joined together at a glance (ie. you don't have to examine the table definition) and less verbose when it comes to performing joins.  The following example only seems to save a few keystrokes, but the difference is much more noticable for each column that's added to the join.

```sql
CREATE TEMPORARY TABLE foo (
	foo_id SERIAL NOT NULL,
	title TEXT NOT NULL,
	PRIMARY KEY (foo_id),
	UNIQUE (title)
);

CREATE TEMPORARY TABLE bar (
	foo_id INT NOT NULL,
	category TEXT NOT NULL,
	PRIMARY KEY (foo_id, category),
	FOREIGN KEY (foo_id) REFERENCES foo (foo_id)
);

SELECT
	foo_id,
	foo.title,
	bar.category
FROM
	bar
	JOIN foo USING (foo_id);
```

Some people like to prefix all of the column names that could potentially be ambiguous (eg. foo_id, foo_name, etc.), but it's a bit verbose for my tastes.
