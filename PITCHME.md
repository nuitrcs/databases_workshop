# Intro to Databases 

---

# What is a Database?


---
<section data-background-color="#401F68">
<span class="special">Structured or organized collection of data</span>
</section>
---

# Databases can take many forms...

---?image=presentation_assets/serverroom.jpeg
<section style="text-align: center;">
<h2> <span style="color: #ffffff;">Sometimes this</span></h2>
</section>
---

## Files on disk

```
20160101.csv	20160601.csv
20160102.csv	20160602.csv
20160103.csv	20160603.csv
20160104.csv	20160604.csv
20160105.csv	20160605.csv
```

---

## Spreadsheet

![spreadsheet](https://upload.wikimedia.org/wikipedia/commons/2/23/Spreadsheet_animation.gif)

---

## Relational Database Management Systems

<img src="http://logos-download.com/wp-content/uploads/2016/10/PostgreSQL_logo_Postgre_SQL.png" 
width="200"> <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/SQLite370.svg/500px-SQLite370.svg.png" width="200"> <img src="https://upload.wikimedia.org/wikipedia/en/thumb/6/62/MySQL.svg/640px-MySQL.svg.png" width="200"> 

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/50/Oracle_logo.svg/2000px-Oracle_logo.svg.png" width="200"> <img src="http://vignette3.wikia.nocookie.net/logopedia/images/c/cd/MicrosoftSQLServer.png/revision/latest?cb=20150614233628" width="200">

--- 

## Specialty database systems

Document database (MongoDB)

Graph database (neo4j)

Distributed (Hbase, Impala)

---

## Small single file
*or*
## large and distributed

---

## Running Locally
*or* 
## on many servers

---

# Today: Relational Databases and SQL

RDBMS: **_R_**elational **_D_**ata**_B_**ase **_M_**anagement **_S_**ystem


SQL: **_S_**tructured **_Q_**uery **_L_**anguage

---

## Why?
<hr>

Most common

Suitable for a wide variety of cases

Accessible and (relatively) easy to use

Familiar structure

---

## Tables of related data
<hr>

Rows of records/observations

Columns of attributes or fields

Tables are linked together with *keys* (relational model of data) 

---

![example](https://bam.files.bbci.co.uk/bam/live/content/zg9syrd/large) <small>[Source: BBC](https://bam.files.bbci.co.uk/bam/live/content/zg9syrd/large)</small>


---

## When are Databases Useful?

Multiple people need access to the same, current data

Individual observations (records) are changing or being added

Enforce constraints about data values, structure, and relationships

Automatically maintain attributes of data like "last modified," "created date"


---

## How does it work?

**SERVER** (usually): 

&nbsp;&nbsp;&nbsp;&nbsp;Holds the data

&nbsp;&nbsp;&nbsp;&nbsp;Manages access and permissions

&nbsp;&nbsp;&nbsp;&nbsp;Can hold multiple databases

&nbsp;&nbsp;&nbsp;&nbsp;Runs locally or on a remote system

**CLIENT** program connects to the server to access the data

---


## Entity-relationship diagram example

<img src="http://www.postgresqltutorial.com/wp-content/uploads/2018/03/dvd-rental-sample-database-diagram.png" 
width="500">


