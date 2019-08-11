# SQL Part 3

Creating tables, importing and exporting data

This part requires you to connect to a database where you have permissions to create new tables.

# `CREATE TABLE`

The basic syntax of creating a table is:

```sql
CREATE TABLE table_name (
   column_name TYPE COLUMN_CONSTRAINT,
   column_name TYPE COLUMN_CONSTRAINT,
   column_name TYPE COLUMN_CONSTRAINT,
   CONSTRAINT table_constraint
);
```

Constraints are conditions on one or more columns that link tables together and control what values a column can take.

Let's start with a concrete example.  We'll make a table for students.

```sql
CREATE TABLE student (
    id serial PRIMARY KEY,
    first_name text,
    last_name text,
    admission_year smallint 
);
```

A common style convention is to name tables with the singular form of a noun, not plural.  Tables and columns are named in all lowercase, with underscores as necessary to separate words.  People differ on whether `id` columns should just be `id` or `tablename_id` (so in this case, `student_id` vs. `id`).

Breaking this down piece by piece:

* `serial` is an integer that autoincrements.  It's a good choice when you want an ID column generated for you.  
* `PRIMARY KEY` means that the column in `NOT NULL` and `UNIQUE` -- so each row has a unique value on this column.  PostgreSQL will generate a name for this primary key constraint for you (you don't really need the name for most things you'll do).  An index will also be created from this column, which makes looking up rows in the table with values of this column more efficient.  
* `first_name` and `last_name` are type text.  Text isn't a standard SQL type, so if you'll be working across different systems, you might use `varchar` or `char` instead.  With these, you'd specify the number of characters, but with text you don't.  
* `admission_year` is a `smallint` which takes the range -32768 to 32767 (2 bits).  This range is sufficient for a year.  There are also `int` and `float` among some others.

[PostgreSQL Types](https://www.postgresql.org/docs/current/static/datatype.html)

Every table should have a primary key, although this isn't enforced in PostgreSQL.  You can use a combination of columns as the primary key instead of just defining it on one of them.

What does the describe command tell us about our new table:

```sql
practice=# \d student
                              Table "public.student"
     Column     |   Type   |                      Modifiers                       
----------------+----------+------------------------------------------------------
 id             | integer  | not null default nextval('student_id_seq'::regclass)
 first_name     | text     | 
 last_name      | text     | 
 admission_year | smallint | 
Indexes:
    "student_pkey" PRIMARY KEY, btree (id)
```

You can see the primary key index that was made, and the student id sequence.

Note that unless you're a superuser, others may not have permissions to your table.  Managing users and permissions is outside the scope of this workshop.

## Adding Values

### Insert

How do we add data to this table?  We can issue insert commands:

```sql
INSERT INTO student (first_name, last_name, admission_year) 
VALUES ('Alice', 'Walker', 2017),
       ('Bob', 'Williams', 2016),
       ('Charlie', 'Weston', 2016);
```


Each row goes in `()`, with each set of values separated by commas.  

Let's check:

```sql
SELECT * FROM student;
```

```sql
practice=# select * from student;
 id | first_name | last_name | admission_year 
----+------------+-----------+----------------
  1 | Alice      | Walker    |           2017
  2 | Bob        | Williams  |           2016
  3 | Charlie    | Weston    |           2016
(3 rows)
```

The id column was generated for us, starting at 1.  

Specifying the columns by name in the insert statement is optional.  We're doing it here because we're not inserting into every column - just 3 - and letting the ID be assigned automatically.  Specifying the columns by name also allows us specify what the order is.


An alternative to above without specifying the column names:

```sql
INSERT INTO student 
VALUES (DEFAULT, 'Alice', 'Walker', 2017),
       (DEFAULT, 'Bob', 'Williams', 2016),
       (DEFAULT, 'Charlie', 'Weston', 2016);
```

```sql
SELECT * FROM student;
```

It inserted them in again!  That's because there wasn't anything in the table definition precluding duplicates on names.  We'll learn how to delete rows later.

But, this demonstrates one reason you want a primary key -- without an ID column here, it would be very difficult to delete only one of each of the duplicates.  


### `copy`

We can also [copy](https://www.postgresql.org/docs/current/static/sql-copy.html) data from a file.  The PostgreSQL copy command works in reference to the file system on the database server.  So if you aren't running the server on your local machine, you can't use copy.  With the `psql` client, the `\copy` command, which uses the same syntax as `copy`, will transfer files between your local computer and the database server. 

Get the CSV file from: https://raw.githubusercontent.com/nuitrcs/databases_workshop/master/datafiles/studentdata.csv and save it somewhere you know the path to.  You can get it by opening another connection to the server and using

```
curl https://raw.githubusercontent.com/nuitrcs/databases_workshop/master/datafiles/studentdata.csv > studentdata.csv
```

In in-person workshops where we're using a remote server for `psql`, the file needs to be on that server. The file is in /tmp/studentdata.csv


Note that `\` commands like `\copy` in `psql` need to be all on one line without comments or line breaks in them.

```sql
\copy student (first_name, last_name, admission_year) FROM '/tmp/studentdata.csv' CSV HEADER
```

Change the path as appropriate above to the data file on your system.


Client programs have their own data import functions.  `\copy` is specific to the `psql` command line program.  





## Temporary Tables

You can create tables from result sets as well:

```sql
CREATE TABLE a_actors AS 
SELECT * FROM actor 
WHERE actor.first_name LIKE 'A%';
```

```sql
SELECT * FROM a_actors;
```

When creating a table, you also have the option to make the table temporary -- it will be deleted when your database session ends.  This is often useful with tables created from select statements, but it can be used with any table creation command.

```sql
CREATE TEMP TABLE b_actors AS 
SELECT * FROM actor 
WHERE actor.first_name LIKE 'B%';
```

```sql
SELECT * FROM b_actors;
```

Temporary tables can be useful for creating intermediate tables (to help simplify or speed up complex queries) or result sets you may want to export or use later.

Temporary tables are automatically deleted when your session ends.  Often users will have permissions to create temporary tables, but not permanent tables.  You become the owner of the table, and others will not have permission to view temporary tables by default:

```sql
\dt
```

Creating tables in this way copies all of the data.  The new table is independent from the table or tables it was created from.  

# Views

Instead of creating a new table (permanent or temporary), you can instead create a view (either permanent or temporary) that is essentially a saved query that you can reference as a table.  You can query the view like a table, but the data isn't copied -- it pulls the results from the original tables.  So if the original tables are updated, the results of the view will change.

```sql
CREATE TEMP VIEW actor_film_names AS 
SELECT title, first_name, last_name
FROM actor a
INNER JOIN film_actor fa
ON a.actor_id=fa.actor_id
INNER JOIN film f
ON f.film_id=fa.film_id;
```

```sql
SELECT * FROM actor_film_names LIMIT 5;
```

# Exporting Data

You can use [Copy](https://www.postgresql.org/docs/current/static/sql-copy.html) (or `\copy`) to export data too.  You have to specify an absolute file path when writing an output file.

```sql
\copy student TO '/Users/username/documents/mystudents.csv' CSV HEADER
```

You can copy tables by name or enter a query in \(\) in that place instead.

```sql
\copy (SELECT * FROM student LIMIT 5) TO '/Users/username/documents/mystudents5.csv' CSV HEADER
```

In `psql` there is also a `\o` command to open a file for writing (and then close it later):

```
\o out.txt
SELECT * FROM actor;
\o
```  

This just prints the output as it would be in the terminal to the file.  



