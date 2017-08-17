# SQL Part 3: Exercises
----

There may be other ways to achieve the same result.  Remember that SQL commands are not case sensitive (but data values are).

## Exercise: Create and Populate Basketball Tables

Using the example in [Part 3](part3.md#an-example), create the tables and then populate them with the data.  Either write insert statements or create csv files and import them.

Hint: If you make a mistake and want to clear everything in your database, do:

```
drop schema public cascade;
create schema public;
```

## Exercise: Design a Database

Get the data from https://raw.githubusercontent.com/nuitrcs/databases_workshop/master/datafiles/allstudents.csv

Decide how to divide this data up into tables, create the tables, and import the data.

Some instructors are listed as TBD: decide how to handle this data.

The active column is 1 for true and 0 for false.  

Note: For importing the data, you can use `\copy` with `psql`, but if you're running `psql` on a remote server (as we do in in-person workshops), the files would need to be on that server (you can use `scp` if you know how).  Another option for importing data is to use DataGrip.  Right click on the table name, and choose Import Data from File.  There is a dialogue box then that you can use to map data from your file to a table.  

