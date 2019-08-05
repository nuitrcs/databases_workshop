# SQL Part 2: Exercises
----

There may be other ways to achieve the same result.  Remember that SQL commands are not case sensitive (but data values and table and column names are).

All of these exercises use the `dvdrental` database.  

Exercises often use multiple commands or aspects of SQL, but they are titled/grouped by their focus.

[Answers](part2_exercises_with_answers.md)


## Exercise: Subqueries

What films are actors with ids 129 and 195 in together?

Challenge: How many actors are in more films than actor id 47?  Hint: this takes 2 subqueries (one nested in the other).  Work inside out: 1) how many films is actor 47 in; 2) which actors are in more films than this? 3) Count those actors.


## Exercise: Inner Joins

Select `first_name`, `last_name`, `amount`, and `payment_date` by joining the customer and payment tables.  

Select film\_id, category\_id, and name from joining the film\_category and category tables, only where the category\_id is less than 10.


## Exercise: Joining and Grouping: Customer Spending

Get a list of the names of customers who have spent more than $150, along with their total spending.

Who is the customer with the highest average payment amount?


## Exercise: Joining for Better Addresses

Create a list of addresses that includes the name of the city instead of an ID number and the name of the country as well.   

## Exercise: Joining Customers, Payments, and Staff

Join the customer and payment tables together with an inner join; select customer id, name, amount, and date and order by customer id.  Then join the staff table to them as well to add the staff's name.  


## Exercise: Joining and Grouping: Films and Actors

Repeating an exercise from Part 1, but adding in information from additional tables:  Which film (_by title_) has the most actors?  Which actor (_by name_) is in the most films?

Challenge: Which two actors have been in the most films together?  Hint: You can join a table to itself by including it twice with different aliases.  Hint 2: Try writing the query first to find the answer in terms of actor ids (not names); then for a super challenge (it takes a complicated query), rewrite it to get the actor names instead of the IDs.  Hint 3: make sure not to count pairs twice (a in the movie with b and b in the movie with a) and avoid counting cases of an actor being in a movie with themselves.






