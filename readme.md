This is a work in progress.

# What is this?

This is a collection of thoughts based on my own experiences designing and maintaining databases.  I'm going to assume that you're already familiar with the Relational Model and understand how things like Foreign Key constraints work.  I'm also going to assume that you're using a database that has fully formed features like transactions and check constraints (if you're still using MySQL, you're doing yourself and your customers a serious disservice).

All examples are for PostgreSQL.  If you're using something else, you'll need to check with the documentation for comparable features.

# Table of contents

1. Organizing Source Code
	1. Replaceable vs. Irreplaceable Objects
	2. Migration Ready Source
2. Table Design
	1. Naming Conventions
	2. Constraints
3. Migrations
	1. Feature Detection
	2. Nuke Replaceable Objects Without Regret
	3. Code Like Replaceable Objects Don't Exist

Unconfirmed topics:

* Git Versioning
* Triggers
* Functions
* Large scale redesigns (ie. that last guy was such a bonehead)
* Migrating from other RDBMS
* Foreign Data Wrappers (FDW)

# Recommended reading

* [Use the Index, Luke](http://use-the-index-luke.com/): a fantastic resource on how to write and use indexes
* [Modern SQL](http://modern-sql.com/): beyond SQL-92 (not for MySQL users!)
* [The Database Programmer](http://database-programmer.blogspot.ca/2008/09/comprehensive-table-of-contents.html): advanced articles on database design and lover of the "Less Code, More Data" mantra
