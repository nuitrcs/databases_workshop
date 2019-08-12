# SQL Part 4: Exercises
----

There may be other ways to achieve the same result.  Remember that SQL commands are not case sensitive (but data values are).

## Exercise: Alter

Create and populate the `food` table below using the commands provided. 

Then add a new text column `color`.

Read how to alter a table by [changing a column name](http://www.postgresqltutorial.com/postgresql-rename-column/) (or [official documentation](https://www.postgresql.org/docs/current/static/sql-altertable.html)).  Then rename the `color` column you just created to `primary_color`.

```sql
CREATE TABLE food (
	id serial primary key,
	name text not null,
	type text,
	favorite boolean default false	
);

INSERT INTO food (name, type) 
VALUES 
	('broccoli','vegetable'), 
	('lime', 'fruit'), 
	('green beans', 'vegetable'), 
	('milk', 'dairy'), 
	('yogurt', 'dairy'), 
	('banana', 'fruit'), 
	('lemon', 'fruit'), 
	('tortilla', 'carbohydrate'), 
	('rice', 'carbohydrate');
``` 


#### Solution

```sql
ALTER TABLE food ADD COLUMN color text;
ALTER TABLE food RENAME color TO primary_color;
```

## Exercise: Update

Using the food table created and altered above, set the values of the `primary_color` column.  Then set the values of the `favorite` column based on your favorites.


#### Solution

Assuming that your table is starting looking like 

```sql
select * from food;
 id |    name     |     type     | favorite | primary_color 
----+-------------+--------------+----------+---------------
  1 | broccoli    | vegetable    | f        | 
  2 | lime        | fruit        | f        | 
  3 | green beans | vegetable    | f        | 
  4 | milk        | dairy        | f        | 
  5 | yogurt      | dairy        | f        | 
  6 | banana      | fruit        | f        | 
  7 | lemon       | fruit        | f        | 
  8 | tortilla    | carbohydrate | f        | 
  9 | rice        | carbohydrate | f        | 
(9 rows)
```

Then, a few different ways you could do updates:

```sql
UPDATE food SET primary_color='green' 
WHERE name IN ('broccoli', 'lime', 'green beans');

-- or 
UPDATE food SET primary_color='green' 
WHERE id <= 3;
```
```sql
UPDATE food SET primary_color='white' 
WHERE id IN (4, 5, 8, 9);

-- next statement dependent on the ones above having been run
UPDATE food SET primary_color='yellow' 
WHERE primary_color IS NULL AND type='fruit';
```

```sql
UPDATE food SET favorite='t' 
WHERE id IN (2, 8);
```




## Exercise: Update with Join


Create and populate tables using the supplied code below.

Set the value of `last_taught` in `course` to the most recent date the course was taught using the `course_offering` table.

Hint: you'll need to join to a subquery (the results of another query).  Think first about how to get the most recent date for each course, and then how to use that information in the update.  Alternatively, create a temporary table with the results of the query, then write an update statement using the temporary table.

```sql
CREATE TABLE course (
	id int primary key,
	name text not null,
	last_taught date
);

INSERT INTO course (id, name) 
VALUES (1, 'Chemistry'),
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
VALUES (1, 'Spring 2015', '2015-03-01'),
	(1, 'Spring 2017', '2017-03-01'),
	(2, 'Fall 2016', '2016-09-01'),
	(2, 'Spring 2017', '2017-03-01'),
	(3, 'Spring 2016', '2016-03-01'),
	(4, 'Winter 2015', '2015-01-01'),
	(4, 'Winter 2017', '2017-01-01'),
	(4, 'Winter 2016', '2016-01-01');
```


#### Solution

```sql
UPDATE course 
SET last_taught = maxdate 
FROM (SELECT course_id, max(date) AS maxdate 
		FROM course_offering
		GROUP BY course_id) foo
WHERE 
	id=course_id;
```


## Exercise: Delete

Using the table created, altered, and updated above, delete any white foods that aren't a favorite.

Using the `course` table created above, delete any courses that were last offered before 2017 (start date before 2017).  Note that you'll also need to delete entries from course\_offering table too.  Be careful not to delete old offerings of courses you aren't deleting.


#### Solutions

```sql
DELETE FROM food 
WHERE primary_color='white' 
AND NOT favorite;
```

One option (you could also do this with USING):

```sql
ALTER TABLE course
DROP last_taught;

DELETE FROM course_offering
WHERE course_id NOT IN 
 (SELECT course_id 
  FROM course_offering
  WHERE date >= '2017-01-01');

DELETE FROM course
WHERE id NOT IN
 (SELECT course_id 
  FROM course_offering);
```

