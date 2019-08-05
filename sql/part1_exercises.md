# SQL Part 1: Exercises
----

There may be other ways to achieve the same result.  Remember that SQL commands are not case sensitive (but data values are).

All of these exercises use the `dvdrental` database.  

Exercises often use multiple commands or aspects of SQL, but they are titled/grouped by their focus.

[Answers](part1_exercises_with_answers.md)



## Exercise: Describe Commands

Get a list of the tables in the database.


## Exercise: Select 

Get a list of actors with the first name Julia.

Get a list of actors with the first name Chris, Cameron, or Cuba.  

Select the row from customer for customer named Jamie Rice.

Select amount and payment_date from payment where the amount paid was less than $1.  



## Exercise: Distinct

What are the different rental durations that the store allows?




## Exercise: Order By

What are the IDs of the last 3 customers to return a rental?



## Exercise: Counting

How many films are rated NC-17?  How many are rated PG or PG-13?


Challenge: How many different customers have entries in the rental table?  [Hint](http://www.w3resource.com/sql/aggregate-functions/count-with-distinct.php)



## Exercise: Group By

Does the average replacement cost of a film differ by rating?


Challenge: Are there any customers with the same last name? 

## Exercise: Functions

What is the average rental rate of films?  Can you round the result to 2 decimal places?

Challenge: What is the average time that people have rentals before returning?  Hint: the output you'll get may include a number of hours > 24.  You can use the function `justify_interval` on the result to get output that looks more like you might expect.

Challenge 2: Select the 10 actors who have the longest names (first and last name combined).


## Exercise: Count, Group, and Order

Which film (id) has the most actors?  Which actor (id) is in the most films?

