# SQL Part 4

Transactions

Update, delete, alter, drop


# Changing and Deleting Tables

## Alter

You can [alter](https://www.postgresql.org/docs/current/static/sql-altertable.html) a table after it's created.  Things you can do include: renaming the table, renaming columns, changing column types, adding or dropping constraints, adding or changing default values.


## Drop

To delete a table, use the drop command.  This cannot be undone, so best to really mean it and do it in a transaction.



# Transactions

SQL commands are executed immediately, unless you put them in a transaction.  With select statements, this doesn't make much of a difference.  But when altering the database, you often want to make sure commands execute correctly before committing the change to the database.  

In PostgreSQL, transactions begin with `begin;` and end with either `commit;` to commit the change or `rollback;` to undo everything since `begin`.  If there is an error in a transaction, you can't commit it; you have to use rollback to roll it back and then try again.  Note: The SQL standard for this is `START TRANSACTION`.

Let's see how transactions work.  We'll create a table and add some data to work with.

```sql
CREATE TABLE color (
	id serial primary key,
	name text,
	hex text -- hex color specification used with html
);

INSERT INTO color (name, hex) 
VALUES ('beige', '#F5F5DC'), ('coral', '#FF7F50'), 
		('cyan', '#00FFFF'), ('gold', '#FFD700');
```


Now, let's alter this table in a transaction; we'll add a boolean column called websafe and have it default to true:

```sql
begin;

ALTER TABLE color ADD COLUMN websafe boolean default true;

commit;
```

If I made a typo in the alter statement, and then I try to commit, PostgreSQL won't let me.  It will rollback the transaction instead of committing.  Also, once you have an error in a PostgreSQL transaction, you can't run any other commands.  

```sql
practice=# begin;
BEGIN

practice=# select * from turkey;
ERROR:  relation "turkey" does not exist
LINE 1: select * from turkey;
                      ^

practice=# select * from color;
ERROR:  current transaction is aborted, commands ignored until end of transaction block

practice=# commit;
ROLLBACK
```

Breaking this down, `select * from turkey;` caused an error, because we don't have a table named turkey.  This error is printed.  Then we tried to run a valid query `select * from color;` but instead of getting the result, we get an error telling us to end the transaction.  We use `commit;` to end the transaction, but the feedback output we receive is `ROLLBACK`, indicating that the transaction was rolled back, not committed.  Since we were just issuing select statements, they would have had no effect on the database anyway. 

Let's do an example with commands that do change the database.

```sql
begin;
DROP TABLE color;
select * from color;
commit;
select * from color;
```

```sql
practice=# begin;
BEGIN

practice=# DROP TABLE color;
DROP TABLE

practice=# select * from color;
ERROR:  relation "color" does not exist
LINE 1: select * from color;
                      ^

practice=# commit;
ROLLBACK

practice=# select * from color;
 id | name  |   hex   | websafe 
----+-------+---------+---------
  1 | beige | #F5F5DC | t
  2 | coral | #FF7F50 | t
  3 | cyan  | #00FFFF | t
  4 | gold  | #FFD700 | t
(4 rows)
```

Because the transaction was rolled back, the colors table was not dropped.  We can also explicitly use the rollback command on our own:

```sql
practice=# begin;
BEGIN

practice=# drop table color;
DROP TABLE

practice=# rollback;
ROLLBACK

practice=# select * from color;
 id | name  |   hex   | websafe 
----+-------+---------+---------
  1 | beige | #F5F5DC | t
  2 | coral | #FF7F50 | t
  3 | cyan  | #00FFFF | t
  4 | gold  | #FFD700 | t
(4 rows)


```

When dropping tables that are referenced by other tables, you are likely to get an error.  If another table depends on the table you're trying to delete, you have to update or delete that table first.  

# Changing and Deleting Rows in a Table

## Update

To change the values in a row, use `UPDATE`.  

Let's create a table and add some data to play with:

```sql
CREATE TABLE workshop (
	id serial primary key,
	name text not null,
	date date,
	beginner boolean default false
);

INSERT INTO workshop (name, date)
VALUES ('Intro to Python', '2017-07-10'), 
		('Python Data Analysis', '2017-08-03'), 
		('Databases', '2017-08-17'), 
		('Intro to R', '2017-09-07');
```

This gives us:

```sql
select * from workshop;
 id |         name         |    date    | beginner 
----+----------------------+------------+----------
  1 | Intro to Python      | 2017-07-10 | f
  2 | Python Data Analysis | 2017-08-03 | f
  3 | Databases            | 2017-08-17 | f
  4 | Intro to R           | 2017-09-07 | f
(4 rows)
```

Now, when we inserted values, we let the `beginner` column take the default value of false.  Let's change that for the second course.  

```sql
UPDATE workshop SET beginner='t' WHERE id=2;
```

The output we get, `UPDATE 1`, tells us how many rows were affected by the update.  Whatever rows are selected by the `where` clause of the query will be affected -- it doesn't have to be a single row.  

Now, let's change the date.  Say we got the date off by one day for the last two workshops.  

```sql
UPDATE workshop SET date=date+1 WHERE id >= 3;
```

```sql
practice=# select * from workshop;
 id |         name         |    date    | beginner 
----+----------------------+------------+----------
  1 | Intro to Python      | 2017-07-10 | f
  2 | Python Data Analysis | 2017-08-03 | t
  3 | Databases            | 2017-08-18 | f
  4 | Intro to R           | 2017-09-08 | f
(4 rows)

```



## Delete

We can also delete rows.  If you neglect to add a where clause or mis-specify it, you can delete everything in your table.  So remember to use transactions.

```sql
begin;
DELETE FROM workshop WHERE id=4;
commit;
```

Note that if you try to delete rows that are referenced in other tables via foreign keys, you'll get an error. You'll need to update or delete references first, or manage these relationships automatically with something called [cascades](https://www.postgresql.org/docs/current/static/ddl-constraints.html#DDL-CONSTRAINTS-FK).


