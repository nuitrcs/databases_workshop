# Creating and Designing Databases

When do you want to use a database?

* Want subsets of data or individual rows (not always entire dataset)
* Updating or changing subsets or individual rows, or adding data incrementally over time
* Need multiple people to have access to current data
* You have a lot of rectangles of data with some relationships between them


# Concepts

## Types of Tables

Let's look at the example `dvdrental` database http://www.postgresqltutorial.com/postgresql-sample-database and what types of tables are there.

There isn't really a standard classification of table types, but there are some common ones in practice.  Another person's take on this is at http://mark.random-article.com/musings/db-tables.html, and it aligns generally with what we have here, with some additional details.

* Main Entity tables: think nouns: holding observations about units/things/cases -- entities -- of interest
* Relationship tables: linking other tables together to define many-to-many relationships.  Note: One-to-many relationships can be captured through a foreign key column in the table that's the "many" in the one to many relationship: e.g. a city column in an address table.  Many addresses to one city, so the column linking these two is in the address table.
* Allowed value table or lookup table: contain the set of values that a field (column) in another table can take on.  Lookup tables usually don't change (at least much) during the life of a database -- they're fairly static.
* List table: Many to one tables that contain a set of acceptable values (like a lookup table) and maybe additional details about entities that can have many entities of another type associated with them.  Lookup tables usually have a fairly limited set of values, while these tables often have many more rows and are expected to grow over time.

There's another common type of table that isn't in the `dvdrental` database: an entity-attribute-value table.  Instead of having one column for each attribute of an entity you might want to record, there are tables that store key-value pairs associated with an entity.  This is useful when a full matrix of data would be sparsely filled, or when the set of keys isn't known at the start.

An example might be ministers/secretaries for major departments in a national government -- the set varies from country to country.  You would have at least three columns: country id, ministry/title, and an ID or name of the person.  You could do this instead of having a column for each type of minister.

A variation of this also comes up when you might have observations over time of a variable.  You might have the entity it's associated with, the data/time indicator, and a value.  For example, yearly GDP for countries.  You might have columns country id, year, and GDP value.  It's much easier to add entries to this over time than to change the structure of the table to keep adding new years over time.

## Normalization

First normal form is a property of relations in relational databases -- meaning it defines how data should be structured and related to each other.  It comes down generally to:

* No repeating groups: no duplicated columns or duplicated rows.
* The above implies that there is a unique key -- either a single column or combination of columns that uniquely identifies each row.
* The order of the rows or columns does not matter.
* One value per "cell" in a table (each column in a row)

There is also second and third normal forms (and more after that) that specify additional conditions on how data should be organized.  The resources at the end of the page link to more information.  



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
Chris | Paul | 72 | 175 | Rockets | Houston | 2017 | 2019
Chris | Paul | 72 | 175 | Thunder | Oklahoma City | 2019 | 


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
8 | Oklahoma City | Thunder



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
3 | 6 | 2017 | 2019
3 | 8 | 2019 | NULL


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

[Answers](create_exercises_answers.md)

### Exercise: Design a Database

Get the data from `https://raw.githubusercontent.com/nuitrcs/databases_workshop/master/datafiles/allstudents.csv`

(During the workshop, if connected to a remote server, the file is downloaded at `/tmp/allstudents.csv` already)

Decide how to divide this data up into tables, create the tables, and import the data.

Some instructors are listed as TBD: decide how to handle this data.

The active column is 1 for true and 0 for false. 

[Answers](create_exercises_answers.md)

# More Things to Know About

## Indexes

http://www.postgresqltutorial.com/postgresql-indexes/

When we set a primary key on a table, the database creates an index for that column.  We see this if we describe the table with `\d` is psql.  

We can also add additional indexes ourselves on columns.

An index makes it much faster to retrieve rows from the table using the columns that are part of the index -- so if you'd frequently filter on a column, a column can be useful.

But they also take up space, so there's a trade-off.  

And they make inserting data slower.

If we were to make an index on `player.last_name` from the basketball example:

```sql
CREATE INDEX idx_player_last_name 
ON player(last_name);  -- tablename(column_name)
```

This uses the common naming convention for the index of `idx` then table name then column name, but this isn't required.  

Or we create an index over multiple columns: 

```sql
CREATE INDEX idx_player_names 
ON player(last_name, first_name);
```

The order here matters.  When just `last_name` is used in a where clause, the index is still useful.  But not if only `first_name` is used.  

## Schemas

http://www.postgresqltutorial.com/postgresql-schema/

Schemas are collections of objects (tables, functions, indexes, sequences, etc.) in a database.  It's another prefix that may go on an object to identify it: `schema.table.column`.  

Schemas are useful for: 

1) organization
2) giving different permissions to different users (although you can do this by table as well)




## Functions

We've used functions that are built into the database, but you can also write your own functions.  There's a language specific to postgres, but there are also ways to write functions in R, perl, and other languages directly in the database.  But the database admin has to enable this functionality, and the other language plug-ins can be a security risk.

## Triggers

http://www.postgresqltutorial.com/postgresql-triggers/

Triggers run a specified function whenever a specific event, such as an insert, update, or delete, occurs.  They are useful for enforcing data constraints that you can't do with the other constraints we discussed.  

They can be used to log changes to one table in another (who made a change and when).


## Slow Bulk Inserts

Table constraints, from primary keys to foreign keys to check constraints, can make inserting data into a database much slower because more work has to be done to verify that the values are allowed.  (Under some cases, such as with foreign keys, deleting data can also be slower.)  This isn't a huge issue if you're inserting a few rows at a time.  But if you're importing a lot of data, it can be useful to strip a table of its constraints and then add them back in later.  But then you need to assume responsibility for data integrity.  There's no command to do this easily in PostgreSQL.  

The same applies to triggers, but there is a way to disable and enable triggers:

```sql
ALTER TABLE tbl_name DISABLE TRIGGER ALL;
-- do something here 	
ALTER TABLE tbl_name ENABLE TRIGGER ALL;
```

## Views

Views can make your life easier by making queries simpler.  If there are common joins, having a view to reference instead can be useful.



# Another Example

We'll work through this together.  Below are some notes to guide discussion.

We're going to use data from https://data.stanford.edu/congress\_text which is made available under the ODC Attribution License. [Codebook](https://stacks.stanford.edu/file/druid:md374tz9962/codebook_v2.pdf)

This is data on speeches made in congress -- from the official congressional record.  They have processed the data to have word and phrase (bigram) counts as well.  There's some metadata about the speeches and the speakers too. 

Some of these are really big files, so we're going to talk about what's in them.  The zip files that don't start with "hein" are small and OK to download.  

`phrase_clusters.zip` has three files that link terms to topics - both good terms and bad matches to exclude

`vocabulary.zip` has three files that define phrases used in the analysis.

`speakermap_stats.zip` has two files that appear to be statistics about the quality of the data processing - probably not relevant here.

`audit.zip` (don't need to download) has data on the data processing - shouldn't be needed in a database for analysis

`hein-daily.zip` (don't download) has files for: speeches (just an ID column and a text column: speech\_id, speech), descriptions (see below), bigram counts by speaker and party (speakerid, phrase, count OR party (R, D, or I), phrase, count), speakermap files that link the speech to the speaker with info about the speaker (see [114_SpeakerMap.txt](../data/114_SpeakerMap.txt)). 

description files, which contain details on the raw extractions of metadata about the speeches:

```
speech_id|chamber|date|number_within_file|speaker|first_name|last_name|state|gender|line_start|line_end|file|char_count|word_count
1140000001|None|20150106|00001|The BECERRA|Unknown|BECERRA|Unknown|M|000918|000958|01062015.txt|1358|244
1140000010|H|20150106|00010|The CLERK|Unknown|Unknown|Unknown|Special|000959|000966|01062015.txt|236|36
1140000011|H|20150106|00011|Mr. MASSIE|Unknown|MASSIE|Unknown|M|000967|000973|01062015.txt|245|41
1140000012|H|20150106|00012|The CLERK|Unknown|Unknown|Unknown|Special|000974|000975|01062015.txt|30|4
1140000013|H|20150106|00013|Mr. BRIDENSTINE|Unknown|BRIDENSTINE|Unknown|M|000976|001006|01062015.txt|1013|183
1140000014|H|20150106|00014|The CLERK|Unknown|Unknown|Unknown|Special|001007|001009|01062015.txt|30|4
1140000015|H|20150106|00015|Mr. KING of Iowa|Unknown|KING|Iowa|M|001010|001027|01062015.txt|559|92
1140000016|H|20150106|00016|The CLERK|Unknown|Unknown|Unknown|Special|001028|001046|01062015.txt|603|88
1140000017|H|20150106|00017|Ms. PELOSI|Unknown|PELOSI|Unknown|F|001639|001822|01062015.txt|5780|1002
1140000018|H|20150106|00018|Mr. BOEHNER|Unknown|BOEHNER|Unknown|M|001823|001956|01062015.txt|4148|765
1140000019|H|20150106|00019|Mr. CONYERS|Unknown|CONYERS|Unknown|M|001957|001970|01062015.txt|491|86
...
```

`hein-bound.zip` (don't download) has the same types of files as in `hein-daily.zip`.  They are from a slightly different version of the data source than the daily files.  We'll ignore these for this workshop, as they have similar structure to the files above.


Where do we start?

What are our main units of observation?  Our entity tables?  From the description, we should be looking at something related to speeches and something related to speakers.  Where does this data exist in the data files?  What uniquely identifies a speaker?

What is the relationship between speeches and speakers?  One to many?  Many to many?

What other data do we want to keep?  The aggregate counts?  The topical mappings?  Will we be doing our own counts, or using their bigrams?

What other data would we like to add into the database to make things easier?

How will we be searching the text (and how often)?  You can do full text index for searching within postgres, but the search syntax is a bit strange.  

From a practical standpoint, how would we put this all together?  You'd want to use R or Python to do some of the data manipulation and extraction, and then to put the data in the database (or write CSV files that correspond to table for import).

What can we do:

* Write a table definition for speaker
* Write a table definition for speech
* What are some of the lookup tables we might use?
* Do we need some metadata/detail tables?
* Term and topic tables - write definitions

# Resources

Some additional links for database design.

## Normalization and overall design advice

https://support.office.com/en-us/article/database-design-basics-eb2159cf-1e30-401a-8084-bd4f9c9ca1f5

https://www.1keydata.com/database-normalization/ 

https://www.guru99.com/database-normalization.html

## Common mistakes

https://www.red-gate.com/simple-talk/sql/database-administration/five-simple-database-design-errors-you-should-avoid/


