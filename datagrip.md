# DataGrip Notes

When you open up DataGrip for the first time, you will likely have a blank window.  Open the Manage Data Sources view (File > Data Sources).  This will open a new subwindow with menus on the left.  

Click the + button to add a database.  The + button is a pull-down menu; choose Data Source, then PostgreSQL.  

By default, you'll have a Data Source with a name like postgres@localhost or something similar.  Information to fill in:

* Name: your choice, something you'll recognize and remember, like workshop dvd
* Host: name of the server (will be supplied to you)
* Database: dvdrental
* User: your username
* Password: your password (for the database)

Driver should say PostgreSQL.  If there is a link to use the default driver or download a driver, do that.  

Click the Test Connection button to make sure the information is working.  

Click OK in the bottom right.  

When you get back to the main DataGrip window, you should see the database connection you just set up on the left.  If you expand that entry (multiple times), you'll see the name of the database, then the option to see a list of tables.  Expanding individual tables gives you information about the columns.  

If you right-click on the main entry, there is an option at the bottom of the menu that comes up for diagrams.  If you choose Show Visualization, you'll get a diagram of the objects in the database and the relationships between them.  

You will also have a default .sql file opened for you.  This is a text file where you can type SQL queries and send them to the database.  You'll get results in a window below.  The green triangle button works when your cursor is on a line with a statement.  It will prompt you whether you want to execute just that query or the whole file.  Or, you can highlight code to avoid the prompt.  


 