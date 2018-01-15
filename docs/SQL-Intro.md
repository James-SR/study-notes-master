# SQL for Data Science
***
Notes taken during/inspired by the Datacamp course 'Intro to SQL for Data Science' by Nick Carchedi. 

Course sections:

* Part 1 - Selecting columns
* Part 2 - Filtering rows
* Part 3 - Aggregate Functions
* Part 4 - Sorting, grouping and joins
* [Additional - KD Nuggets Overview and courses](https://www.kdnuggets.com/2016/06/seven-steps-mastering-sql-data-science.html)


## Selecting columns

SQL is the native language for interacting with databases (RDBMS).  Many non-SQL systems, such as Hadoop and Spark, will often have a SQL above to make extracting and analysing possible.   


```r
library(RPostgreSQL)
```

```
## Warning: package 'RPostgreSQL' was built under R version 3.4.3
```

```
## Loading required package: methods
```

```
## Loading required package: DBI
```

```r
# create a connection
# save the password that we can "hide" it as best as we can by collapsing it
pw <- {"PASSWORD"}

# loads the PostgreSQL driver
drv <- dbDriver("PostgreSQL")
# creates a connection to the postgres database
# note that "con" will be used later in each connection to the database
con <- dbConnect(drv, dbname = "films",
                 host = "localhost", port = 5432,
                 user = "postgres", password = pw)
rm(pw) # removes the password

# Sets knitr to use this connection as the default so we don't need to specify it for every chunk
knitr::opts_chunk$set(connection = "con")
```

We can take a look at the first ten rows using - note that SQL is NOT case sensitive, however it is good practice to make SQL keywords uppercase to distinguish them from other parts of your query, like column and table names.

It's also good practice (but not necessary for the exercises in this course) to include a semicolon at the end of your query. This tells SQL where the end of your query is.


```sql
SELECT name FROM people
LIMIT 10;
```


<div class="knitsql-table">


|name               |
|:------------------|
|50 Cent            |
|A. Michael Baldwin |
|A. Raven Cruz      |
|A.J. Buckley       |
|A.J. DeLucia       |
|A.J. Langer        |
|Aaliyah            |
|Aaron Ashmore      |
|Aaron Hann         |
|Aaron Hill         |

</div>

If multiple columns are needed we use a comma after each field, or if all columns are needed we use SELECT *


```sql
SELECT title, release_year, country
FROM films
LIMIT 10;

SELECT *
FROM films
LIMIT 10;
```


<div class="knitsql-table">


Table: (\#tab:unnamed-chunk-2)Displaying records 1 - 10

id   title                                               release_year  country    duration  language   certification      gross    budget
---  -------------------------------------------------  -------------  --------  ---------  ---------  --------------  --------  --------
1    Intolerance: Love's Struggle Throughout the Ages            1916  USA             123  NA         Not Rated             NA    385907
2    Over the Hill to the Poorhouse                              1920  USA             110  NA         NA               3000000    100000
3    The Big Parade                                              1925  USA             151  NA         Not Rated             NA    245000
4    Metropolis                                                  1927  Germany         145  German     Not Rated          26435   6000000
5    Pandora's Box                                               1929  Germany         110  German     Not Rated           9950        NA
6    The Broadway Melody                                         1929  USA             100  English    Passed           2808000    379000
7    Hell's Angels                                               1930  USA              96  English    Passed                NA   3950000
8    A Farewell to Arms                                          1932  USA              79  English    Unrated               NA    800000
9    42nd Street                                                 1933  USA              89  English    Unrated          2300000    439000
10   She Done Him Wrong                                          1933  USA              66  English    Approved              NA    200000

</div>

Often your results will include many duplicate values. If you want to select all the unique values from a column, you can use the DISTINCT keyword.


```sql
SELECT DISTINCT certification
FROM films;
```


<div class="knitsql-table">


|certification |
|:-------------|
|X             |
|Not Rated     |
|NA            |
|Approved      |
|Unrated       |
|R             |
|NC-17         |
|PG-13         |
|PG            |
|M             |

</div>

We can also use the count function to determine the number of rows in one or more columns.


```sql
SELECT COUNT(*)
FROM reviews;
```


<div class="knitsql-table">


|count |
|:-----|
|4968  |

</div>

We often want to count the number of non-missing values in a column, which we do by specifying the column to count.  We can then combine this with DISTINCT to determine the number of distinct or unique values in the column.


```sql
SELECT COUNT(*)
FROM people;
```


<div class="knitsql-table">


|count |
|:-----|
|8397  |

</div>



```sql

SELECT COUNT(birthdate)
FROM people;
```


<div class="knitsql-table">


|count |
|:-----|
|6152  |

</div>


```sql
SELECT COUNT(DISTINCT birthdate)
FROM people;
```


<div class="knitsql-table">


|count |
|:-----|
|5398  |

</div>


## Filtering
