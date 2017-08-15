# SQL Part 3: Exercises
----

There may be other ways to achieve the same result.  Remember that SQL commands are not case sensitive (but data values are).

## Exercise: Create and Populate Basketball Tables

Using the example in [Part 3](part3.md), create the tables and then populate them with the data.  Either write insert statements or create csv files and read them in.

Hint: If you make a mistake and want to clear everything in your database, do:

```
drop schema public cascade;
create schema public;
```

#### Solution

```sql
CREATE TABLE player (
	id serial PRIMARY KEY,
	fname text NOT NULL,
	lname text NOT NULL,
	height smallint CHECK (height > 0), 
	weight smallint
);

CREATE TABLE team (
	id smallserial primary key, 
	name text not null,
	city text not null,
	unique (name, city)
);

CREATE TABLE player_team (
	player_id int references player(id),
	team_id smallint references team(id), 
	start_year smallint not null, 
	end_year smallint default null,
	PRIMARY KEY (player_id, team_id, start_year)
);
```

Add in the data using insert statements.

```sql
INSERT INTO player -- don't need to specify columns if using all in order
VALUES (1 , ' LeBron ', ' James ',  81 ,  249),
(2 , ' Tim ', ' Duncan ',  82 ,  256 ),
(3 , ' Chris ', ' Paul ',  72 ,  175),
(4 , ' Greg ', ' Monroe ',  83 ,  250);

INSERT INTO team
VALUES (1 , ' Cavaliers ', ' Cleveland'),
(2 , ' Heat ', ' Miami'),
(3 , ' Spurs ', ' San Antonio'),
(4 , ' Hornets ', ' New Orleans'),
(5 , ' Clippers ', ' Los Angeles'),
(6 , ' Rockets ', ' Houston'),
(7 , ' Pistons ', ' Detroit'),
(8 , ' Bucks ', ' Milwaukee');

INSERT INTO player_team 
VALUES (1 ,  1 ,  2003 ,  2010),
(1 ,  2 ,  2010 ,  2014),
(1 ,  1 ,  2014 ,  NULL),
(2 ,  3 ,  1997 ,  2016),
(3 ,  4 ,  2005 ,  2011),
(3 ,  5 ,  2011 ,  2017),
(3 ,  6 ,  2017 ,  NULL),
(4 ,  7 ,  2010 ,  2015),
(4 ,  8 ,  2015 ,  NULL);
```


## Exercise: Design a Database

Get the data from 

Decide how to divide this data up into tables, create the tables, and import the data.

Some instructors are listed as TBD: decide how to handle this data.

The active column is 1 for true and 0 for false.  Figure out how to import this as a boolean.

#### Solution

You might define the tables differently or choose different data types; this is one option.  

These are just the create table options.  You'll need to divide the data file up and import it on your own (use copy or `\copy`).

```sql
create table instructor (
	id smallint primary key,
	firstname text,
	lastname text,
	title text
);

create table course (
	id smallint primary key,
	title text,
	startdate date,
	instructor_id smallint references instructor(id)
);

create table student (
	id smallint primary key,
	admission_year smallint,
	active boolean
);

create table grade (
	student_id smallint references student(id),
	course_id smallint references course(id),
	grade decimal(2,1), -- could just use a float or numeric
	primary key (student_id, course_id) -- assumes students can't retake courses
);

```





