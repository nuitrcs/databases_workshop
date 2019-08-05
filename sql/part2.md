# SQL Part 2

This section first covers the topics of aliasing and subqueries, then we get to joining tables, which is the real power of relational databases.

* [Aliasing](#aliasing)
* [Subqueries](#subqueries)
* [Joins](#joins)
    - [`INNER JOIN`](#inner-join)
    - [`LEFT JOIN`](#left-join)
    - [`FULL OUTER JOIN`](#full-outer-join)


# Aliasing

You can rename columns and tables in queries.  This will mostly be useful when we're joining tables together, but it can also be useful when you're working with functions.

```sql
SELECT language_id, count(*) AS c1 
FROM film GROUP BY language_id;
```

```sql
 language_id |  c1  
-------------+------
           1 | 1000
(1 row)
```

In the output above, the name of the column is the alias.

One important note is that _column_ aliases can't be used in where or having clauses:

```sql
SELECT title, rating AS rate
FROM film 
WHERE rate = 'G';
```

```sql
dvdrental=# SELECT title, rating AS rate
dvdrental-# FROM film 
dvdrental-# WHERE rate = 'G';
ERROR:  column "rate" does not exist
LINE 3: WHERE rate = 'G';
              ^
```

because the where operation happens before the selection.  


# Subqueries

We can use the results of one query as values in another query.  For example, if we wanted to get titles of films with below average rental rates:  

To start, what is the average rental rate?

```sql
SELECT avg(rental_rate) FROM film;
```

Then films with rates below that; order by rental rate to see the most expensive ones below that.

```sql
SELECT title, rental_rate FROM film
WHERE rental_rate < (SELECT avg(rental_rate) FROM film) 
ORDER BY rental_rate DESC;
```

The subquery is executed first, and then the result is used the broader query.

We can also use subqueries with `IN`.  Find customers with an address in `postal_code` 35200

```sql
SELECT * FROM customer 
WHERE address_id IN (SELECT address_id FROM address WHERE postal_code = '52137');
```

But you can also do the above query by joining tables together.  (`IN` is an expensive operation, meaning it can take a long time to run in large databases.)

You can also use a subquery to select from another result set.  In such cases, you have to give the subquery a name.

```sql
SELECT count(customer_id) FROM 
(SELECT customer_id, count(*) 
 FROM rental GROUP BY customer_id
 HAVING count(*) > 30) AS foo;
```

`foo` is a common throwaway name that gets used -- you can pick any name you want for the alias though.

## Exercise

Find the titles of movies that have the maximum replacement fee.


# Joins

Looking at the database [diagram](http://www.postgresqltutorial.com/postgresql-sample-database), we can see that information is split between lots of tables.  The lines between the tables show where there is a column in one table that is linked to a column in another table.  These are called foreign keys.  

There are also tables that only serve the purpose of linking two other tables to each other.  For example, the `film_actor` table.   

In the diagram, there are key icons next to some columns.  These columns are primary key columns.  A primary key can be a single column or a combination of multiple columns.  Primary keys have to have unique values.  They are frequently used to link tables to each other (although you could link tables with other columns too), and they are also used to index a table, which among other things makes lookups (where conditions) on those columns faster.

More on primary keys and foreign keys later, but for now, how to join tables.

## `INNER JOIN`

The first and most common type of join is called an inner join.  You specify the tables to join, the conditions to use to match the tables up, and you get back the rows from both tables that meet the conditions.

Let's start with the example we just used above: customers with postal code 52137. To start with, how do we join the tables generally:

```sql
SELECT * FROM customer 
INNER JOIN address 
ON customer.address_id = address.address_id;
```

This matches up the customer to the full address information.

Then we can select a specific postal code if we want:

```sql
SELECT * FROM customer 
INNER JOIN address 
ON customer.address_id = address.address_id
WHERE postal_code='52137';
```

Note that both tables have a column called `address_id`.  We add the table name to the front of the column name when referencing them.  You can do this anytime, but typically only do it when you're joining and there's ambiguity. 

We can also group by, order by, and use other where clause conditions on the joined tables.  For example, we can count the customers in each postal code.

```sql
SELECT postal_code, count(*) 
FROM customer 
INNER JOIN address 
ON customer.address_id = address.address_id
GROUP BY postal_code
ORDER BY count DESC;
``` 


### Alternative Syntax

There's another syntax we can use as well to get the same result:

```sql
SELECT * FROM customer, address
WHERE customer.address_id = address.address_id;
```

adding in the postal code:

```sql
SELECT * FROM customer, address
WHERE customer.address_id = address.address_id
AND postal_code='52137';
```

Using this syntax, you're really doing something called a cross join, and then limiting the results with a where clause.  

To think about how this works, you have every row in the first table matched to every row in the second table.  So if one table has n rows, and the second has m rows, you'd have n x m rows.  Then you have to select from this cross of the two tables the cases where values match up as you want them to.

With `INNER JOIN`, the `ON` part is required.  With a cross join like this, you could omit the `WHERE` clause and still get results.  (But it's rare that you'd really want to do this is real life.)  Example: 


```sql
SELECT customer_id, customer.address_id, address.address_id, address 
FROM customer, address
LIMIT 10;
```

### Exercises

Join the store table to the address table to add the address information to the store information.



### Table Names and Aliases

We can alias tables as well as columns.  If a column name appears in both tables, then we have to specify the table name when selecting it.

```sql 
SELECT first_name, last_name, customer.address_id, postal_code 
FROM customer
INNER JOIN address
ON customer.address_id = address.address_id;
```

If we don't put a table name in front of `address_id` we get an error:

```sql
dvdrental=# SELECT first_name, last_name, address_id, postal_code 
dvdrental-# FROM customer
dvdrental-# INNER JOIN address
dvdrental-# ON customer.address_id = address.address_id;
ERROR:  column reference "address_id" is ambiguous
LINE 1: SELECT first_name, last_name, address_id, postal_code 
                                      ^
```

To make the references easier, it's common to alias table names

```sql 
SELECT first_name, last_name, c.address_id, postal_code 
FROM customer AS c
INNER JOIN address AS a
ON c.address_id = a.address_id;
```

and we often drop the `AS`:

```sql 
SELECT first_name, last_name, c.address_id, postal_code 
FROM customer c
INNER JOIN address a
ON c.address_id = a.address_id;
```

The _table_ aliases can be used in the where clause as well as the select part of the statement.

---

Break for exercises: [part2_exercises.md](part2_exercises.md) - Subqueries, Inner Joins, and Joining and Grouping: Customer Spending

---





### More than 2 Tables

We can join more than 2 tables together.  Let's match the names of actors with the titles of the films they've been in.  We'll need the `film` and `actor` tables, as well as the `film_actor` table that links the two.


```sql
SELECT title, first_name, last_name 
FROM film f 
INNER JOIN film_actor fa ON f.film_id=fa.film_id 
INNER JOIN actor a ON fa.actor_id=a.actor_id;
```

Or

```sql
SELECT title, first_name, last_name
FROM film f, film_actor fa, actor a
WHERE f.film_id=fa.film_id AND fa.actor_id=a.actor_id;
```

### Exercise

Join store, address, and city tables to show the store\_id, address, and city name.


## `LEFT JOIN`

With an inner join, we only get the results that are in both tables.  But there are other types of joins.

![](presentation_assets/joins.png)



If we want to know which rows in a table don't have a match in the other table, we use a `LEFT JOIN` or `RIGHT JOIN` (depending on which table you want all of the results from).

In the dvd database, there can be films that don't have an inventory record.  We don't want these to be dropped from our results of joining the film and inventory tables.  Start with the join.  

```sql
SELECT f.film_id, title, inventory_id, store_id 
FROM film f 
LEFT JOIN inventory i
ON f.film_id=i.film_id;
```

Now find the rows where there isn't matching inventory:

```sql
SELECT f.film_id, title, inventory_id, store_id 
FROM film f 
LEFT JOIN inventory i
ON f.film_id=i.film_id
WHERE i.film_id IS NULL;
```

### Exercise

Are all cities listed in the city table associated with an address?  If any aren't, which cities are they?


## `FULL OUTER JOIN`

A `FULL OUTER JOIN` is like doing a left and right join at the same time: you get rows that are in both tables, plus rows from both tables that don't match the other table.

There aren't any tables with this type of relationship to each other in the dvdrental database, so we aren't going to do an example here.  The syntax is the same as the other joins.  




---

Break for exercises: [part2_exercises.md](part2_exercises.md) - Remaining Sections

---