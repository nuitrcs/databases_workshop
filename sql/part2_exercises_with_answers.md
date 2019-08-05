# SQL Part 2: Exercises
----

There may be other ways to achieve the same result.  Remember that SQL commands are not case sensitive (but data values are).

All of these exercises use the `dvdrental` database.  

Exercises often use multiple commands or aspects of SQL, but they are titled/grouped by their focus.


## Exercise: Subqueries

What films are actors with ids 129 and 195 in together?

Challenge: How many actors are in more films than actor id 47?  Hint: this takes 2 subqueries (one nested in the other).  Work inside out: 1) how many films is actor 47 in; 2) which actors are in more films than this? 3) Count those actors.



#### Solution

```sql
SELECT film_id FROM film_actor
WHERE actor_id=129
AND film_id IN (SELECT film_id FROM film_actor WHERE actor_id=195);
```

Challenge: 

```sql
SELECT count(actor_id) FROM
  (SELECT actor_id, count(film_id)
   FROM film_actor
   GROUP BY actor_id
   HAVING count(film_id) > (SELECT count(*) -- nested subquery
   		 			 FROM film_actor 
   		 			 WHERE actor_id=47) -- end nested subquery
   	) foo; -- ending and aliasing subquery
```


## Exercise: Inner Joins

Select `first_name`, `last_name`, `amount`, and `payment_date` by joining the customer and payment tables.  


Select film\_id, category\_id, and name from joining the film\_category and category tables, only where the category\_id is less than 10.


#### Solutions


```sql
SELECT first_name, last_name, amount, payment_date
FROM customer c 
INNER JOIN payment p
ON c.customer_id=p.customer_id;
```

```sql
SELECT film_id, c.category_id, name
FROM film_category fc
INNER JOIN category c
ON fc.category_id = c.category_id
WHERE c.category < 10;
```

TODO: check above



## Exercise: Joining and Grouping: Customer Spending

Get a list of the names of customers who have spent more than $150, along with their total spending.

Who is the customer with the highest average payment amount?


#### Solution

```sql
SELECT first_name, last_name, sum(amount)
FROM customer c 
INNER JOIN payment p
ON c.customer_id=p.customer_id
GROUP BY first_name, last_name
HAVING sum(amount) > 150;
```

```sql
SELECT c.customer_id, first_name, last_name, avg(amount)
FROM customer c 
INNER JOIN payment p
ON c.customer_id=p.customer_id
GROUP BY c.customer_id, first_name, last_name
ORDER BY avg(amount) DESC 
LIMIT 1;
```






## Exercise: Joining Customers, Payments, and Staff



Join the customer and payment tables together with an inner join; select customer id, name, amount, and date and order by customer id.  Then join the staff table to them as well to add the staff's name.  

#### Solutions

```sql
SELECT
 customer.customer_id,
 first_name,
 last_name,
 amount,
 payment_date
FROM
 customer
INNER JOIN payment ON payment.customer_id = customer.customer_id
ORDER BY
 customer.customer_id;
```

```sql
SELECT
 customer.customer_id,
 customer.first_name customer_first_name,
 customer.last_name customer_last_name,
 staff.first_name staff_first_name,
 staff.last_name staff_last_name,
 amount,
 payment_date
FROM
 customer
INNER JOIN payment ON payment.customer_id = customer.customer_id
INNER JOIN staff ON payment.staff_id = staff.staff_id
ORDER BY
 customer.customer_id;
```

## Exercise: Joining for Better Addresses

Create a list of addresses that includes the name of the city instead of an ID number and the name of the country as well.   


#### Solution


```sql
SELECT address, address2, district, postal_code, city, country 
FROM address
INNER JOIN city ON address.city_id=city.city_id
INNER JOIN country ON city.country_id = country.country_id;
```

or

```sql
SELECT address, address2, district, postal_code, city, country 
FROM address, city, country
WHERE address.city_id=city.city_id 
AND city.country_id = country.country_id;
```




## Exercise: Joining and Grouping: Films and Actors

Repeating an exercise from Part 1, but adding in information from additional tables:  Which film (_by title_) has the most actors?  Which actor (_by name_) is in the most films?

Challenge: Which two actors have been in the most films together?  Hint: You can join a table to itself by including it twice with different aliases.  Hint 2: Try writing the query first to find the answer in terms of actor ids (not names); then for a super challenge (it takes a complicated query), rewrite it to get the actor names instead of the IDs.  Hint 3: make sure not to count pairs twice (a in the movie with b and b in the movie with a) and avoid counting cases of an actor being in a movie with themselves.


#### Solution


```sql
SELECT title, count(actor_id) 
FROM film, film_actor
WHERE film.film_id=film_actor.film_id
GROUP BY title
ORDER BY count(actor_id) DESC 
LIMIT 1;
```

```sql
SELECT first_name, last_name, count(film_id) 
FROM actor, film_actor
WHERE actor.actor_id=film_actor.actor_id
GROUP BY first_name, last_name
ORDER BY count(film_id) DESC 
LIMIT 1;
```

** Alternative Syntax:**

```sql
SELECT title, count(actor_id) 
FROM film, film_actor
WHERE film.film_id=film_actor.film_id
GROUP BY title
ORDER BY count(actor_id) DESC 
LIMIT 1;
```

```sql
SELECT first_name, last_name, count(film_id) 
FROM actor, film_actor
WHERE actor.actor_id=film_actor.actor_id
GROUP BY first_name, last_name
ORDER BY count(film_id) DESC 
LIMIT 1;
```

Challenge:

```sql
SELECT a.actor_id, b.actor_id, count(*)
FROM film_actor a, film_actor b  -- join the table to itself
WHERE a.film_id=b.film_id  -- on the film id
      AND a.actor_id > b.actor_id  -- avoid duplicates and matching to the same actor
GROUP BY a.actor_id, b.actor_id
ORDER BY count(*) DESC 
LIMIT 1;
```

Super Challenge:

```sql
SELECT c.first_name, c.last_name, d.first_name, d.last_name, fcount
FROM
(SELECT a.actor_id AS a1, b.actor_id AS a2, count(*) AS fcount
FROM film_actor a, film_actor b  -- join the table to itself
WHERE a.film_id=b.film_id  -- on the film id
      AND a.actor_id > b.actor_id  -- avoid duplicates and matching to the same actor
GROUP BY a.actor_id, b.actor_id) foo  -- this is the query from above
INNER JOIN actor c ON c.actor_id=a1
INNER JOIN actor d ON d.actor_id=a2
ORDER BY fcount DESC LIMIT 1;
```

There are other ways to accomplish the above.




