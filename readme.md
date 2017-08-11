# Databases

This workshop uses PostgreSQL.  Much of the material applies generically to SQL and other relational database systems, but some of it is specific to PostgreSQL.

This workshop uses the database discussed in, and follows much of the content of, the [PostgreSQL Tutorial](http://www.postgresqltutorial.com/).

The workshop starts with the presentation below, then [SQL Part 1](sql/part1.md).

## Presentation Materials

[![GitPitch](https://gitpitch.com/assets/badge.svg)](https://gitpitch.com/nuitrcs/databases_workshop/)

## Note

These materials are created for an in-person workshop.  Participants will be connecting to a database server that will only be active for the duration of the workshop.  In-person workshop participants do not need to install PostgreSQL.

Those wishing to work through the materials on their own will need to [install PostgreSQL](https://www.postgresql.org/download/) on their own systems or run a database instance/server via a cloud computing provider.  

See Section 1. Getting started with PostgreSQL, from the [PostgreSQL Tutorial](http://www.postgresqltutorial.com/) for details on how to set up your own database server.

## Supplementary Software

While in-person workshop participants do not need to install PostgreSQL, you will need a terminal program capable of creating an SSH connection to a remote server.  On a Mac, the built-in Terminal program will work.  On Windows, we suggest [PuTTY](http://www.putty.org/) if you don't already have another program installed.

Typing long commands in a terminal can be tedious.  We also recommend you install [DataGrip](https://www.jetbrains.com/datagrip/) for working with databases.  It has a free 30 day trial or you can apply for a JetBrains academic license for free.

This repository also includes materials for connecting to a database using Python or R.  For Python, you will need to install the `psycopg2` package.  For R, you will need the package `RPostgreSQL`.

## Resources

### Background

[Basic Explanation of Relational Databases](http://www.bbc.co.uk/education/guides/ztsvb9q/revision/1): from the BBC, a quick explanation of relational databases

### Additional Exercises/Tutorials

[PostgreSQL Exercises](https://pgexercises.com/): interactive, online exercises to practice SQL skills in a PostgreSQL environment.  

[SQL Tutorial](https://sqlzoo.net/): from SQL Zoo.  Not specific to PostgreSQL.  Also has interactive exercises. 

[Try SQL](https://www.codeschool.com/courses/try-sql): from Code School; the basic course is free.  Interactive, online tutorial.

[Intermediate SQL Tutorial](https://www.dataquest.io/blog/sql-intermediate/): intermediate level SQL tutorial from Dataquest; uses PostgreSQL and Python (pandas, psycopg2) and includes exercises.  For when you're ready for more practice beyond the basics.

### Other Options

The workshop uses the PostgreSQL database system, but if you're working on your own projects, [SQLite](https://www.sqlite.org/) may be a good option.  SQLite doesn't require running a server and it creates a database in a single, portable file locally on your computer.

