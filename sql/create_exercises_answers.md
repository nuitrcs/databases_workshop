# Create Design Exercises
----

There may be other ways to achieve the same result.  Remember that SQL commands are not case sensitive (but data values are).

## Exercise: Create and Populate Basketball Tables

Using the example in [Part 3](part3.md#an-example), create the tables and then populate them with the data.  Either write insert statements or create csv files and import them.

Hint: If you make a mistake, you might want to delete tables and start over.  If you need help deleting a table, please ask a workshop assistant or look at the material in the next part of the workshop to see the delete and drop commands.


```sql
CREATE TABLE player (
	id int PRIMARY KEY,
	first_name text NOT NULL,
	last_name text NOT NULL,
	height smallint CHECK (height > 0), 
	weight smallint
);

CREATE TABLE team (
	id smallint primary key, 
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

#### Solution

Add in the data using insert statements.

```sql
INSERT INTO player -- don't need to specify columns if using all in order
VALUES (default, 'LeBron', 'James', 81, 249), -- default indicates to assign the value
(default, 'Tim', 'Duncan', 82, 256),
(default, 'Chris', 'Paul', 72, 175);

-- with the approach below, default is implied for the ID column
INSERT INTO team (name, city)
VALUES ('Cavaliers', 'Cleveland'),
('Heat', 'Miami'),
('Spurs', 'San Antonio'),
('Hornets', 'New Orleans'),
('Clippers', 'Los Angeles'),
('Rockets', 'Houston'),
('Lakers', 'Los Angeles'),
('Oklahoma City', 'Thunder');

-- check the ID values below before inserting
INSERT INTO player_team 
VALUES (1, 1, 2003, 2010),
(1, 2, 2010, 2014),
(1, 1, 2014, 2018),
(1, 7, 2018, NULL),
(2, 3, 1997, 2016),
(3, 4, 2005, 2011),
(3, 5, 2011, 2017),
(3, 6, 2017, 2019),
(3, 8, 2019, NULL);
```


## Exercise: Design a Database

Get the data from https://raw.githubusercontent.com/nuitrcs/databases_workshop/master/datafiles/allstudents.csv

Decide how to divide this data up into tables, create the tables, and import the data.

Some instructors are listed as TBD: decide how to handle this data.

The active column is 1 for true and 0 for false.  

Note: For importing the data, you can use `\copy` with `psql`, but if you're running `psql` on a remote server (as we do in in-person workshops), the files would need to be on that server (you can use `scp` if you know how).  Another option for importing data is to use DataGrip.  Right click on the table name, and choose Import Data from File.  There is a dialogue box then that you can use to map data from your file to a table.  

#### Solution

You might define the tables differently or choose different data types; this is one option.  The solution uses `smallint` for the ids instead of `serial` because the ID numbers are already given in the data.

These are just the create table options.  You'll need to divide the data file up and import it on your own (using `\copy` or DataGrip).  

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





