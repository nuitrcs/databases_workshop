# SQL Part 4

* Transactions
* Update
* Delete
* Alter
* Drop


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

```sql
SELECT * FROM color;
```


Now, let's [alter](https://www.postgresql.org/docs/current/static/sql-altertable.html) this table in a transaction; we'll add a boolean (true/false) column called websafe and have it default to true:

```sql
begin;

ALTER TABLE color ADD COLUMN websafe boolean default true;

\d color

SELECT * FROM color;

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

Let's drop for real:

```sql
begin;
DROP TABLE color;
commit;
SELECT * FROM color;

```

Ok - let's recreate it to see what else we can alter:

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

```sql
SELECT * FROM color;
```

Let's change the name of the hex column

```sql
begin;
ALTER TABLE color RENAME hex TO webcolor;
\d color
SELECT * FROM color;
commit;
```

Let's change the type of the column to `char(7)` since hex colors have `#` and 6 letters/numbers.

```sql
begin;
ALTER TABLE color ALTER hex TYPE char(7);
\d color
SELECT * FROM color;
commit;
```

### Exercise: Alter

Create and populate the `food` table below using the commands provided. 

Insert a few additional rows into the table.

Then add a new text column `color`.  Then rename the `color` column you just created to `primary_color`.

```sql
CREATE TABLE food (
    id serial primary key,
    name text not null,
    type text,
    favorite boolean default false    
);

INSERT INTO food (name, type) 
VALUES ('broccoli','vegetable'), 
    ('lime', 'fruit'), 
    ('green beans', 'vegetable'), 
    ('milk', 'dairy'), 
    ('yogurt', 'dairy'), 
    ('banana', 'fruit'), 
    ('lemon', 'fruit'), 
    ('tortilla', 'carbohydrate'), 
    ('rice', 'carbohydrate');
``` 

[Answers](part4_exercises_with_answers.md)

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

You can update multiple columns with the same statement.

```sql
UPDATE workshop 
SET name='Introduction to R', beginner='t'
WHERE id=4;
```


### Exercise: Update

Using the food table created and altered above, set the values of the `primary_color` column (try to set the value for a few rows together with one query).  Then set the values of the `favorite` column based on your favorites.

[Answers](part4_exercises_with_answers.md)

## Update with Join

[Updating via a join](http://www.postgresqltutorial.com/postgresql-update-join/) (or [official documentation](https://www.postgresql.org/docs/9.6/static/sql-update.html))

```sql
CREATE TABLE product_segment (
    ID SERIAL PRIMARY KEY,
    segment VARCHAR NOT NULL,
    discount NUMERIC (4, 2)
);
 
 
INSERT INTO product_segment (segment, discount)
VALUES
    ('Grand Luxury', 0.05),
    ('Luxury', 0.06),
    ('Mass', 0.1);
    
CREATE TABLE product(
    id serial primary key,
    name varchar not null,
    price numeric(10,2),
    net_price numeric(10,2),
    segment_id int not null,
    foreign key(segment_id) references product_segment(id)
);
 
 
INSERT INTO product (name, price, segment_id) 
VALUES ('diam', 804.89, 1),
    ('vestibulum aliquet', 228.55, 3),
    ('lacinia erat', 366.45, 2),
    ('scelerisque quam turpis', 145.33, 3),
    ('justo lacinia', 551.77, 2),
    ('ultrices mattis odio', 261.58, 3),
    ('hendrerit', 519.62, 2),
    ('in hac habitasse', 843.31, 1),
    ('orci eget orci', 254.18, 3),
    ('pellentesque', 427.78, 2),
    ('sit amet nunc', 936.29, 1),
    ('sed vestibulum', 910.34, 1),
    ('turpis eget', 208.33, 3),
    ('cursus vestibulum', 985.45, 1),
    ('orci nullam', 841.26, 1),
    ('est quam pharetra', 896.38, 1),
    ('posuere', 575.74, 2),
    ('ligula', 530.64, 2),
    ('convallis', 892.43, 1),
    ('nulla elit ac', 161.71, 3);
```

Now, we want to update the prices in the product table using information about the discounts from the `product_segment` table.  

First, how do we join these tables?

```sql
SELECT * 
FROM product
LEFT JOIN product_segment
ON product.segment_id=product_segment.id;
```

We want to update the net\_price. 


```sql
UPDATE product
SET net_price = price - price * discount
FROM
product_segment
WHERE
product.segment_id = product_segment.id;
```



### Exercise

Create and populate tables using the supplied code below.


```sql
CREATE TABLE course (
    id int primary key,
    name text not null,
    last_taught date
);

INSERT INTO course (id, name) 
VALUES 
    (1, 'Chemistry'),
    (2, 'Physics'),
    (3, 'History'),
    (4, 'English'),
    (5, 'French');
    
CREATE TABLE course_offering (
    course_id int references course(id),
    quarter_name text,
    date date,
    primary key (course_id, quarter_name)
);

INSERT INTO course_offering 
VALUES 
    (1, 'Spring 2015', '2015-03-01'),
    (1, 'Spring 2017', '2017-03-01'),
    (2, 'Fall 2016', '2016-09-01'),
    (2, 'Spring 2017', '2017-03-01'),
    (3, 'Spring 2016', '2016-03-01'),
    (4, 'Winter 2015', '2015-01-01'),
    (4, 'Winter 2017', '2017-01-01'),
    (4, 'Winter 2016', '2016-01-01');
```


Set the value of `last_taught` in `course` to the most recent date the course was taught using the `course_offering` table.

Hint: you'll need to join to a subquery (the results of another query).  Think first about how to get the most recent date for each course, and then how to use that information in the update.  Alternatively, create a temporary table with the results of the query, then write an update statement using the temporary table.

[Answers](part4_exercises_with_answers.md)

## Delete

We can also delete rows.  If you neglect to add a where clause or mis-specify it, you can delete everything in your table.  So remember to use transactions.

```sql
begin;
DELETE FROM workshop WHERE id=4;
commit;
```

Note that if you try to delete rows that are referenced in other tables via foreign keys, you'll get an error. You'll need to update or delete references first, or manage these relationships automatically with something called [cascades](https://www.postgresql.org/docs/current/static/ddl-constraints.html#DDL-CONSTRAINTS-FK).

You can also delete using another table as we did with update, but the syntax is different:

```sql
DELETE FROM product
USING product_segment 
WHERE product.segment_id=product_segment.id
AND discount < .1;
```

Or we could rewrite this with a subquery:

```sql
DELETE FROM product
WHERE segment_id IN 
(SELECT segment_id 
 FROM product_segment
 WHERE discount < .1);
```

### Exercise: Delete

Using the `food` table created, altered, and updated above, delete any white foods that aren't a favorite.

Using the `course` table created above, first alter the table to remove the last\_taught column.  Then delete any courses that were last offered before 2017 (start date before 2017).  Note that you'll also need to delete entries from course\_offering table too.  Be careful not to delete old offerings of courses you aren't deleting.

[Answers](part4_exercises_with_answers.md)
