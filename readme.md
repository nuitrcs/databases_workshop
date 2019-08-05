# Databases

There are four databases workshops that cover parts of the material in this repo.

The workshops use PostgreSQL.  Much of the material applies generically to SQL and other relational database systems, but some of it is specific to PostgreSQL.

Some of the workshops use the database discussed in, and follow much of the content of, the [PostgreSQL Tutorial](http://www.postgresqltutorial.com/), which makes use of the [Skalia database](https://www.jooq.org/sakila).


## Selecting and Joining Data

[Intro presentation](https://gitpitch.com/nuitrcs/databases_workshop/)

The workshop follows the files below, which have links to the exercises.

[Part 1](sql/part1.md)

[Part 2](sql/part2.md)



## Exploring Data

See [Exploring Data](sql/exploring.md) for materials and exercises.

This workshop uses data files from the [nycflights13 R package](https://github.com/hadley/nycflights13) and open data from the city of Evanston.

## Updating and Changing Data

Uses materials from [Part 4](sql/part4.md), which are in the process of being updated/expanded.

## Creating and Designing Databases

[Part 3](sql/part3.md) materials are relevant, but materials for this workshop are currently being updated/expanded.  

## R and Python

The [R](/r) and [Python](/python) materials may also be of interest to those taking any of the above workshops.


## Note

These materials are created for an in-person workshop.  Participants will be connecting to a database server that will only be active for the duration of the workshop.  In-person workshop participants do not need to install PostgreSQL.

Those wishing to work through the materials on their own will need to [install PostgreSQL](https://www.postgresql.org/download/) on their own systems or run a database instance/server via a cloud computing provider.  

See Section 1. Getting started with PostgreSQL, from the [PostgreSQL Tutorial](http://www.postgresqltutorial.com/) for details on how to set up your own database server.

## Supplementary Software

While in-person workshop participants do not need to install PostgreSQL, you will need a terminal program capable of creating an SSH connection to a remote server.  On a Mac, the built-in Terminal program will work.  On Windows, we suggest [PuTTY](http://www.putty.org/) if you don't already have another program installed.

This repository also includes materials for connecting to a database using Python or R.  For Python, you will need to install the `psycopg2` package.  For R, you will need the package `RPostgres`.

# Resources

## Background

[Basic Explanation of Relational Databases](http://www.bbc.co.uk/education/guides/ztsvb9q/revision/1): from the BBC, a quick explanation of relational databases

## Software

[DataGrip Tutorial](https://www.youtube.com/watch?v=Xb9K8IAdZNg): video on how to use the DataGrip program; it even uses the same database we use in this workshop.

## Reference

[PostgreSQL cheat sheet](http://www.postgresqltutorial.com/wp-content/uploads/2018/03/PostgreSQL-Cheat-Sheet.pdf): a list of basic commands and patterns for statements

[PostgreSQL Documentation](https://www.postgresql.org/docs/current/static/index.html): official documentation 

[dvdrental Diagram](http://www.postgresqltutorial.com/wp-content/uploads/2018/03/printable-postgresql-sample-database-diagram.pdf): entity-relationship diagram for the database used in the workshop

[psql commands cheat sheet](http://www.postgresonline.com/downloads/special_feature/postgresql83_psql_cheatsheet.pdf): describe commands, other slash commands

## Additional Exercises/Tutorials

_These resources use PostgreSQL or SQL generally._

[Mode SQL Tutorials](https://mode.com/sql-tutorial/): good introduction, and you can try running queries on their platform

[PostgreSQL Exercises](https://pgexercises.com/): interactive, online exercises to practice SQL skills in a PostgreSQL environment.  

[SQL Tutorial](https://sqlzoo.net/): from SQL Zoo.  Not specific to PostgreSQL.  Also has interactive exercises. 

[Learn SQL](https://www.codecademy.com/courses/learn-sql/) from Code Academy with a free interactive online course

[Try SQL](https://www.codeschool.com/courses/try-sql): from Code School; the basic course is free.  Interactive, online tutorial.

[Intermediate SQL Tutorial](https://www.dataquest.io/blog/sql-intermediate/): intermediate level SQL tutorial from Dataquest; uses PostgreSQL and Python (pandas, psycopg2) and includes exercises.  For when you're ready for more practice beyond the basics.

[Welcome to SQL](https://www.khanacademy.org/computing/hour-of-code/hour-of-sql/v/welcome-to-sql) from Khan Academy provides a one-hour intro to databases that includes writing SQL to create tables, insert data, and query data.

[Intro to SQL: Querying and managing data](https://www.khanacademy.org/computing/computer-programming/sql) from Khan Academy

[Intro to SQL](https://www.kaggle.com/learn/intro-to-sql) from Kaggle uses Google's BigQuery instead of PostgreSQL.  Commands are generally standard SQL though.

[SQL for Data Analysis](https://www.udacity.com/course/sql-for-data-analysis--ud198#) from Udacity - an online self-paced course


