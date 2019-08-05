# SQL Part 1: Exercises
----

There may be other ways to achieve the same result.  Remember that SQL commands are not case sensitive (but data values are).

All of these exercises use the `dvdrental` database.  

Exercises often use multiple commands or aspects of SQL, but they are titled/grouped by their focus.


## Exercise: Describe Commands

Get a list of the tables in the database.


#### Solution

```sql
\dt
```

Note that you can type a string with wildcards (*) after the `dt` to just get tables with names matching the pattern.  Example:

```sql
\dt c*
```

To get tables with names that start with c.






## Exercise: Select 

Get a list of actors with the first name Julia.

Get a list of actors with the first name Chris, Cameron, or Cuba.  

Select the row from customer for customer named Jamie Rice.

Select amount and payment_date from payment where the amount paid was less than $1.  



#### Solution

```sql
SELECT * FROM actor WHERE first_name='Julia';
```

Remember that string/text values are case-sensitive, so 'julia' or 'JULIA' will not match.

```sql
SELECT * FROM actor 
WHERE first_name='Chris' 
OR first_name='Cameron' 
OR first_name='Cuba';

SELECT * FROM actor 
WHERE first_name IN ('Chris', 'Cameron', 'Cuba');
```

Either of the above will work for the second task.

```sql
SELECT * FROM customer
WHERE first_name='Jamie' AND last_name='Rice';
```

```sql
SELECT amount, payment_date FROM payment 
WHERE amount < 1;
```


## Exercise: Distinct

What are the different rental durations that the store allows?

```sql
SELECT DISTINCT rental_duration FROM film;
```





## Exercise: Order By

What are the IDs of the last 3 customers to return a rental?


#### Solution

```sql
SELECT customer_id FROM rental 
WHERE return_date IS NOT NULL -- without this, you get NULL values first
ORDER BY return_date DESC 
LIMIT 3;
```





## Exercise: Counting

How many films are rated NC-17?  How many are rated PG or PG-13?


Challenge: How many different customers have entries in the rental table?  [Hint](http://www.w3resource.com/sql/aggregate-functions/count-with-distinct.php)

#### Solution

```sql
SELECT count(*) FROM film 
WHERE rating = 'NC-17';
```


```sql
SELECT count(*) FROM film 
WHERE rating in ('PG', 'PG-13');
```

or 

```sql
SELECT count(*) FROM film 
WHERE rating = 'PG' OR rating = 'PG-13';
```

Challenge:

```sql
SELECT COUNT(DISTINCT customer_id) FROM rental;
```





## Exercise: Group By

Does the average replacement cost of a film differ by rating?


Challenge: Are there any customers with the same last name? 

#### Solution

```sql
SELECT rating, avg(replacement_cost) FROM film
GROUP BY rating;
```

Challenge:

```sql
SELECT last_name, count(*) FROM customer 
GROUP BY last_name HAVING count(*) > 1;
```

Answer: no



## Exercise: Functions

What is the average rental rate of films?  Can you round the result to 2 decimal places?

Challenge: What is the average time that people have rentals before returning?  Hint: the output you'll get may include a number of hours > 24.  You can use the function `justify_interval` on the result to get output that looks more like you might expect.

Challenge 2: Select the 10 actors who have the longest names (first and last name combined).


#### Solution

```sql
SELECT avg(rental_rate) FROM film;
SELECT round(avg(rental_rate), 2) FROM film;
```


Challenge:

```sql
SELECT justify_interval(avg(return_date-rental_date)) FROM rental;
```


Challenge 2:

```sql
SELECT concat(first_name, last_name), length(concat(first_name, last_name)) 
FROM actor 
ORDER BY length(concat(first_name, last_name)) desc
LIMIT 10;
```

or 


```sql
SELECT first_name || last_name, length(first_name || last_name) 
FROM actor 
ORDER BY length(first_name || last_name)  DESC
LIMIT 10;
```


## Exercise: Count, Group, and Order

Which film (id) has the most actors?  Which actor (id) is in the most films?

#### Solution

```sql
SELECT film_id, count(actor_id)
FROM film_actor
GROUP BY film_id
ORDER BY count(actor_id) DESC 
LIMIT 1;
```

```sql
SELECT actor_id, count(film_id)
FROM film_actor
GROUP BY actor_id
ORDER BY count(film_id) DESC 
LIMIT 1;
```
