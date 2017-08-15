# SQL Part 3

Creating tables, importing and exporting data

This part requires you to connect to a database where you have permissions to create new tables.

# `CREATE TABLE`

The basic syntax of creating a table is:

```sql
CREATE TABLE table_name (
 column_name TYPE column_constraint,
 column_name TYPE column_constraint,
 column_name TYPE column_constraint,
 CONSTRAINT table_constraint
);
```

Constraints are conditions on one or more columns that link tables together and control what values a column can take.

Let's start with a concrete example.  We'll make a table for students.

```sql
CREATE TABLE student (
	id serial PRIMARY KEY,
	first_name text,
	last_name text,
	admission_year smallint 
);
```

A common style convention is to name tables with the singular form of a noun, not plural.  Tables and columns are named in all lowercase, with underscores as necessary to separate words.  People differ on whether `id` columns should just be `id` or `tablename_id` (so in this case, `student_id` vs. `id`).

Breaking this down piece by piece:

* `serial` is an integer that autoincrements.  It's a good choice when you want an ID column generated for you.  
* `PRIMARY KEY` means that the column in `NOT NULL` and `UNIQUE` -- so each row has a unique value on this column.  PostgreSQL will generate a name for this primary key constraint for you (you don't really need the name for most things you'll do).  An index will also be created from this column, which makes looking up rows in the table with values of this column more efficient.  
* `first_name` and `last_name` are type text.  Text isn't a standard SQL type, so if you'll be working across different systems, you might use `varchar` or `char` instead.  With these, you'd specify the number of characters, but with text you don't.  
* `admission_year` is a `smallint` which takes the range -32768 to 32767 (2 bits).  This range is sufficient for a year.  There are also `int` and `float` among some others.

Every table should have a primary key, although this isn't enforced in PostgreSQL.  You can use a combination of columns as the primary key instead of just defining it on one of them.

What does the describe command tell us about our new table:

```sql
practice=# \d student
                              Table "public.student"
     Column     |   Type   |                      Modifiers                       
----------------+----------+------------------------------------------------------
 id             | integer  | not null default nextval('student_id_seq'::regclass)
 first_name     | text     | 
 last_name      | text     | 
 admission_year | smallint | 
Indexes:
    "student_pkey" PRIMARY KEY, btree (id)
```

You can see the primary key index that was made, and the student id sequence.


## Adding Values

### Insert

How do we add data to this table?  We can issue insert commands:

```sql
INSERT INTO student (first_name, last_name, admission_year) 
VALUES 
	('Alice', 'Walker', 2017),
	('Bob', 'Williams', 2016),
	('Charlie', 'Weston', 2016);
```

Let's check:

```sql
SELECT * FROM student;
```

```sql
practice=# select * from student;
 id | first_name | last_name | admission_year 
----+------------+-----------+----------------
  1 | Alice      | Walker    |           2017
  2 | Bob        | Williams  |           2016
  3 | Charlie    | Weston    |           2016
(3 rows)
```

The id column was generated for us, starting at 1.  


### `copy`

We can also [copy](https://www.postgresql.org/docs/current/static/sql-copy.html) data from a file.  Get the CSV file from: and save it somewhere you know the path to.

```sql
COPY student (first_name, last_name, admission_year) -- columns
FROM 'studentdata.csv' -- file (full path as needed)
CSV HEADER; -- CSV format (= comma delimiter), and there's a header in the file
```

Note: With the `psql` client, you generally need to use `\copy` instead of copy.  It uses the same general format.


# An Example

We had a primary key constraint above, but there are other constraints.  The most important is foreign key, which links tables together.  But we can also enforce other constraints.

## Database Design

Let's work through an example of players and teams.  When we're designing our database, we want to avoid duplication as much as possible.

First Name  | Last Name  | Height  | Weight  | TeamName  | TeamCity  | StartingYear  | EndingYear 
:---|:---|---|---|:---|:---|---|---
LeBron  | James  | 81 | 249 | Cavaliers  | Cleveland  | 2003 | 2010
LeBron  | James  | 81 | 249 | Heat  | Miami  | 2010 | 2014
LeBron  | James  | 81 | 249 | Cavaliers  | Cleveland  | 2014 | 
Tim  | Duncan  | 82 | 256 | Spurs  | San Antonio  | 1997 | 2016
Chris  | Paul  | 72 | 175 | Hornets  | New Orleans  | 2005 | 2011
Chris  | Paul  | 72 | 175 | Clippers  | Los Angeles  | 2011 | 2017
Chris | Paul | 72 | 175 | Rockets | Houston | 2017 | 
Greg  | Monroe  | 83 | 250 | Pistons  | Detroit  | 2010 | 2015
Greg  | Monroe  | 83 | 250 | Bucks  | Milwaukee  | 2015 | 

What tables could we make to hold this data?

* player
* team
* player_team

Player:

id | first_name | last_name | height | weight
---|---|---|---|---
1 | LeBron | James | 81 | 249
2 | Tim | Duncan | 82 | 256 
3 | Chris | Paul | 72 | 175
4 | Greg | Monroe | 83 | 250

Team:

id | name | city 
---|---|---
1 | Cavaliers | Cleveland
2 | Heat | Miami
3 | Spurs | San Antonio
4 | Hornets | New Orleans
5 | Clippers | Los Angeles
6 | Rockets | Houston
7 | Pistons | Detroit
8 | Bucks | Milwaukee



Player-Team:
player_id | team_id | starting_year | ending_year
---|---|---|---
1 | 1 | 2003 | 2010
1 | 2 | 2010 | 2014
1 | 1 | 2014 | NULL
2 | 3 | 1997 | 2016
3 | 4 | 2005 | 2011
3 | 5 | 2011 | 2017
3 | 6 | 2017 | NULL
4 | 7 | 2010 | 2015
4 | 8 | 2015 | NULL


## Table Creation

First, player table.  Start with the data types.

```sql
CREATE TABLE player ( 
	id SERIAL,
	first_name TEXT, 
	last_name TEXT, 
	height SMALLINT, 
	weight SMALLINT
);
```

Add in constraints

```sql
CREATE TABLE player (
	id serial PRIMARY KEY,
	first_name text NOT NULL,
	last_name text NOT NULL,
	height smallint CHECK (height > 0), 
	weight smallint
);
```

We use primary key like above, and also set the name columns to be not null.  We can also add in value constraints like ensuring that height is a positive number.

Moving on to the team table:

```sql
CREATE TABLE team (
	id smallserial primary key, 
	name text not null,
	city text not null,
	unique (name, city)
);
```

For team, we added a constraint across two columns, saying that the combination of team name and city needs to be unique.

For player-team, start with types:

```sql
CREATE TABLE player_team ( 
	player_id int,
	team_id smallint, 
	start_year smallint, 
	end_year smallint
);
```

Add in constraints, a default value, and link ids to the other tables:

```sql
CREATE TABLE player_team (
	player_id int REFERENCES player(id),
	team_id smallint REFERENCES team(id), 
	start_year smallint NOT NULL, 
	end_year smallint DEFAULT NULL
);
```

The id columns, REFERENCES establishes a foreign key that constrains the values of `player_id` to be values that exist in the player table, id column.  For `team_id`, it has to take on values in the team table, id column.

An alternative format to specify the foreign keys is:

```sql
CREATE TABLE player_team (
	player_id int,
	team_id smallint, 
	start_year smallint not null, 
	end_year smallint default null,
	FOREIGN KEY (player_id) REFERENCES player(id),
	FOREIGN KEY (team_id) REFERENCES team(id)
);
```

We also set the start year to not be null, but for the end year, it defaults to null (this is the default default, but we're being explicit).

Our player-team table is still missing a primary key.  We can't use player id and team id because players have left teams and later come back to them.  So we could use 3 columns: player id, team id, and start year:

```sql
CREATE TABLE player_team (
	player_id int references player(id),
	team_id smallint references team(id), 
	start_year smallint not null, 
	end_year smallint default null,
	PRIMARY KEY (player_id, team_id, start_year)
);
```

We could also just add an ID column to the table instead.  Having a primary key helps keep you from getting into a situation where you can't easily delete duplicate values from a table (or accidently introduce them in the first place).



# Exporting Data

You can use [Copy](https://www.postgresql.org/docs/current/static/sql-copy.html) (or `\copy`) to export data too.  You have to specify an absolute file path.

```sql
COPY student 
TO '/Users/username/documents/mystudents.csv'
CSV HEADER;
```

In `psql` there is also a `\o` command to open a file for writing (and then close it later).  

