# SQL Part 1

## Connecting

To connect to a database, you need the following information:

* Host/server name: 
* Database name: dvdrental
* Username: 
* Password: 
* Port: 5432 (default for PostgreSQL)

You also need a client program to connect to the database.  It's suggested that you start with `psql`, which is a command-line client.  Output below is from `psql`.


# Database Schema

We're working with the `dvdrental` database from this [PostgreSQL Tutorial](http://www.postgresqltutorial.com/).

The schema (the set of tables, their columns and types, and the relationships between them) is diagramed here: http://www.postgresqltutorial.com/postgresql-sample-database.  

![schema](http://www.postgresqltutorial.com/wp-content/uploads/2013/05/PostgreSQL-Sample-Database.png)

We can also get information directly from the database itself.  These commands are specific to each database system.  The commands below are for PostgreSQL.  

First, use `\d` to get a list of relations (tables, views, sequences) -- we'll talk about what views and sequences are later.  

```sql
\dvdrental=# \d
                     List of relations
 Schema |            Name            |   Type   |  Owner   
--------+----------------------------+----------+----------
 public | actor                      | table    | postgres
 public | actor_actor_id_seq         | sequence | postgres
 public | actor_info                 | view     | postgres
 public | address                    | table    | postgres
 public | address_address_id_seq     | sequence | postgres
 public | category                   | table    | postgres
 public | category_category_id_seq   | sequence | postgres
 public | city                       | table    | postgres
 public | city_city_id_seq           | sequence | postgres
 public | country                    | table    | postgres
 public | country_country_id_seq     | sequence | postgres
 public | customer                   | table    | postgres
 public | customer_customer_id_seq   | sequence | postgres
 public | customer_list              | view     | postgres
 public | film                       | table    | postgres
 public | film_actor                 | table    | postgres
 public | film_category              | table    | postgres
 public | film_film_id_seq           | sequence | postgres
 public | film_list                  | view     | postgres
 public | inventory                  | table    | postgres
 public | inventory_inventory_id_seq | sequence | postgres
 public | language                   | table    | postgres
 public | language_language_id_seq   | sequence | postgres
 public | nicer_but_slower_film_list | view     | postgres
 public | payment                    | table    | postgres
 public | payment_payment_id_seq     | sequence | postgres
 public | rental                     | table    | postgres
 public | rental_rental_id_seq       | sequence | postgres
 public | sales_by_film_category     | view     | postgres
 public | sales_by_store             | view     | postgres
 public | staff                      | table    | postgres
 public | staff_list                 | view     | postgres
 public | staff_staff_id_seq         | sequence | postgres
 public | store                      | table    | postgres
 public | store_store_id_seq         | sequence | postgres
(35 rows)
```


Or use `\dt` to get a list of just tables:

```sql
dvdrental=# \dt
             List of relations
 Schema |     Name      | Type  |  Owner   
--------+---------------+-------+----------
 public | actor         | table | postgres
 public | address       | table | postgres
 public | category      | table | postgres
 public | city          | table | postgres
 public | country       | table | postgres
 public | customer      | table | postgres
 public | film          | table | postgres
 public | film_actor    | table | postgres
 public | film_category | table | postgres
 public | inventory     | table | postgres
 public | language      | table | postgres
 public | payment       | table | postgres
 public | rental        | table | postgres
 public | staff         | table | postgres
 public | store         | table | postgres
(15 rows)
```


To get information for an individual table, use 

```
\d tablename
```

```sql
dvdrental=# \d actor
                                         Table "public.actor"
   Column    |            Type             |                        Modifiers                         
-------------+-----------------------------+----------------------------------------------------------
 actor_id    | integer                     | not null default nextval('actor_actor_id_seq'::regclass)
 first_name  | character varying(45)       | not null
 last_name   | character varying(45)       | not null
 last_update | timestamp without time zone | not null default now()
Indexes:
    "actor_pkey" PRIMARY KEY, btree (actor_id)
    "idx_actor_last_name" btree (last_name)
Referenced by:
    TABLE "film_actor" CONSTRAINT "film_actor_actor_id_fkey" FOREIGN KEY (actor_id) REFERENCES actor(actor_id) ON UPDATE CASCADE ON DELETE RESTRICT
Triggers:
    last_updated BEFORE UPDATE ON actor FOR EACH ROW EXECUTE PROCEDURE last_updated()
```

There are also other `\d` describe functions, among them:

`\dn`: list schemas, where are collections of objects (tables, functions) grouped together under a common name

`\df`: list functions

# Select

Select is the command we use most often is SQL.  It let's us select data (specified rows and columns) from one or more tables.  Columns are selected by name, rows are selected with conditional statements (values of a particular column meeting some criteria).  

The basic format of a `SELECT` command is 

```sql
SELECT column_1, column_2 
FROM table1;
```

`SELECT` and `FROM` are reserved keywords.  SQL is case-insensitive, but many times you'll see the key terms in all caps.  Note that you use a semicolon `;` to end the statement.  You can also split a SQL statement across multiple lines -- the space between the terms doesn't matter (a new line counts as space).

Let's start with the customer table.   The columns in the customer table are:

```sql
dvdrental=# \d customer
                                          Table "public.customer"
   Column    |            Type             |                           Modifiers                            
-------------+-----------------------------+----------------------------------------------------------------
 customer_id | integer                     | not null default nextval('customer_customer_id_seq'::regclass)
 store_id    | smallint                    | not null
 first_name  | character varying(45)       | not null
 last_name   | character varying(45)       | not null
 email       | character varying(50)       | 
 address_id  | smallint                    | not null
 activebool  | boolean                     | not null default true
 create_date | date                        | not null default ('now'::text)::date
 last_update | timestamp without time zone | default now()
 active      | integer                     | 
Indexes:
    "customer_pkey" PRIMARY KEY, btree (customer_id)
    "idx_fk_address_id" btree (address_id)
    "idx_fk_store_id" btree (store_id)
    "idx_last_name" btree (last_name)
Foreign-key constraints:
    "customer_address_id_fkey" FOREIGN KEY (address_id) REFERENCES address(address_id) ON UPDATE CASCADE ON DELETE RESTRICT
Referenced by:
    TABLE "payment" CONSTRAINT "payment_customer_id_fkey" FOREIGN KEY (customer_id) REFERENCES customer(customer_id) ON UPDATE CASCADE ON DELETE RESTRICT
    TABLE "rental" CONSTRAINT "rental_customer_id_fkey" FOREIGN KEY (customer_id) REFERENCES customer(customer_id) ON UPDATE CASCADE ON DELETE RESTRICT
Triggers:
    last_updated BEFORE UPDATE ON customer FOR EACH ROW EXECUTE PROCEDURE last_updated()
```

To select columns (all rows) we can name the columns:

```sql
SELECT customer_id, store_id FROM customer;
```

Generally with default `psql` settings we will get paged output.  Hit space to get more.  Type `q` or control-c to get your database prompt back.

```
 customer_id | store_id 
-------------+----------
         524 |        1
           1 |        1
           2 |        1
           3 |        1
           4 |        2
           5 |        1
           6 |        2
           7 |        1
           8 |        2
           9 |        2

```

A pipe character `|` delimits columns.  

If we want all of the columns, we can use `*` as shorthand:

```sql
SELECT * FROM customer;
```

```
 customer_id | store_id | first_name  |  last_name   |                  email                   | address_id | activebool | create_date |       last_update       | active 
-------------+----------+-------------+--------------+------------------------------------------+------------+------------+-------------+-------------------------+--------
         524 |        1 | Jared       | Ely          | jared.ely@sakilacustomer.org             |        530 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           1 |        1 | Mary        | Smith        | mary.smith@sakilacustomer.org            |          5 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           2 |        1 | Patricia    | Johnson      | patricia.johnson@sakilacustomer.org      |          6 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           3 |        1 | Linda       | Williams     | linda.williams@sakilacustomer.org        |          7 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           4 |        2 | Barbara     | Jones        | barbara.jones@sakilacustomer.org         |          8 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           5 |        1 | Elizabeth   | Brown        | elizabeth.brown@sakilacustomer.org       |          9 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           6 |        2 | Jennifer    | Davis        | jennifer.davis@sakilacustomer.org        |         10 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           7 |        1 | Maria       | Miller       | maria.miller@sakilacustomer.org          |         11 | t          |           
```

## `limit`

Instead of getting all rows, we can specify a limit of the number of rows to retrieve.

```sql
SELECT customer_id, store_id, first_name, last_name 
FROM customer
LIMIT 5;
```

```sql
 customer_id | store_id | first_name | last_name 
-------------+----------+------------+-----------
         524 |        1 | Jared      | Ely
           1 |        1 | Mary       | Smith
           2 |        1 | Patricia   | Johnson
           3 |        1 | Linda      | Williams
           4 |        2 | Barbara    | Jones
(5 rows)
```

Here, we got all of the output on a single page, and we can see the row count output at the end.

The order of the rows is not random, but it is not guaranteed to be in any particular order by default either.  

## `where`

Instead of getting all rows or a specific number of rows, we can also specify which rows we want by specifying conditions on the values of particular columns (e.g. equals, greater than, less than).

For example, we can select rows from `customer` that have a `store_id=2` with:

```sql
SELECT * FROM customer WHERE store_id=2;
```

(_Going forward, output will only be included when there's something about it to discuss._)

You can combined conditions together with `AND` and `OR`:

```sql
SELECT * FROM customer WHERE store_id=2 AND customer_id=400;
```

`WHERE` operators include:

| Operator | Description |
|:---:|---|
| = | Equal |
| > | Greater than |
| < | Less than |
| >= | Greater than or equal |
| <= | Less than or equal |
| <> or != | Not equal |
| AND | Logical operator AND |
| OR | Logical operator OR |


## `distinct`

## `order by`

## `between`, `in`, `like`

## `group by`

## `having`

## Joins

### `inner join`

### `left join`

### `right join`

