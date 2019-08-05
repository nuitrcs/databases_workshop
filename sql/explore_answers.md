# Exercise Answers

Answers to some of the exercises in [explore.md](explore.md).


## Exploring a database

### WITH

Select all of the flights made by the oldest plane in the data.  

Hint: Get the year of the oldest plane with

```sql
SELECT min(year) 
FROM planes;
```

Then get the tailnum for this plane.  Then get the flights.  Use a WITH clause in your query.

```sql
WITH minyear AS 
  (SELECT min(year) AS year FROM planes),
oldplane AS 
  (SELECT tailnum FROM planes, minyear WHERE planes.year = minyear.year)
SELECT * 
FROM flights, oldplane
WHERE flights.tailnum = oldplane.tailnum;
```


### More exploring

Write a query to find the names of all businesses that had a violation of type `'(6) FOOD PROTECTION: Potentially hazardous food properly thawed.'`.

```sql
SELECT name
FROM business
INNER JOIN violations
ON business.license = violations.license
WHERE violation = '(6) FOOD PROTECTION: Potentially hazardous food properly thawed.';
```

Select the business name, violation type, and comment for only violations that were found on the most recent inspection of a business.

```sql
SELECT name, violation, comments
FROM violations
INNER JOIN business
ON business.license = violations.license
WHERE date = last_inspection;
```

Select violations for inspections where the inspection score was below 80.  Select an appropriate/informative subset of the columns.  Order the results in an meaningful order.  Hint: you need to use both the business license and inspection date information to join the tables properly.  

```sql
SELECT name, inspections.date, score, violation
FROM inspections
INNER JOIN business
ON inspections.license = business.license
INNER JOIN violations
ON violations.license = business.license AND violations.date = inspections.date
WHERE score < 80
ORDER BY name, date;
```

Bonus: What is the most common violation type in inspections with total scores less than 80? 

```sql
SELECT violation, count(*)
FROM inspections
INNER JOIN business
ON inspections.license = business.license
INNER JOIN violations
ON violations.license = business.license AND violations.date = inspections.date
WHERE score < 80
GROUP BY violation
ORDER BY count DESC;
```


## Data Types

Select `time_hour` and `time_hour` cast as date from `weather`; limit to a few rows.

```sql
SELECT time_hour, time_hour::date 
FROM weather
LIMIT 20;
```

## Missing Rows

Are there days for any airports where there are no weather observations at all?

Note that you can't do: `count(distinct year, month, day)` - you'll get an error.  So you'll need to use a subquery.

Step 1: select distinct dates by origin airport.

Step 2: Count number of days by origin airport from the results of Step 1 (step 1 becomes a subquery).

This won't exactly tell you what days are missing, but it will let you compare the number of days there to the expected number of days in a year.

```sql
SELECT origin, count(*)
FROM (SELECT DISTINCT origin, year, month, day
   FROM weather) AS distinct_days
GROUP BY origin
ORDER BY count DESC;
```


Alternative with WITH:

```sql
WITH distinct_days AS 
  (SELECT DISTINCT origin, year, month, day
   FROM weather)
SELECT origin, count(*) 
FROM distinct_days
GROUP BY origin
ORDER BY count DESC;
```



Bonus exercise: Which hour of the day is most likely to be missing a weather observation?

```sql
SELECT hour, count(*)
FROM weather
GROUP BY hour
ORDER BY count;
```

The midnight (hour 0) hour has the fewest observations in the data, so is therefore the most likely to be missing.

Bonus exercise 2: Are there any duplicate measurements (same airport and time) in the weather data?

```sql
SELECT origin, year, month, day, hour, count(*) 
FROM weather
GROUP BY origin, year, month, day, hour
HAVING count(*) > 1;
```



## LIKE

How many Starbucks are in the Evanston inspection data?

```sql
SELECT count(*) 
FROM business
WHERE name LIKE 'Starbucks%';
```

How many distinct violation types have FOOD in the violation name?

```sql
SELECT count(DISTINCT violation)
FROM violations
WHERE violation LIKE '%FOOD%';
```

Which violation type is most likely to be a critical violation (see the comments)?

```sql
SELECT violation, count(*) 
FROM violations
WHERE comments LIKE '%CRITICAL VIOLATION%'
GROUP BY violation
ORDER BY count; 
```

Find any violation entries that do not conform to the pattern of `(#) VIOLATION TITLE: violation description`

```sql
SELECT DISTINCT violation
FROM violations
WHERE violation NOT LIKE '(%) %: %';
```

Challenge: Get all addresses where the street name starts with A (and only those addresses).


```sql
SELECT address
FROM business
WHERE address LIKE '% A% AVE';
```

Note that the solution above is specific to the values in the data.  There are no "Streets" that start with A, only "Avenues",  We'd have to change the query if the data was different.











## String Splitting

Get just the street name from the business address (the result won't be perfect at this stage -- just do what you can with a simple query).  Group and count to see which street has the most food businesses.

Let's take this in steps

```sql
SELECT split_part(address, ' ', 2)
FROM business;
```

What are the unique values?  (To check)

```sql
SELECT DISTINCT split_part(address, ' ', 2)
FROM business;
```

Which are most common?

```sql
SELECT DISTINCT split_part(address, ' ', 2) AS street, count(*)
FROM business
GROUP BY street
ORDER BY count DESC;
```

## Numerical Functions

How much total precipitation did each airport get by month?  Hint: Use `sum()`, and you'll need to group by several columns.  Order the results in a reasonable order.

```sql
SELECT origin, month, sum(precip) 
FROM weather
GROUP BY origin, month
ORDER BY origin, month;
```

You could include year above, but since everything is 2013, it's not necessary.

```sql
SELECT month, round(avg(precip), 2)
FROM (SELECT origin, month, sum(precip) AS precip
      FROM weather
      GROUP BY origin, month) AS monthly
GROUP BY month
ORDER BY month;
```

## Numerical Distributions

Examine the distribution of temperature like we did with humidity.  Then do it by airport (origin).  

```sql
SELECT trunc(temp, -1), count(*)
FROM weather
GROUP BY trunc(temp, -1)
ORDER BY trunc(temp, -1);
```

```sql
SELECT origin, trunc(temp, -1), count(*)
FROM weather
GROUP BY origin, trunc(temp, -1)
ORDER BY origin, trunc(temp, -1);
```


## Date Part Functions

Which day of the week had the most rain at JFK airport?

```sql
SELECT date_part('dow', time_hour) AS dow, sum(precip) 
FROM weather
WHERE origin='JFK'
GROUP BY dow
ORDER BY sum(precip) DESC;
```

What week of the year in 2018 were the most food inspections done?

```sql
SELECT date_part('week', date) AS week, count(*)
FROM inspections
WHERE date >= '2018-01-01' AND date <= '2018-12-31'
GROUP BY week
ORDER BY count DESC;
```

Bonus (Hard, because we didn't learn everything you need): What days had no inspections?  The `generate_series()` function can be used to create a series of dates.  

Example:

```sql
SELECT generate_series('2018-01-01', '2019-05-14', '1 day'::interval) AS days;
```

The above are timestamps, but they can be compared to dates.

Can you use the above to help you find dates with no inspections?  


There are a few options:

```sql
SELECT date 
FROM generate_series('2018-01-01', '2019-05-14', '1 day'::interval) AS days
LEFT JOIN inspections
ON days = inspections.date
WHERE inspections.date IS NULL;
```

OR 

```sql
WITH days AS 
     (SELECT * 
      FROM generate_series('2018-01-01', 
         '2019-05-14', '1 day'::interval) AS day)
SELECT day
FROM days 
WHERE day NOT IN (SELECT date FROM INSPECTIONS);
```

What days of the week are days without inspections?  Any that aren't on the weekend?

These use the WITH option, but you can do this as well with other versions of the query.

```sql
WITH days AS (SELECT * FROM generate_series('2018-01-01', '2019-05-14', '1 day'::interval) AS day)
SELECT day, date_part('dow', day)
FROM days 
WHERE day NOT IN (SELECT date FROM INSPECTIONS);
```

```sql
WITH days AS (SELECT * FROM generate_series('2018-01-01', '2019-05-14', '1 day'::interval) AS day)
SELECT day, date_part('dow', day)
FROM days 
WHERE day NOT IN (SELECT date FROM INSPECTIONS) 
AND date_part('dow', day) NOT IN (0,6);
```



## Lag

We don't just have to lag dates themselves -- we can also lag other columns by their date values.  Let's get the changes in temperature from one hour to the next at JFK.  And the find the biggest change.  

Remember to limit to just rows for JFK, and order by time_hour overall.  And you'll want the absolute value of the difference with function `abs`.

```sql
WITH changes AS 
   (SELECT time_hour, temp,
       lag(temp) OVER (ORDER BY time_hour) AS lagged,
       abs(temp-lag(temp) OVER (ORDER BY time_hour)) AS diff
    FROM weather
    WHERE origin='JFK'
    ORDER BY time_hour)
SELECT time_hour, temp, lagged, diff 
FROM changes
WHERE diff = (SELECT max(diff) FROM changes);
```

This seems impossible, so let's take a look:

```sql
SELECT *
FROM weather
WHERE origin='JFK'
AND date_trunc('day', time_hour) = '2013-05-09';
```

Looks like bad data entry.


