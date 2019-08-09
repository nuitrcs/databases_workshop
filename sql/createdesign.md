# Creating and Designing Databases

# Concepts

## Types of Tables

Let's look at the example `dvdrental` database http://www.postgresqltutorial.com/postgresql-sample-database and what types of tables are there.

There isn't really a standard classification of table types, but there are some common ones in practice.  Another person's take on this is at http://mark.random-article.com/musings/db-tables.html, and it aligns generally with what we have here, with some additional details.

* Main Entity tables: think nouns: holding observations about units/things/cases -- entities -- of interest
* Relationship tables: linking other tables together to define many-to-many relationships
* Allowed value table or lookup table: contain the set of values that a field (column) in another table can take on.  Lookup tables usually don't change (at least much) during the life of a database -- they're fairly static.
* List table: Many to one tables that contain a set of acceptable values (like a lookup table) and maybe additional details about entities that can have many entities of another type associated with them.  Lookup tables usually have a fairly limited set of values, while these tables often have many more rows and are expected to grow over time.

There's another common type of table that isn't in the `dvdrental` database: an entity-attribute-value table.  Instead of having one column for each attribute of an entity you might want to record, there are tables that store key-value pairs associated with an entity.  This is useful when a full matrix of data would be sparsely filled, or when the set of keys isn't known at the start.

An example might be ministers/secretaries for major departments in a national government -- the set varies from country to country.  You would have at least three columns: country id, ministry/title, and an ID or name of the person.  You could do this instead of having a column for each type of minister.

A variation of this also comes up when you might have observations over time of a variable.  You might have the entity it's associated with, the data/time indicator, and a value.  For example, yearly GDP for countries.  You might have columns country id, year, and GDP value.  It's much easier to add entries to this over time that to change the structure of the table to keep adding new years over time.

## Normalization

First normal form is a property of relations in relational databases -- meaning it defines how data should be structured and related to each other.  It comes down generally to:

* No repeating groups: no duplicated columns or duplicated rows.
* The above implies that there is a unique key -- either a single column or combination of columns that uniquely identifies each row.
* The order of the rows or columns does not matter.
* One value per "cell" in a table (each column in a row)

There is also second and third normal forms that specify additional conditions on how data should be organized.  See more here: https://www.1keydata.com/database-normalization/ 



# Creating a Database

When you start the PostgreSQL database server, it will make a few databases by default.  One will be called template1, another will likely be called postgres, and there will likely be a third named after your username on the system.

You can create a new database with the command line utility [`createdb`](https://www.postgresql.org/docs/11/app-createdb.html), or from within an existing database with the [`create database`](https://www.postgresql.org/docs/11/sql-createdatabase.html) statement.

At a minimum, you'll need to name the database.  By default, the current user becomes the owner of the created database (owner has full permissions).

Examples:

Command line (not in the database)

```sql
createdb -h localhost workshop
```

From within a different database (such as template1)

```sql
CREATE DATABASE workshop;
```

You can create a database using another database as a template -- this copies the schema (structure) but not the data.

## Where to Host?

While you're learning, you may want to run a database server directly on your own machine.  However, if you plan to share the database with anyone, or want to run a process that will interact with the database on it's own, you'll probably want to set up a database in the cloud instead from the start.

You can dump the contents of a database to file, and then use that file to restore those contents back to another database in the future.  But while this works in theory, in practice there can always be little things that go wrong.  Also, this single file can get really large in size, which makes transfer difficult.  

Dumping a database is a good way to save a copy of a database you are administering/running yourself.  On cloud services, there are ways to automate backups and take snapshots of the database (which is slightly different than dumping the files).

See [`pg_dump`](https://www.postgresql.org/docs/11/app-pgdump.html) and [`pg_restore`](https://www.postgresql.org/docs/11/app-pgrestore.html).




# Creating Tables

The basic syntax of creating a table is:

```sql
CREATE TABLE table_name (
 column_name TYPE COLUMN_CONSTRAINT,
 column_name TYPE COLUMN_CONSTRAINT,
 column_name TYPE COLUMN_CONSTRAINT,
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

[PostgreSQL Types](https://www.postgresql.org/docs/current/static/datatype.html)

Primary key = unique and not null, which are two separate constraints you can also put on columns.  

[Column Constraints](https://www.postgresql.org/docs/11/ddl-constraints.html)

# Constraints

We had a primary key constraint above, but there are other constraints.  The most important is foreign key, which links tables together.  But we can also enforce other constraints.

## Database Design

Let's work through an example of players and teams.  When we're designing our database, we want to avoid duplication.

First Name  | Last Name  | Height  | Weight  | TeamName  | TeamCity  | StartingYear  | EndingYear 
:---|:---|---|---|:---|:---|---|---
LeBron | James | 81 | 249 | Cavaliers | Cleveland | 2003 | 2010
LeBron | James | 81 | 249 | Heat | Miami | 2010 | 2014
LeBron | James | 81 | 249 | Cavaliers | Cleveland | 2014 | 2018
LeBron | James | 81 | 249 | Lakers | Los Angeles | 2018 | 
Tim | Duncan | 82 | 256 | Spurs | San Antonio | 1997 | 2016
Chris | Paul | 72 | 175 | Hornets | New Orleans | 2005 | 2011
Chris | Paul | 72 | 175 | Clippers | Los Angeles | 2011 | 2017
Chris | Paul | 72 | 175 | Rockets | Houston | 2017 | 


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


Team:

id | name | city 
---|---|---
1 | Cavaliers | Cleveland
2 | Heat | Miami
3 | Spurs | San Antonio
4 | Hornets | New Orleans
5 | Clippers | Los Angeles
6 | Rockets | Houston
7 | Lakers | Los Angeles



Player-Team:

player_id | team_id | starting_year | ending_year
---|---|---|---
1 | 1 | 2003 | 2010
1 | 2 | 2010 | 2014
1 | 1 | 2014 | 2018
1 | 7 | 2018 | NULL
2 | 3 | 1997 | 2016
3 | 4 | 2005 | 2011
3 | 5 | 2011 | 2017
3 | 6 | 2017 | NULL


(We've simplified above to use the year and not the full date; we'd want a full date in reality because players can leave and rejoin a team during a season.)

## Table Creation

First, player table.  Start with the data types.

```sql
CREATE TABLE player ( 
	id INT,
	first_name TEXT, 
	last_name TEXT, 
	height SMALLINT, 
	weight SMALLINT
);
```

We're making the player id an imt type because we are assigning our own ids.  You could use a serial to automatically assign ID numbers.  That wouldn't prevent us from supplying our own ids, but doing so would potentially conflict with the autogenerated sequence ([more info](https://stackoverflow.com/questions/244243/how-to-reset-postgres-primary-key-sequence-when-it-falls-out-of-sync)).  Since we have to link players to teams later, we'll need to know what both the player ID and team ID are before we do the linking.  So we make this column just an int instead and assign IDs ourselves.  The downside of that is keeping track of what ids have already been used.

Add in constraints

```sql
CREATE TABLE player (
	id int PRIMARY KEY,
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
	id smallint primary key, 
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

We also set the start year to not be null, but for the end year, it defaults to null (this is the default, but we're being explicit).

Our player-team table is still missing a primary key.  We can't use player id and team id because players have left teams and later come back to them.  So we could use 3 columns: player id, team id, and start year (again, ignoring the possibility that a player leaves a team and rejoins in the same year):

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

### Exercise: Create and Populate Basketball Tables

Using the example of a basketball team, create the tables and then populate them with the data.  Either write insert statements or create csv files and import them.

Need to see how to insert data?  Take a look at [Part 3](part3.md).

Hint: If you make a mistake, you might want to delete tables and start over.  See [Part 4](part4.md) to remember how.

### Exercise: Design a Database

Get the data from https://raw.githubusercontent.com/nuitrcs/databases_workshop/master/datafiles/allstudents.csv

Decide how to divide this data up into tables, create the tables, and import the data.

Some instructors are listed as TBD: decide how to handle this data.

The active column is 1 for true and 0 for false. 


# Resources

https://www.red-gate.com/simple-talk/sql/database-administration/five-simple-database-design-errors-you-should-avoid/


