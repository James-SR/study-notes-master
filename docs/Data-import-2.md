# Importing data - Part 2
***
Notes taken during/inspired by the Datacamp course 'Importing Data in R (Part 2)' by Filip Schouwenaars.

## Importing from Databases - 1

In a professional or commercial setting, you often deal with more complicated file structures and source systems that simple flat files.  Often the data is stored in a DBMS or Database Management System and SQL is the usual way of quering the DBMS.  As there can be slight differences, you are likely to need different packages, some include:

* **MySQL**: Use the RMySQL package
* **PostgresSQL**: Use the RPostgresSQL package
* **Oracle**: Use the ROracle (etc...)

Conventions are specified in the DBI - another R package, DBI is the interface and the other packages are the implentation.  Some of the packages will automaticlaly install the DBI package as well. To connect to a database we would so something like the following.


```r
# Load the DBI package
library(DBI)
```

```
## Loading required package: methods
```

```r
# Edit dbConnect() call - the first part specifies how connections are map to the database
con <- dbConnect(RMySQL::MySQL(), 
                 dbname = "tweater", 
                 host = "courses.csrrinzqubik.us-east-1.rds.amazonaws.com", 
                 port = 3306,
                 user = "student",
                 password = "datacamp")

# Build a vector of table names: tables
tables <- dbListTables(con)

# Display structure of tables
str(tables)
```

```
##  chr [1:3] "comments" "tweats" "users"
```

```r
# Import the users table from tweater: users
users <- dbReadTable(con, "users")

# Print users
users
```

```
##   id      name     login
## 1  1 elisabeth  elismith
## 2  2      mike     mikey
## 3  3      thea   teatime
## 4  4    thomas tomatotom
## 5  5    oliver olivander
## 6  6      kate  katebenn
## 7  7    anjali    lianja
```

```r
# Or we can import all tables using lapply
tables <- lapply(tables, dbReadTable, conn = con)

# Print out tables
tables
```

```
## [[1]]
##      id tweat_id user_id            message
## 1  1022       87       7              nice!
## 2  1000       77       7             great!
## 3  1011       49       5            love it
## 4  1012       87       1   awesome! thanks!
## 5  1010       88       6              yuck!
## 6  1026       77       4      not my thing!
## 7  1004       49       1  this is fabulous!
## 8  1030       75       6           so easy!
## 9  1025       88       2             oh yes
## 10 1007       49       3           serious?
## 11 1020       77       1 couldn't be better
## 12 1014       77       1       saved my day
## 
## [[2]]
##   id user_id
## 1 75       3
## 2 88       4
## 3 77       6
## 4 87       5
## 5 49       1
## 6 24       7
##                                                                  post
## 1                                       break egg. bake egg. eat egg.
## 2                           wash strawberries. add ice. blend. enjoy.
## 3                       2 slices of bread. add cheese. grill. heaven.
## 4               open and crush avocado. add shrimps. perfect starter.
## 5 nachos. add tomato sauce, minced meat and cheese. oven for 10 mins.
## 6                              just eat an apple. simply and healthy.
##         date
## 1 2015-09-05
## 2 2015-09-14
## 3 2015-09-21
## 4 2015-09-22
## 5 2015-09-22
## 6 2015-09-24
## 
## [[3]]
##   id      name     login
## 1  1 elisabeth  elismith
## 2  2      mike     mikey
## 3  3      thea   teatime
## 4  4    thomas tomatotom
## 5  5    oliver olivander
## 6  6      kate  katebenn
## 7  7    anjali    lianja
```

## SQL Queries Inside R

OFten you don't want an entire tabel from a database, but a selection from the table.  You can use SQL queries from inside R to extract only what you are interested in.  You can alternatively use subset on the imported table, but often it is easier to extract only what you need first, particularly when working with large databases.  The SQL goes inside e.g. dbGetQuery(con, "SQL QUERY").


```r
# Connect to the database
library(DBI)
con <- dbConnect(RMySQL::MySQL(),
                 dbname = "tweater",
                 host = "courses.csrrinzqubik.us-east-1.rds.amazonaws.com",
                 port = 3306,
                 user = "student",
                 password = "datacamp")

# Import tweat_id column of comments where user_id is 1: elisabeth
elisabeth <- dbGetQuery(con, "SELECT tweat_id FROM comments WHERE user_id = 1")

# Print elisabeth
elisabeth
```

```
##   tweat_id
## 1       87
## 2       49
## 3       77
## 4       77
```

```r
# Import post column of tweats where date is higher than '2015-09-21': latest
latest <- dbGetQuery(con, "SELECT post FROM tweats WHERE date > '2015-09-21'")

# Print latest
latest
```

```
##                                                                  post
## 1               open and crush avocado. add shrimps. perfect starter.
## 2 nachos. add tomato sauce, minced meat and cheese. oven for 10 mins.
## 3                              just eat an apple. simply and healthy.
```

```r
# Create data frame specific using boolean
specific <- dbGetQuery(con, "SELECT message FROM comments WHERE tweat_id = 77 AND user_id > 4")

# Print specific
specific
```

```
##   message
## 1  great!
```

```r
# Create data frame short selecting two columns
short <- dbGetQuery(con, "SELECT id, name FROM users WHERE CHAR_LENGTH(name) < 5")

# Print short
short
```

```
##   id name
## 1  2 mike
## 2  3 thea
## 3  6 kate
```

```r
# We can also join elements from different tables using the same id/key

dbGetQuery(con, "SELECT post, message
  FROM tweats INNER JOIN comments on tweats.id = tweat_id
    WHERE tweat_id = 77")
```

```
##                                            post            message
## 1 2 slices of bread. add cheese. grill. heaven.             great!
## 2 2 slices of bread. add cheese. grill. heaven.      not my thing!
## 3 2 slices of bread. add cheese. grill. heaven. couldn't be better
## 4 2 slices of bread. add cheese. grill. heaven.       saved my day
```

You've used dbGetQuery() multiple times now. This is a virtual function from the DBI package, but is actually implemented by the RMySQL package. Behind the scenes, the following steps are performed:

* Sending the specified query with dbSendQuery();
* Fetching the result of executing the query on the database with dbFetch();
* Clearing the result with dbClearResult().

Let's not use dbGetQuery() this time and implement the steps above. This is tedious to write, but it gives you the ability to fetch the query's result in chunks rather than all at once. You can do this by specifying the n argument inside dbFetch().

**It is important to close the connection to the database once complete using the dbDisconnect() function**


```r
# Send query to the database
res <- dbSendQuery(con, "SELECT * FROM comments WHERE user_id > 4")

# Use dbFetch() twice
dbFetch(res, n = 2)
```

```
##     id tweat_id user_id message
## 1 1022       87       7   nice!
## 2 1000       77       7  great!
```

```r
dbFetch(res) # imports all
```

```
##     id tweat_id user_id  message
## 1 1011       49       5  love it
## 2 1010       88       6    yuck!
## 3 1030       75       6 so easy!
```

```r
# Clear res
dbClearResult(res)
```

```
## [1] TRUE
```

```r
# Create the data frame  long_tweats
long_tweats <- dbGetQuery(con, "SELECT post, date FROM tweats WHERE CHAR_LENGTH(post) > 40")

# Print long_tweats
print(long_tweats)
```

```
##                                                                  post
## 1                           wash strawberries. add ice. blend. enjoy.
## 2                       2 slices of bread. add cheese. grill. heaven.
## 3               open and crush avocado. add shrimps. perfect starter.
## 4 nachos. add tomato sauce, minced meat and cheese. oven for 10 mins.
##         date
## 1 2015-09-14
## 2 2015-09-21
## 3 2015-09-22
## 4 2015-09-22
```

```r
# Disconnect from the database
dbDisconnect(con)
```

```
## [1] TRUE
```



# References {-}
