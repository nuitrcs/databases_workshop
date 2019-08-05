# Exploring Databases

This workshop focuses on summarizing, cleaning, counting, and otherwise working with the data in a database.  

We cover:

* [Datatypes](#data-types)
* [Casting data to convert between types](#casting)
* [Dealing with `NULL` values](#null)
* [Using `LIKE` to query text data](#like)
* [String functions](#string-functions)
* [Numerical functions](#numerical-functions)
* [Date functions](#date-functions)


This workshop uses PostgreSQL, and it was written for version 11.4.  Much of the material though is relevant to other database systems and nearly all is the same for earlier versions of Postgres.  

# Exploring a Database

When we start working with a database, the first step is to find out what's in it.  We can use the describe commands in psql.  Other clients have other ways to get information about the tables and connections between them.

We have 2 sets of tables:

Evanston food inspection:
* business
* inspections
* violations

Flight data:
* airlines
* flights
* airports
* weather
* planes

One useful step is to see how many rows are in each table.

```sql
SELECT count(*) FROM flights;
SELECT count(*) FROM airlines;
```

For small tables, might as well print out all of the values:

```sql
SELECT * FROM airlines;
```

It's useful to see in practice how tables join together.  Let's look at the flights data.  The flights table says that the carrier column references the airlines table.  Let's do that join.

```sql
SELECT *  -- can use * in a join too
FROM flights
LEFT JOIN airlines
ON flights.carrier = airlines.carrier
LIMIT 10;
```

Hard to read.  How about just a few interesting columns?

```sql
SELECT name, flight, origin, dest 
FROM flights
LEFT JOIN airlines
ON flights.carrier = airlines.carrier
LIMIT 50;
```

## Subqueries and WITH

Is the same flight (airline and flight number combination) always between the same origin and destination cities?  This is a little tough to figure out with a query, and there are a few ways we could approach this.  Let's do it by 

1. getting all of the unique combinations of name, flight, origin, dest
2. grouping that result by name, flight and counting how many rows are in each group


First: 

```sql
SELECT DISTINCT name, flight, origin, dest
FROM flights
LEFT JOIN airlines
ON flights.carrier = airlines.carrier;
```

Now, we need to use the above as a subquery:

```sql
SELECT name, flight, count(*) 
FROM (SELECT DISTINCT name, flight, origin, dest
FROM flights
LEFT JOIN airlines
ON flights.carrier = airlines.carrier) AS f1
GROUP BY name, flight
HAVING count(*) > 1;
```

Yes, same carrier and flight number can be between different cities.  How can we see what cities.  We're going to use data from the first query more than once, so there's another way we can structure it.  With a `WITH` clause, which lets us run subqueries and give them names first, then write queries to use them.  First, restructure the same query above:

```sql
WITH f1 AS
  (SELECT DISTINCT name, flight, origin, dest
   FROM flights
   LEFT JOIN airlines
   ON flights.carrier = airlines.carrier)
SELECT name, flight, count(*) 
FROM f1
GROUP BY name, flight
HAVING count(*) > 1;
```

Now, also get data on origin and dest for duplicate flight numbers:

```sql
WITH f1 AS
  (SELECT DISTINCT name, flight, origin, dest
   FROM flights
   LEFT JOIN airlines
   ON flights.carrier = airlines.carrier),
dups AS 
  (SELECT name, flight, count(*) 
   FROM f1
   GROUP BY name, flight
   HAVING count(*) > 1)
SELECT f1.*, count 
FROM f1
INNER JOIN dups
ON f1.name = dups.name AND f1.flight = dups.flight;
```

### EXERCISE

Select all of the flights made by the oldest plane in the data.  

Hint: Get the year of the oldest plane with

```sql
SELECT min(year) 
FROM planes;
```

Then get the tailnum for this plane.  Then get the flights.  Use a WITH clause in your query.

Hint: fill in the \_\_\_ with the right values below.

```sql
WITH minyear AS 
  (SELECT ___ AS year FROM planes),
oldplane AS 
  (SELECT ___ FROM planes, minyear 
   WHERE planes.___= minyear.___)
SELECT * 
FROM flights, oldplane
WHERE ___ = ___;
```

Using cross joins above works only because `minyear` and `oldplane` each only have one row -- otherwise you'd get wrong results.

[Answers](explore_answers.md)

## Food Inspections
 
Food inspection data source information: 

* https://data.cityofevanston.org/Health-Human-Services/Food-Establishment-Violations/spu5-riv2/data
* https://data.cityofevanston.org/Health-Human-Services/Food-Establishment-Businesses/vu4y-h82f/data
* https://data.cityofevanston.org/Health-Human-Services/Food-Establishment-Inspections/dj7x-d2cq/data

What's in this data?  Select a few rows from each table to take a look.

Just because there isn't a formal foreign key relationship between tables, doesn't mean we can't join them.

Let's see how many violations per inspection.  

Inspections are identified by a license number and a date.  

```sql
SELECT inspections.license, score, inspections.date, count(violation)
FROM inspections
LEFT JOIN violations
ON inspections.date = violations.date 
AND inspections.license = violations.license
GROUP BY inspections.license, inspections.date, score;
```


### EXERCISES

Hint: all of these only require joins -- no subqueries or WITH clauses.  

Write a query to find the names of all businesses that had a violation of type `'(6) FOOD PROTECTION: Potentially hazardous food properly thawed.'`.  Hint: first think about how to join the tables, then think about what columns to select and which rows you want in the result.

Select the business name, violation type, and comment for only violations that were found on the most recent inspection of a business.

Select violations for inspections where the inspection score was below 80.  Select an appropriate/informative subset of the columns.  Order the results in an meaningful order.  Hint: you need to use both the business license and inspection date information to join the tables properly.  

Bonus: What is the most common violation type in inspections with total scores less than 80? 

[Answers](explore_answers.md)

# Data Types and Missing Data

## Data Types

Each column has to have one and only one type of data.

https://www.postgresql.org/docs/current/datatype.html

## Casting

We can convert between different types by casting.  We may need to do this to use functions that expect one specific type of data.  For example, we have integer data, but the function expects numeric.  

Sometimes conversion is done automatically, other times you must be explicit about it.

How do we cast data?  There are two common ways:

```sql
SELECT CAST(3.2 AS integer);
SELECT 3.2::integer;
```

What is happening here?  Is it rounding, or taking the floor?  Let's check:

```sql
SELECT CAST(3.5 AS integer);
SELECT CAST(3.9 AS integer);
```

We can cast other types too:

```sql
SELECT CAST(3.2 AS text);
```

Normally, we'd cast a column:

```sql 
SELECT humid, humid::int 
FROM weather
LIMIT 10;
```

### EXERCISE

Select `time_hour` and `time_hour` cast as date from `weather`; limit to a few rows.

[Answers](explore_answers.md)

## `NULL`

Recall that `NULL` is used for missing data in SQL. 

When we explore data, it's always good to see where the missing data is and how much there is.

Which columns in the flights data have missing data?  A few ways to approach this:

First, how many rows overall?

```sql
SELECT count(*) 
FROM flights;
```

Then we could count missing in each column separately:

```sql
SELECT count(*) 
FROM flights 
WHERE dep_time IS NULL;
``` 

Or we could count how many are not null in columns -- count will count non-NULL values:

```sql
SELECT count(NULL);
SELECT count(10);
```

```sql
SELECT count(dep_time) AS dep_time_count,
       count(sched_dep_time) AS sched_dep_time_count,
       count(dep_delay) AS dep_delay_count,
       count(arr_time) AS arr_time_count,
       count(sched_arr_time) AS sched_arr_time_count,
       count(arr_delay) AS arr_delay_count,
       count(air_time) AS air_time_count
FROM flights;    
```

We could subtract from total to be a bit more useful:

```sql
SELECT count(*) - count(dep_time) AS dep_time_count,
       count(*) - count(sched_dep_time) AS sched_dep_time_count,
       count(*) - count(dep_delay) AS dep_delay_count,
       count(*) - count(arr_time) AS arr_time_count,
       count(*) - count(sched_arr_time) AS sched_arr_time_count,
       count(*) - count(arr_delay) AS arr_delay_count,
       count(*) - count(air_time) AS air_time_count
FROM flights;  
```

Hmm, interesting pattern.  Are there any flights that have a departure time but not an arrival time?

```sql
SELECT * 
FROM flights
WHERE dep_time IS NOT NULL AND arr_time IS NULL
LIMIT 5;
```

Well that's concerning.  Did these flights disappear in mid-air?  How many are there?

```sql
SELECT count(*) 
FROM flights
WHERE dep_time IS NOT NULL AND arr_time IS NULL;
```

Do these flights have an air time?

```sql
SELECT count(*) 
FROM flights
WHERE dep_time IS NOT NULL AND arr_time IS NULL AND air_time IS NOT NULL;
```

Nope - well that's something at least.  If the flight is missing the arrival time, it's also missing the `air_time`.  

What about delays?

```sql
SELECT dep_delay
FROM flights
WHERE dep_time IS NOT NULL AND arr_time IS NULL;
```

Lots of different values here.

??? It's a mystery!


### EXERCISES

Are there any missing values in the food inspection data?

Which column in the weather table has the most missing values?

[Answers](explore_answers.md)

## Missing Rows

Another way that data can be missing is that there is no row for it at all?  How do we count rows that should be in the data but aren't?  We need to know what should be there for comparison.

The weather data generally has observations for each airport each hour.  Are there any missing observations?

```sql
SELECT origin, year, month, day, count(hour) AS hour_count
FROM weather
GROUP BY origin, year, month, day
ORDER BY hour_count;
```

Yes, there are days without 24 observations. What about days that are missing entirely?

### EXERCISE

Are there days for any airports where there are no weather observations at all?

Note that you can't do: `count(distinct year, month, day)` - you'll get an error.  So you'll need to use a subquery.

Step 1: select distinct dates by origin airport.

Step 2: Count number of days by origin airport from the results of Step 1 (step 1 becomes a subquery).

This won't exactly tell you what days are missing, but it will let you compare the number of days there to the expected number of days in a year.

Bonus exercise: Which hour of the day is most likely to be missing a weather observation?

Bonus exercise 2: Are there any duplicate measurements (same airport and time) in the weather data?


[Answers](explore_answers.md)





# LIKE

`LIKE` lets you do pattern matching on strings.  The only two pattern characters are `_` for a single character and `%` for any number of characters (including none).  In some implementations of SQL, `LIKE` is case-insensitive.  In PostgreSQL, it is case-sensitive; `ILIKE` is the PostgreSQL case-insensitive version.

Get businesses that start with A.

```sql
SELECT name FROM business WHERE name LIKE 'A%';
```   

Note that the following will yield no results:

```sql
SELECT name FROM business WHERE name LIKE 'a%';
``` 

But the following is ok:

```sql
SELECT name FROM business WHERE name ILIKE 'a%';
``` 

Get names that end with y:

```sql
SELECT name FROM business WHERE name LIKE '%y';
```

Get names with an x in them:

```sql
SELECT name FROM business WHERE name LIKE '%x%';
```

Get 4 letter names:

```sql
SELECT name FROM business WHERE name LIKE '____';
```

Businesses with a # in the name

```sql
SELECT name FROM business WHERE name LIKE '%#%';
```

Businesses without a # in the name

```sql
SELECT name FROM business WHERE name NOT LIKE '%#%';
```

Get businesses with at least 3 words in the name

```sql
SELECT name FROM business WHERE name LIKE '% % %';
```

### Exercise

How many Starbucks are in the Evanston inspection data?  How many businesses with Evanston in the name?

How many distinct violation types have FOOD in the violation name?

Which violation type is most likely to be a critical violation (see the comments)?

Bonus: Find any violation entries that do not conform to the pattern of most entries.

Bonus: Get all addresses where the street name starts with A (and only those addresses).

[Answers](explore_answers.md)

# String Functions

https://www.postgresql.org/docs/current/functions-string.html

## Splitting

We saw that the violations had three parts -- a number, a title, and a description.  

Similarly, addresses have multiple parts -- street number, street name, and suffix.  

What if we need to split up strings?

One useful function is `split_part` which splits a string on a delimiter.

Let's get rid of the extra description on the violations to make our summaries easier to read.

```sql
SELECT split_part(violation, ':', 1)
FROM violations;
```

What if we want to get rid of the number at the front, so we can group by the main title?

We can nest function calls

```sql
SELECT split_part(split_part(violation, ':', 1), ') ', 2)
FROM violations;
```

Now let's count by major grouping

```sql
SELECT split_part(split_part(violation, ':', 1), ') ', 2) AS type, count(*)
FROM violations
GROUP BY type
ORDER BY count;
```

### EXERCISE

Get just the street name from the business address (the result won't be perfect at this stage -- just do what you can with a simple query).  Group and count to see which street has the most food businesses.

[Answers](explore_answers.md)

## String length

What is the longest comment in the violations data?

First, need to get the comment lengths:

```sql
SELECT length(comments)
FROM violations;
```

Then what is the longest?

```sql
SELECT max(length(comments)) 
FROM violations;
```

Now the comment with that length:

```sql
SELECT comments
FROM violations
WHERE length(comments) = 
  (SELECT max(length(comments)) FROM violations);
```

## Other functions

Trimming:

```sql
SELECT trim(' A B C   ') = 'A B C';
```

Are there any violations with whitespace at the beginning or end?

```sql
SELECT length(violation) = length(trim(violation)) AS clean
FROM violations
WHERE length(violation) != length(trim(violation));
```

Nope - how else could we have checked that?

Joining:

We can concatenate strings with `||` or `concat` - the difference comes with NULL values:

```sql
SELECT 'a' || 'b' AS joined;
```

```sql
SELECT concat('a', 'b') AS joined;
```


```sql
SELECT 'a' || NULL AS joined;
```

```sql
SELECT concat('a', NULL) AS joined;
```

With columns instead:

```sql
SELECT city || ', ' || state 
FROM business;
```

```sql
SELECT DISTINCT city || ', ' || state 
FROM business;
```

### EXERCISE

There's at least one address in the business table that isn't capitalized like the others.  Look at the string functions, and find a way to fix it.

https://www.postgresql.org/docs/current/functions-string.html

Check the output of:

```sql
SELECT DISTINCT split_part(address, ' ', 2)
FROM business;
```

[Answers](explore_answers.md)

# CASE WHEN

We saw on a previous exercise that a few addresses have a 1/2 or B in with street number, which prevented us from getting the street name with our query:

```sql
SELECT address
FROM business
WHERE address LIKE '% 1/2 %' 
OR address LIKE '% B %';
```

Hmm, "B" can be with number or at the end.  There's just one case, so let's be more specific in the query:

```sql
SELECT address
FROM business
WHERE address LIKE '% 1/2 %' 
OR address LIKE '1916 B %';
```

So our counts of business by street wasn't accurate.

We can use `CASE WHEN` to do different data transformations in different situations.

```sql
SELECT CASE WHEN address LIKE '% 1/2 %' 
                  OR address LIKE '1916 B %'
            THEN split_part(address, ' ', 3)
        ELSE split_part(address, ' ', 2) 
        END
      AS streetname
FROM business;
```

Also remember `upper`:


```sql
SELECT CASE WHEN address LIKE '% 1/2 %' 
                  OR address LIKE '1916 B %'
            THEN split_part(address, ' ', 3)
        ELSE split_part(upper(address), ' ', 2) 
        END
      AS streetname, count(*)
FROM business
GROUP BY streetname
ORDER BY count DESC;
```

# Numerical Functions

## Transformations
There are purely numerical functions in SQL - ones that allow you to add, subtract, take exponents, etc.  Those are transformations of the data.  For example:

```sql
SELECT dep_delay FROM flights LIMIT 100;
SELECT dep_delay/60 FROM flights LIMIT 100;
```

WAIT!  What happened?

```sql
SELECT dep_delay/60.0 FROM flights LIMIT 100;
SELECT dep_delay/60::numeric FROM flights LIMIT 100;
```

## Single Variable Aggregation

Sometimes we do want to transform data, but for now we're interested here in aggregate functions: https://www.postgresql.org/docs/current/functions-aggregate.html -- ones that help us summarize the data.


Some basics:

```sql
SELECT min(temp), max(temp) 
FROM weather;
```

By group:

```sql
SELECT origin, avg(temp), stddev(temp)
FROM weather
GROUP BY origin;
```

Let's round those to make it easier to read:

```sql
SELECT origin, 
       round(avg(temp), 2), 
       round(stddev(temp), 2)
FROM weather
GROUP BY origin;
```

### EXERCISE

How much total precipitation did each airport get by month?  Hint: Use `sum()`, and you'll need to group by more than one column.  Order the results in a reasonable order.

Bonus: Using the results above, compute the average for each month across the three airports.

[Answers](explore_answers.md)

## Two-variable Statistics

There are a few two variable statistical functions available.  Correlation is one of them.

```sql
SELECT corr(temp, humid)
FROM weather;
```

```sql
SELECT origin, corr(temp, humid)
FROM weather
GROUP BY origin;
```

```sql
SELECT origin, corr(precip, humid)
FROM weather
GROUP BY origin;
```

# Numerical Distributions

When we have categorical data, we can count the number of observations in each category.  We can't do this with numerical data because there are too many distinct values (we could do it, but it isn't helpful).  In statistical programs, we'd make a histogram to see the data, but we can't do visualizations directly in the database.  What can we do?

We can round the values.  Let's look at humidity:

```sql
SELECT round(humid), count(*)
FROM weather
GROUP BY round(humid)
ORDER BY round(humid);
```

What's up with 95%?


What if we want to make bigger bins -- we still have nearly 100 values, which is a lot.  We can round to different place values:

```sql
SELECT round(humid, -1), count(*)
FROM weather
GROUP BY round(humid, -1)
ORDER BY round(humid, -1);
```

This is rounding.  We might want to truncate the number instead, so just take the first digit effectively, instead of rounding up or down:

```sql
SELECT trunc(humid, -1), count(*)
FROM weather
GROUP BY trunc(humid, -1)
ORDER BY trunc(humid, -1);
```

### EXERCISE

Examine the distribution of temperature like we did with humidity.  Then do it by airport (origin).  



# Date Functions

```sql
SELECT now();
```

Why doesn't the date/time for `now()` look right?  It uses UTC or GMT -- the time in England.

Dates are stored according to the ISO 8601 standard for dates: 

```
YYYY-MM-DD HH:MM:SS
```

Date/time fields are ordered largest to smallest.  

Date/time functions: https://www.postgresql.org/docs/current/functions-datetime.html

`min` and `max` work on dates too:

```sql
SELECT min(date), max(date) 
FROM inspections;
```

[Answers](explore_answers.md)

## Date/time parts

Functions exist to extract individual components of date/time data. 

Fields (components) include:

* century: 2019-01-01 = century 21
* decade: 2019-01-01 = decade 201
* year, month, day
* hour, minute, second
* week
* dow: day of week

dow is day of week. The week starts with Sunday, which has a value of 0, and ends on Saturday with a value of 6.

```sql
SELECT date_part('month', now()),
       EXTRACT(MONTH FROM now());
```

What day of the week do most food inspections take place on?

```sql
SELECT date_part('dow', date), count(*)
FROM inspections
GROUP BY date_part('dow', date)
ORDER BY date_part('dow', date);
```

Instead of extracting single fields, we can also truncate dates.  This is useful for aggregating over months across multiple years, or similar things.  

```sql
SELECT date_trunc('month', now());
SELECT date_trunc('hour', now());
```

The weather and flights tables have year, month, day, hour broken out as separate fields, but this isn't normal.  Typically you'd only have the timestamp fields.  

Before we did:

```sql
SELECT origin, month, sum(precip) 
FROM weather
GROUP BY origin, month
ORDER BY origin, month;
```

But if we didn't already have a month field, we could use a date function instead.

Because it's only one year of data, we could do `date_part`:

```sql
SELECT origin, 
       date_part('month', time_hour) AS dp_month, 
       sum(precip)
FROM weather
GROUP BY origin, dp_month
ORDER BY origin, dp_month;
```

But if we had multiple years, we'd want the year too.  

```sql
SELECT origin, 
       date_trunc('month', time_hour) AS dt_month, 
       sum(precip)
FROM weather
GROUP BY origin, dt_month
ORDER BY origin, dt_month;
```

### EXERCISES

Which day of the week had the most rain at JFK airport?


What week of the year in 2018 were the most food inspections done?


Bonus (Hard, because we didn't learn everything you need): What days had no inspections?  The `generate_series()` function can be used to create a series of dates.  

Example:

```sql
SELECT generate_series('2018-01-01', '2019-05-14', '1 day'::interval) AS days;
```

The above are timestamps, but they can be compared to dates.

Can you use the above to help you find dates with no inspections?  

Hint: Try starting from one of the templates below (either can work).

Join the results of generate series to inspections as if it's a table:

```sql
SELECT date 
FROM generate_series('2018-01-01', '2019-05-14', '1 day'::interval) AS days
___ JOIN ___
ON ___
WHERE ___;
```

Or use WITH to give a name to the results and the column, and then use it in a query:

```sql
WITH days AS 
     (SELECT * 
      FROM generate_series('2018-01-01', 
         '2019-05-14', '1 day'::interval) AS day)
SELECT ___
FROM ___ 
WHERE ___;
```

What days of the week are days without inspections?  Any that aren't on the weekend?

[Answers](explore_answers.md)

## Lead and Lag

When looking at a timeseries, sometimes we want to find the time between events.  So we need to subtract time between an event and the lag of the event -- the previous occurence.  

Let's find out the longest time span between inspections in the food data.  First, let's see how the lag (and lead) function works.

```sql
SELECT date,
       lag(date) OVER (ORDER BY date),
       lead(date) OVER (ORDER BY date)
  FROM inspections
 LIMIT 20;
```

Let's get the time between inspections:

```sql
SELECT date, 
       lag(date) OVER (ORDER BY date) AS lagged,
       date-lag(date) OVER (ORDER BY date) AS span
FROM inspections;
```

Now, let's find the longest span.  We don't just want the span, but also all the dates associated with the span:

```sql
WITH breaks AS 
   (SELECT date, 
       lag(date) OVER (ORDER BY date) AS lagged,
       date-lag(date) OVER (ORDER BY date) AS span
    FROM inspections)
SELECT date, lagged, span 
FROM breaks
WHERE span = (SELECT max(span) FROM breaks);
```

### EXERCISE

We don't just have to lag dates themselves -- we can also lag other columns by their date values.  Let's get the changes in temperature from one hour to the next at JFK.  And the find the biggest change.  

Remember to limit to just rows for JFK, and order by time_hour overall.  And you'll want the absolute value of the difference with function `abs`.

```sql
WITH changes AS 
   (SELECT time_hour, temp,
       lag(temp) OVER (ORDER BY time_hour) AS lagged,
       abs(___) AS diff
    FROM ___
    WHERE ___
    ORDER BY ___)
SELECT time_hour, temp, lagged, diff 
FROM changes
WHERE diff = (___);
```

Bonus: Can you write a query to select all rows where the `time_hour` date part is the day where the max change was observed?  Use `date_trunc()` with the part of the date you're truncating to being the `'day'`, and compare this to the date you want (ex. `'2013-01-01'`).  

[Answers](explore_answers.md)

# Bringing it Together

Everything above is about being able to ask and answer questions about the data directly in the database, without pulling data over into another program.  We haven't learned everything you need, but we have a start at finding and using functions and writing more complicated queries.  






