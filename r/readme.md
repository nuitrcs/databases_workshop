# Databases in R

You will need the `RPostgres` package in R for the materials here, as well as access to a PostgreSQL server.  The examples here use the database from the [PostgreSQL Tutorial](http://www.postgresqltutorial.com/).

RStudio version 1.1 and later has beta support for a database connections tab (by default a tab in the upper right).  This tab is to let you manage your database connections and view the tables and views inside the database like we did with DataGrip.  This functionality requires the installation of some special R packages and system packages.  See ODBC requirements at https://github.com/r-dbi/odbc.  Information on the Connections tab at https://support.rstudio.com/hc/en-us/articles/115010915687.  Information on setting up odbc: https://cran.r-project.org/web/packages/odbc/readme/README.html#connection-strings.  Note that you probably need to edit some system files -- at this point using this functionality requires fairly advanced knowledge of database connections and system files. 


# Files

If you haven't downloaded the entire repository, you can right-click the download links to save just those files.  Be careful of the file type you save the file as.  

R File shown in the workshop: [download R Notebook](https://raw.githubusercontent.com/nuitrcs/databases_workshop/master/r/r_databases.Rmd) | [html](https://nuitrcs.github.io/databases_workshop/r/r_databases.html)

Exercises: [html](https://nuitrcs.github.io/databases_workshop/r/exercises.html)

Exercises with answers: [download RMarkdown](https://raw.githubusercontent.com/nuitrcs/databases_workshop/master/r/exercises_with_answers.Rmd) | [html](https://nuitrcs.github.io/databases_workshop/r/exercises_with_answers.html)


# Resources

[RStudio Guide to Databases in R](https://db.rstudio.com/): covers different packages, connecting, writing queries, drivers, etc.  Also has information about the Database Connections Pane in RStudio (in recent versions).

[Overview of Databases in R](https://rstudio-pubs-static.s3.amazonaws.com/52614_1fa12c657ba7492092bd538205d7f02e.html): this uses SQLite, not PostgreSQL, as an example

[Visualization and Databases in R](https://rviews.rstudio.com/2018/08/16/visualizations-with-r-and-databases/): good discusion and examples of how to do visualizations of data that lives in a database without pulling it all into R first

[Working with Databases in RStudio](http://db.rstudio.com/): features of RStudio, and packages created by RStudio, to help you work with databases in R; some of the features discussed are only available in development versions of RStudio at this point.

[Databases Using R](https://rviews.rstudio.com/2017/05/17/databases-using-r/) by Edgar Ruiz is an up-to-date overview

[SQL Databases and R](http://www.datacarpentry.org/R-ecology-lesson/05-r-and-databases.html) from Data Carpentry (part of a larger introductory R workshop)
