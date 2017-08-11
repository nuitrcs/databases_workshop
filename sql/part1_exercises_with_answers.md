# SQL Part 1: Exercises
----

There may be other ways to achieve the same result.  Remember that SQL commands are not case sensitive (but data values are).

All of these exercises use the `dvdrental` database.  


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

Select the row from actors for actor Jamie Rice.

Select payment rows where the amount paid was less than $1.  

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
SELECT * FROM actor
WHERE first_name='Jamie' AND last_name='Rice';
```

```sql
SELECT * FROM payment 
WHERE amount < 1;
```


## Exercise: Working with Dates


#### Solution