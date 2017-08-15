# SQL Part 4

Transactions

Update, delete, alter, drop


# Transactions

SQL commands are executed immediately, unless you put them in a transaction.  With select statements, this doesn't make much of a difference.  But when altering the database, you often want to make sure commands execute correctly before committing the change to the database.  

Transactions begin with `begin;` and end with either `commit;` to commit the change or `rollback;` to undo everything since `begin`.  If there is an error in a transaction, you can't commit it; you have to use rollback to roll it back and then try again.   


# Changing and Deleting Tables

## Alter

You can [alter](https://www.postgresql.org/docs/current/static/sql-altertable.html) a table after it's created.  Things you can do include: renaming the table, renaming columns, changing column types, adding or dropping constraints, adding or changing default values.

## Drop

To delete a table, use the drop command.  This cannot be undone, so best to really mean it and do it in a transaction.