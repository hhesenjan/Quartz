---
title: "The Basics Of SQL"
created: 2024-09-25
modified: 2024-09-25
tags: ["CPP", "MySQL"]
draft: false
---

## Introduction

SQL is the standard language for managing databases and the records inside of the databases. It's widely used by organizations to manage and automate database setups, table setups, and inserting data into the tables. The data inside of the databases are commonly used for authentication, authorization, and showing specific data from the database to the website or the application.

## What is Database, Tables, and Records? 

```text
    Databases                 Tables                         Colums
+--------------+           +----------+           +----+----------+----------+
| husenjanprod | --------> | users    | --------> | id | username | password |
+--------------+           +----------+           +----+----------+----------+
| husenjanpdev |           | settings |           | 1  | olav     | hashed   |
+--------------+           +----------+           +----+----------+----------+
       |                   | gifts    |           | 2  | johndoe  | hashed   |
       |                   +----------+           +----+----------+----------+
       |                      Tables                        Columns
	   |                   +----------+           +----+----------+----------+
       +-----------------> | users    | --------> | id | username | password | 
                           +----------+           +----+----------+----------+
                           | settings |           | 1  | testusr  | hashed   |
                           +----------+           +----+----------+----------+
                           | gifts    |
						   +----------+
```

A single database is used for one specific thing, this can be for production or our test environment. Inside a database there are tables which are used to organize the data by users, groups, and gifits. An example of this is the databases `husenjanprod` and `husenjandev` one is used for production and the other one is used for our test environment. Note that it's also possible to create relationships between a column from a table with another column from a different table *(This is called for SQL Relationship)*.

## Working with SQL

In this section of the document, I'll go through the different SQL commands which are useful to know while working with a database. I highly recommend studying the commands and memorizing it as it will become useful when you need to work with an database. 

To see all the databases:

```sql
SHOW DATABASES;
```

To crease an database:

```sql
CREATE DATABASE IF NOT EXISTS mydb;
```

To select an database:

```sql
USE mydb;
```

To list all our tables inside the database:

```sql
SHOW TABLES;
```

To create an table with numbers and characters:

```sql
CREATE TABLE IF NOT EXISTS users(
	id int PRIMARY KEY, -- Integer Column
	firstname varchar(255), -- Character Column
	lastname varchar(255) -- Character Column
);
```

To insert data inside the table created:

```sql
INSERT INTO users(id, firstname, lastname) values (1, 'Husen123', 'Hesenjan123');
```

To change the data inside of the table:

```sql
UPDATE users firstname = 'Husenjan', lastname = 'Hesenjan' WHERE id = 1;
```

The commands shown might seem complicated for that reason I highly recommend playing around with it in an lab environment to better understand SQL before you continue reading the document. 

## SQL Relationships

```text
                                                    +-----------------------------+
    Databases                 Tables                |         Colums              |
+--------------+           +----------+           +-*--+----------+----------+    |
| husenjanprod | --------> | users    | --------> | id | username | password |    |
+--------------+           +----------+           +----+----------+----------+    |
| husenjanpdev |           | settings |           | 1  | husenjan | secret   |    | Interconnected
+--------------+           +----------+           +----+----------+----------+    |
                           | gifts    | ----+     | 2  | testusr  | secret   |    |
                           +----------+     |     +----+----------+----------+    |
						                    |                                     |
                                            |                +--------------------+
                                            |                |
                                            |     +----+-----*---+-----------+
                                            +---> | id | user_id | gift      |
                                                  +----+---------+-----------+
                                                  | 1  | 1       | iPhone 16 |
                                                  +----+---------+-----------+
```

SQL Relationships are used for linking two columns from different tables to each other, this allows us to see which item belogs to a specific user or a object. An example is connecting `id` column from `users` table to `user_id` column inside the `gifts` table. You can use the following SQL commands to create a SQL Relation-ship shown: 

```sql
CREATE TABLE IF NOT EXISTS users(
	id int PRIMARY KEY,
	username varchar(255),
	password varchar(255) 
);

CREATE TABLE IF NOT EXISTS gifts(
	id int PRIMARY KEY,
	user_id int,
	gift varchar(2555),
	FOREIGN KEY (user_id) REFERENCES users(id)
);

INSERT INTO users(id, username, password) values (1, 'Husenjan', 'secret');

INSERT INTO gifts(id, user_id, gift) values (1, 1, 'Husenjan', 'iPhone 16');
```

Note that a SQL Relationship can only be created with columns that are defined `PRIMARY KEY` or `UNIQUE` otherwise the database will throw an error message. It's also possible for us to create One-To-One Relationships, Many-To-Many Relationships, and Many-To-Many Relationships. When two columns are referenced to each other the following command can be used to query results from different tables:

```sql
SELECT users.id, users.username, gifts.gift FROM users INNER JOIN gifts ON users.id = gifts.user_id;
```

## Creating SQL Scripts

We can also create an SQL Script by creating a `tutorial.sql` file which contains all our SQL commands and once it's executed all the commands inside the file will be automatically executed for us.

```sql title="tutorial.sql"
DROP IF EXISTS users;
DROP IF EXISTS gifts;

CREATE TABLE IF NOT EXISTS users(
	id int PRIMARY KEY,
	username varchar(255),
	password varchar(255) 
);

CREATE TABLE IF NOT EXISTS gifts(
	id int PRIMARY KEY,
	user_id int,
	gift varchar(2555),
	FOREIGN KEY (user_id) REFERENCES users(id)
);

INSERT INTO users(id, username, password) values (1, 'Husenjan', 'secret');

INSERT INTO gifts(id, user_id, gift) values (1, 1, 'Husenjan', 'iPhone 16');
```

What is happening inside the script? The `users` and `gifts` table is deleted each execution and re-created again with the original records. 

## Conclusion

Understanding SQL can be difficult in the beginning. However, the more you practice it the better you will become at it and it's important to become familiar with it as a-lot organizations uses SQL is some way. And if you're interested in becoming a developer, software engineer, or security researcher it's important to be familiar with it as you will deal with it a lot.