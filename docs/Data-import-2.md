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



## Web Data

HyperText Transfer Protocol (HTTP) is the 'language of the web' and consists of a set of rules about data exchange between computers.  If the file is a csv file, we can use functions like read.csv() and add in the url in quotations marks, read.csv will recognise this is a URL and will issue a HTTP GET command to download the file.  This will also work on https sites on newer versions of R.  We can also use the readr package and other packages.


```r
# Load the readr package
library(readr)

# Import the csv file: pools
url_csv <- "http://s3.amazonaws.com/assets.datacamp.com/production/course_1478/datasets/swimming_pools.csv"
pools <- read_csv(url_csv)
```

```
## Parsed with column specification:
## cols(
##   Name = col_character(),
##   Address = col_character(),
##   Latitude = col_double(),
##   Longitude = col_double()
## )
```

```r
# Import the txt file: potatoes
url_delim <- "http://s3.amazonaws.com/assets.datacamp.com/production/course_1478/datasets/potatoes.txt"
potatoes <- read_tsv(url_delim)
```

```
## Parsed with column specification:
## cols(
##   area = col_integer(),
##   temp = col_integer(),
##   size = col_integer(),
##   storage = col_integer(),
##   method = col_integer(),
##   texture = col_double(),
##   flavor = col_double(),
##   moistness = col_double()
## )
```

```r
# Print pools and potatoes
pools
```

```
## # A tibble: 20 × 4
##                                         Name
##                                        <chr>
## 1                Acacia Ridge Leisure Centre
## 2                            Bellbowrie Pool
## 3                                Carole Park
## 4                Centenary Pool (inner City)
## 5                             Chermside Pool
## 6                Colmslie Pool (Morningside)
## 7             Spring Hill Baths (inner City)
## 8                 Dunlop Park Pool (Corinda)
## 9                      Fortitude Valley Pool
## 10 Hibiscus Sports Complex (upper MtGravatt)
## 11                 Ithaca Pool ( Paddington)
## 12                             Jindalee Pool
## 13                                Manly Pool
## 14            Mt Gravatt East Aquatic Centre
## 15       Musgrave Park Pool (South Brisbane)
## 16                            Newmarket Pool
## 17                              Runcorn Pool
## 18                             Sandgate Pool
## 19      Langlands Parks Pool (Stones Corner)
## 20                         Yeronga Park Pool
## # ... with 3 more variables: Address <chr>, Latitude <dbl>,
## #   Longitude <dbl>
```

```r
potatoes
```

```
## # A tibble: 160 × 8
##     area  temp  size storage method texture flavor moistness
##    <int> <int> <int>   <int>  <int>   <dbl>  <dbl>     <dbl>
## 1      1     1     1       1      1     2.9    3.2       3.0
## 2      1     1     1       1      2     2.3    2.5       2.6
## 3      1     1     1       1      3     2.5    2.8       2.8
## 4      1     1     1       1      4     2.1    2.9       2.4
## 5      1     1     1       1      5     1.9    2.8       2.2
## 6      1     1     1       2      1     1.8    3.0       1.7
## 7      1     1     1       2      2     2.6    3.1       2.4
## 8      1     1     1       2      3     3.0    3.0       2.9
## 9      1     1     1       2      4     2.2    3.2       2.5
## 10     1     1     1       2      5     2.0    2.8       1.9
## # ... with 150 more rows
```

```r
# https URL to the swimming_pools csv file.
url_csv <- "https://s3.amazonaws.com/assets.datacamp.com/production/course_1478/datasets/swimming_pools.csv"

# Import the file using read.csv(): pools1
pools1 <- read.csv(url_csv)

str(pools1)
```

```
## 'data.frame':	20 obs. of  4 variables:
##  $ Name     : Factor w/ 20 levels "Acacia Ridge Leisure Centre",..: 1 2 3 4 5 6 19 7 8 9 ...
##  $ Address  : Factor w/ 20 levels "1 Fairlead Crescent, Manly",..: 5 20 18 10 9 11 6 15 12 17 ...
##  $ Latitude : num  -27.6 -27.6 -27.6 -27.5 -27.4 ...
##  $ Longitude: num  153 153 153 153 153 ...
```

Some packages, like the readxl package, do not currently recognise urls.  However, we can use the donwload.file() or other command to download the file and then read it in locally.  This process can be much quicker that browsing the internet then downloading the file.


```r
library(readxl)

# Specification of url: url_xls
url_xls <- "http://s3.amazonaws.com/assets.datacamp.com/production/course_1478/datasets/latitude.xls"

# Download file behind URL, name it local_latitude.xls
download.file(url_xls, destfile = "local_latitude.xls")

# Import the local .xls file with readxl: excel_readxl
excel_readxl <- read_excel("local_latitude.xls")
```


```r
# https URL to the wine RData file.
url_rdata <- "https://s3.amazonaws.com/assets.datacamp.com/production/course_1478/datasets/wine.RData"

# Download the wine file to your working directory
download.file(url_rdata, destfile = "wine_local.RData")

# Load the wine data into your workspace using load()
load("wine_local.RData")

# Print out the summary of the wine data
summary(wine)
```

```
##     Alcohol        Malic acid        Ash        Alcalinity of ash
##  Min.   :11.03   Min.   :0.74   Min.   :1.360   Min.   :10.60    
##  1st Qu.:12.36   1st Qu.:1.60   1st Qu.:2.210   1st Qu.:17.20    
##  Median :13.05   Median :1.87   Median :2.360   Median :19.50    
##  Mean   :12.99   Mean   :2.34   Mean   :2.366   Mean   :19.52    
##  3rd Qu.:13.67   3rd Qu.:3.10   3rd Qu.:2.560   3rd Qu.:21.50    
##  Max.   :14.83   Max.   :5.80   Max.   :3.230   Max.   :30.00    
##    Magnesium      Total phenols     Flavanoids    Nonflavanoid phenols
##  Min.   : 70.00   Min.   :0.980   Min.   :0.340   Min.   :0.1300      
##  1st Qu.: 88.00   1st Qu.:1.740   1st Qu.:1.200   1st Qu.:0.2700      
##  Median : 98.00   Median :2.350   Median :2.130   Median :0.3400      
##  Mean   : 99.59   Mean   :2.292   Mean   :2.023   Mean   :0.3623      
##  3rd Qu.:107.00   3rd Qu.:2.800   3rd Qu.:2.860   3rd Qu.:0.4400      
##  Max.   :162.00   Max.   :3.880   Max.   :5.080   Max.   :0.6600      
##  Proanthocyanins Color intensity       Hue           Proline      
##  Min.   :0.410   Min.   : 1.280   Min.   :1.270   Min.   : 278.0  
##  1st Qu.:1.250   1st Qu.: 3.210   1st Qu.:1.930   1st Qu.: 500.0  
##  Median :1.550   Median : 4.680   Median :2.780   Median : 672.0  
##  Mean   :1.587   Mean   : 5.055   Mean   :2.604   Mean   : 745.1  
##  3rd Qu.:1.950   3rd Qu.: 6.200   3rd Qu.:3.170   3rd Qu.: 985.0  
##  Max.   :3.580   Max.   :13.000   Max.   :4.000   Max.   :1680.0
```

We can also read http content using the httr package.  This includes JSON formatted text, which httr will convert to a named list.


```r
# Load the httr package
library(httr)

# Get the url, save response to resp
url <- "http://www.example.com/"
resp <- GET(url)

# Print resp
resp
```

```
## Response [http://www.example.com/]
##   Date: 2017-08-07 19:08
##   Status: 200
##   Content-Type: text/html
##   Size: 1.27 kB
## <!doctype html>
## <html>
## <head>
##     <title>Example Domain</title>
## 
##     <meta charset="utf-8" />
##     <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
##     <meta name="viewport" content="width=device-width, initial-scale=1" />
##     <style type="text/css">
##     body {
## ...
```

```r
# Get the raw content of resp: raw_content
raw_content <- content(resp, as = "raw")

# Print the head of raw_content
head(raw_content)
```

```
## [1] 3c 21 64 6f 63 74
```

```r
# JSON formatted

# Get the url
url <- "http://www.omdbapi.com/?apikey=ff21610b&t=Annie+Hall&y=&plot=short&r=json"
resp <- GET(url)

# Print resp
resp
```

```
## Response [http://www.omdbapi.com/?apikey=ff21610b&t=Annie+Hall&y=&plot=short&r=json]
##   Date: 2017-08-07 19:08
##   Status: 200
##   Content-Type: application/json; charset=utf-8
##   Size: 902 B
```

```r
# Print content of resp as text
content(resp, as = "text")
```

```
## [1] "{\"Title\":\"Annie Hall\",\"Year\":\"1977\",\"Rated\":\"PG\",\"Released\":\"20 Apr 1977\",\"Runtime\":\"93 min\",\"Genre\":\"Comedy, Romance\",\"Director\":\"Woody Allen\",\"Writer\":\"Woody Allen, Marshall Brickman\",\"Actors\":\"Woody Allen, Diane Keaton, Tony Roberts, Carol Kane\",\"Plot\":\"Neurotic New York comedian Alvy Singer falls in love with the ditzy Annie Hall.\",\"Language\":\"English, German\",\"Country\":\"USA\",\"Awards\":\"Won 4 Oscars. Another 26 wins & 8 nominations.\",\"Poster\":\"https://images-na.ssl-images-amazon.com/images/M/MV5BZDg1OGQ4YzgtM2Y2NS00NjA3LWFjYTctMDRlMDI3NWE1OTUyXkEyXkFqcGdeQXVyMjUzOTY1NTc@._V1_SX300.jpg\",\"Ratings\":[{\"Source\":\"Internet Movie Database\",\"Value\":\"8.1/10\"},{\"Source\":\"Rotten Tomatoes\",\"Value\":\"97%\"}],\"Metascore\":\"N/A\",\"imdbRating\":\"8.1\",\"imdbVotes\":\"210,152\",\"imdbID\":\"tt0075686\",\"Type\":\"movie\",\"DVD\":\"28 Apr 1998\",\"BoxOffice\":\"N/A\",\"Production\":\"United Artists\",\"Website\":\"N/A\",\"Response\":\"True\"}"
```

```r
# Print content of resp
content(resp)
```

```
## $Title
## [1] "Annie Hall"
## 
## $Year
## [1] "1977"
## 
## $Rated
## [1] "PG"
## 
## $Released
## [1] "20 Apr 1977"
## 
## $Runtime
## [1] "93 min"
## 
## $Genre
## [1] "Comedy, Romance"
## 
## $Director
## [1] "Woody Allen"
## 
## $Writer
## [1] "Woody Allen, Marshall Brickman"
## 
## $Actors
## [1] "Woody Allen, Diane Keaton, Tony Roberts, Carol Kane"
## 
## $Plot
## [1] "Neurotic New York comedian Alvy Singer falls in love with the ditzy Annie Hall."
## 
## $Language
## [1] "English, German"
## 
## $Country
## [1] "USA"
## 
## $Awards
## [1] "Won 4 Oscars. Another 26 wins & 8 nominations."
## 
## $Poster
## [1] "https://images-na.ssl-images-amazon.com/images/M/MV5BZDg1OGQ4YzgtM2Y2NS00NjA3LWFjYTctMDRlMDI3NWE1OTUyXkEyXkFqcGdeQXVyMjUzOTY1NTc@._V1_SX300.jpg"
## 
## $Ratings
## $Ratings[[1]]
## $Ratings[[1]]$Source
## [1] "Internet Movie Database"
## 
## $Ratings[[1]]$Value
## [1] "8.1/10"
## 
## 
## $Ratings[[2]]
## $Ratings[[2]]$Source
## [1] "Rotten Tomatoes"
## 
## $Ratings[[2]]$Value
## [1] "97%"
## 
## 
## 
## $Metascore
## [1] "N/A"
## 
## $imdbRating
## [1] "8.1"
## 
## $imdbVotes
## [1] "210,152"
## 
## $imdbID
## [1] "tt0075686"
## 
## $Type
## [1] "movie"
## 
## $DVD
## [1] "28 Apr 1998"
## 
## $BoxOffice
## [1] "N/A"
## 
## $Production
## [1] "United Artists"
## 
## $Website
## [1] "N/A"
## 
## $Response
## [1] "True"
```

## JSON and APIs

JSON is both easy for machines to parse and generate and is human readable.  APIs are programtical ways of getting data, consisting of a set of protocols to interact with some other system or database.  JSON can be useful since it is often well structured and can save time over, say, parsing a html page.  So for instance, you can use the OMDb API to return JSON formatted text about a movie, rather than parse an IMDB html page entry.  One package for handling JSON in R is jsonlite.


```r
library(jsonlite)

# wine_json is a JSON
wine_json <- '{"name":"Chateau Migraine", "year":1997, "alcohol_pct":12.4, "color":"red", "awarded":false}'

# Convert wine_json into a list: wine
wine <- fromJSON(wine_json)

# Print structure of wine
str(wine)
```

```
## List of 5
##  $ name       : chr "Chateau Migraine"
##  $ year       : int 1997
##  $ alcohol_pct: num 12.4
##  $ color      : chr "red"
##  $ awarded    : logi FALSE
```

```r
# Definition of quandl_url
quandl_url <- "http://www.quandl.com/api/v1/datasets/IWS/INTERNET_INDIA.json?auth_token=i83asDsiWUUyfoypkgMz"

# Import Quandl data: quandl_data
quandl_data <- fromJSON(quandl_url)

# Print structure of quandl_data
str(quandl_data)
```

```
## List of 17
##  $ errors      : Named list()
##  $ id          : int 2351831
##  $ source_name : chr "Internet World Stats"
##  $ source_code : chr "IWS"
##  $ code        : chr "INTERNET_INDIA"
##  $ name        : chr "India Internet Usage"
##  $ urlize_name : chr "India-Internet-Usage"
##  $ display_url : chr "http://www.internetworldstats.com/asia/in.htm"
##  $ description : chr "Internet Usage and Population Statistics"
##  $ updated_at  : chr "2016-01-01T04:23:55.235Z"
##  $ frequency   : chr "annual"
##  $ from_date   : chr "1998-12-31"
##  $ to_date     : chr "2012-12-31"
##  $ column_names: chr [1:4] "YEAR" "Users" "Population" "% Pen."
##  $ premium     : logi FALSE
##  $ data        : chr [1:13, 1:4] "2012-12-31" "2010-12-31" "2009-12-31" "2007-12-31" ...
##  $ type        : chr "Time Series"
```

There are two types of JSON structures

* JSON objects - has key value pairs e.g. name:James, age:21 etc
* JSON arrays - a sequence of values, numbers, nulls e.g. 4, "a", 10, false, null etc

You can also nest JSON objects or arrays within each other.  Some examples are below.  YOu can also use the minify and prettify functions to convert a JSON string to a more compact of easier to read version.  Similar functions can also be used inside the toJSON() function e.g. toJSON(x, pretty = TRUE)


```r
# Challenge 1
json1 <- '[1, 2, 3, 4, 5, 6]'
fromJSON(json1)
```

```
## [1] 1 2 3 4 5 6
```

```r
# Challenge 2
json2 <- '{"a": [1, 2, 3], "b": [4, 5, 6]}'
fromJSON(json2)
```

```
## $a
## [1] 1 2 3
## 
## $b
## [1] 4 5 6
```

```r
# You can also convert data to JSON from other formats.  Here we take a csv and format it into a JSON array

# URL pointing to the .csv file
url_csv <- "http://s3.amazonaws.com/assets.datacamp.com/production/course_1478/datasets/water.csv"

# Import the .csv file located at url_csv
water <- read.csv(url_csv, stringsAsFactors = FALSE)

# Convert the data file according to the requirements
water_json <- toJSON(water)

# Print out water_json
water_json
```

```
## [{"water":"Algeria","X1992":0.064,"X2002":0.017},{"water":"American Samoa"},{"water":"Angola","X1992":0.0001,"X2002":0.0001},{"water":"Antigua and Barbuda","X1992":0.0033},{"water":"Argentina","X1992":0.0007,"X1997":0.0007,"X2002":0.0007},{"water":"Australia","X1992":0.0298,"X2002":0.0298},{"water":"Austria","X1992":0.0022,"X2002":0.0022},{"water":"Bahamas","X1992":0.0013,"X2002":0.0074},{"water":"Bahrain","X1992":0.0441,"X2002":0.0441,"X2007":0.1024},{"water":"Barbados","X2007":0.0146},{"water":"British Virgin Islands","X2007":0.0042},{"water":"Canada","X1992":0.0027,"X2002":0.0027},{"water":"Cape Verde","X1992":0.002,"X1997":0.0017},{"water":"Cayman Islands","X1992":0.0033},{"water":"Central African Rep."},{"water":"Chile","X1992":0.0048,"X2002":0.0048},{"water":"Colombia","X1992":0.0027,"X2002":0.0027},{"water":"Cuba","X1992":0.0069,"X1997":0.0069,"X2002":0.0069},{"water":"Cyprus","X1992":0.003,"X1997":0.003,"X2002":0.0335},{"water":"Czech Rep.","X1992":0.0002,"X2002":0.0002},{"water":"Denmark","X1992":0.015,"X2002":0.015},{"water":"Djibouti","X1992":0.0001,"X2002":0.0001},{"water":"Ecuador","X1992":0.0022,"X1997":0.0022,"X2002":0.0022},{"water":"Egypt","X1992":0.025,"X1997":0.025,"X2002":0.1},{"water":"El Salvador","X1992":0.0001,"X2002":0.0001},{"water":"Finland","X1992":0.0001,"X2002":0.0001},{"water":"France","X1992":0.0117,"X2002":0.0117},{"water":"Gibraltar","X1992":0.0077},{"water":"Greece","X1992":0.01,"X2002":0.01},{"water":"Honduras","X1992":0.0002,"X2002":0.0002},{"water":"Hungary","X1992":0.0002,"X2002":0.0002},{"water":"India","X1997":0.0005,"X2002":0.0005},{"water":"Indonesia","X1992":0.0187,"X2002":0.0187},{"water":"Iran","X1992":0.003,"X1997":0.003,"X2002":0.003,"X2007":0.2},{"water":"Iraq","X1997":0.0074,"X2002":0.0074},{"water":"Ireland","X1992":0.0002,"X2002":0.0002},{"water":"Israel","X1992":0.0256,"X2002":0.0256,"X2007":0.14},{"water":"Italy","X1992":0.0973,"X2002":0.0973},{"water":"Jamaica","X1992":0.0005,"X1997":0.0005,"X2002":0.0005},{"water":"Japan","X1997":0.04,"X2002":0.04},{"water":"Jordan","X1997":0.002,"X2007":0.0098},{"water":"Kazakhstan","X1997":1.328,"X2002":1.328},{"water":"Kuwait","X1992":0.507,"X1997":0.231,"X2002":0.4202},{"water":"Lebanon","X2007":0.0473},{"water":"Libya","X2002":0.018},{"water":"Malaysia","X1992":0.0043,"X2002":0.0043},{"water":"Maldives","X1992":0.0004},{"water":"Malta","X1992":0.024,"X1997":0.031,"X2002":0.031},{"water":"Marshall Islands","X1992":0.0007},{"water":"Mauritania","X1992":0.002,"X2002":0.002},{"water":"Mexico","X1992":0.0307,"X2002":0.0307},{"water":"Morocco","X1992":0.0034,"X1997":0.0034,"X2002":0.007},{"water":"Namibia","X1992":0.0003,"X2002":0.0003},{"water":"Netherlands Antilles","X1992":0.063},{"water":"Nicaragua","X1992":0.0002,"X2002":0.0002},{"water":"Nigeria","X1992":0.003,"X2002":0.003},{"water":"Norway","X1992":0.0001,"X2002":0.0001},{"water":"Oman","X1997":0.034,"X2002":0.034,"X2007":0.109},{"water":"Peru","X1992":0.0054,"X2002":0.0054},{"water":"Poland","X1992":0.007,"X2002":0.007},{"water":"Portugal","X1992":0.0016,"X2002":0.0016},{"water":"Qatar","X1992":0.065,"X1997":0.099,"X2002":0.099,"X2007":0.18},{"water":"Saudi Arabia","X1992":0.683,"X1997":0.727,"X2002":0.863,"X2007":1.033},{"water":"Senegal","X1992":0,"X2002":0},{"water":"Somalia","X1992":0.0001,"X2002":0.0001},{"water":"South Africa","X1992":0.018,"X2002":0.018},{"water":"Spain","X1992":0.1002,"X2002":0.1002},{"water":"Sudan","X1992":0.0004,"X1997":0.0004,"X2002":0.0004},{"water":"Sweden","X1992":0.0002,"X2002":0.0002},{"water":"Trinidad and Tobago","X2007":0.036},{"water":"Tunisia","X1992":0.008,"X2002":0.013},{"water":"Turkey","X1992":0.0005,"X2002":0.0005,"X2007":0.0005},{"water":"United Arab Emirates","X1992":0.163,"X1997":0.385,"X2007":0.95},{"water":"United Kingdom","X1992":0.0333,"X2002":0.0333},{"water":"United States","X1992":0.58,"X2002":0.58},{"water":"Venezuela","X1992":0.0052,"X2002":0.0052},{"water":"Yemen, Rep.","X1992":0.01,"X2002":0.01}]
```

```r
# Convert mtcars to a pretty JSON: pretty_json
pretty_json <- toJSON(mtcars, pretty = TRUE)

# Print pretty_json
pretty_json
```

```
## [
##   {
##     "mpg": 21,
##     "cyl": 6,
##     "disp": 160,
##     "hp": 110,
##     "drat": 3.9,
##     "wt": 2.62,
##     "qsec": 16.46,
##     "vs": 0,
##     "am": 1,
##     "gear": 4,
##     "carb": 4,
##     "_row": "Mazda RX4"
##   },
##   {
##     "mpg": 21,
##     "cyl": 6,
##     "disp": 160,
##     "hp": 110,
##     "drat": 3.9,
##     "wt": 2.875,
##     "qsec": 17.02,
##     "vs": 0,
##     "am": 1,
##     "gear": 4,
##     "carb": 4,
##     "_row": "Mazda RX4 Wag"
##   },
##   {
##     "mpg": 22.8,
##     "cyl": 4,
##     "disp": 108,
##     "hp": 93,
##     "drat": 3.85,
##     "wt": 2.32,
##     "qsec": 18.61,
##     "vs": 1,
##     "am": 1,
##     "gear": 4,
##     "carb": 1,
##     "_row": "Datsun 710"
##   },
##   {
##     "mpg": 21.4,
##     "cyl": 6,
##     "disp": 258,
##     "hp": 110,
##     "drat": 3.08,
##     "wt": 3.215,
##     "qsec": 19.44,
##     "vs": 1,
##     "am": 0,
##     "gear": 3,
##     "carb": 1,
##     "_row": "Hornet 4 Drive"
##   },
##   {
##     "mpg": 18.7,
##     "cyl": 8,
##     "disp": 360,
##     "hp": 175,
##     "drat": 3.15,
##     "wt": 3.44,
##     "qsec": 17.02,
##     "vs": 0,
##     "am": 0,
##     "gear": 3,
##     "carb": 2,
##     "_row": "Hornet Sportabout"
##   },
##   {
##     "mpg": 18.1,
##     "cyl": 6,
##     "disp": 225,
##     "hp": 105,
##     "drat": 2.76,
##     "wt": 3.46,
##     "qsec": 20.22,
##     "vs": 1,
##     "am": 0,
##     "gear": 3,
##     "carb": 1,
##     "_row": "Valiant"
##   },
##   {
##     "mpg": 14.3,
##     "cyl": 8,
##     "disp": 360,
##     "hp": 245,
##     "drat": 3.21,
##     "wt": 3.57,
##     "qsec": 15.84,
##     "vs": 0,
##     "am": 0,
##     "gear": 3,
##     "carb": 4,
##     "_row": "Duster 360"
##   },
##   {
##     "mpg": 24.4,
##     "cyl": 4,
##     "disp": 146.7,
##     "hp": 62,
##     "drat": 3.69,
##     "wt": 3.19,
##     "qsec": 20,
##     "vs": 1,
##     "am": 0,
##     "gear": 4,
##     "carb": 2,
##     "_row": "Merc 240D"
##   },
##   {
##     "mpg": 22.8,
##     "cyl": 4,
##     "disp": 140.8,
##     "hp": 95,
##     "drat": 3.92,
##     "wt": 3.15,
##     "qsec": 22.9,
##     "vs": 1,
##     "am": 0,
##     "gear": 4,
##     "carb": 2,
##     "_row": "Merc 230"
##   },
##   {
##     "mpg": 19.2,
##     "cyl": 6,
##     "disp": 167.6,
##     "hp": 123,
##     "drat": 3.92,
##     "wt": 3.44,
##     "qsec": 18.3,
##     "vs": 1,
##     "am": 0,
##     "gear": 4,
##     "carb": 4,
##     "_row": "Merc 280"
##   },
##   {
##     "mpg": 17.8,
##     "cyl": 6,
##     "disp": 167.6,
##     "hp": 123,
##     "drat": 3.92,
##     "wt": 3.44,
##     "qsec": 18.9,
##     "vs": 1,
##     "am": 0,
##     "gear": 4,
##     "carb": 4,
##     "_row": "Merc 280C"
##   },
##   {
##     "mpg": 16.4,
##     "cyl": 8,
##     "disp": 275.8,
##     "hp": 180,
##     "drat": 3.07,
##     "wt": 4.07,
##     "qsec": 17.4,
##     "vs": 0,
##     "am": 0,
##     "gear": 3,
##     "carb": 3,
##     "_row": "Merc 450SE"
##   },
##   {
##     "mpg": 17.3,
##     "cyl": 8,
##     "disp": 275.8,
##     "hp": 180,
##     "drat": 3.07,
##     "wt": 3.73,
##     "qsec": 17.6,
##     "vs": 0,
##     "am": 0,
##     "gear": 3,
##     "carb": 3,
##     "_row": "Merc 450SL"
##   },
##   {
##     "mpg": 15.2,
##     "cyl": 8,
##     "disp": 275.8,
##     "hp": 180,
##     "drat": 3.07,
##     "wt": 3.78,
##     "qsec": 18,
##     "vs": 0,
##     "am": 0,
##     "gear": 3,
##     "carb": 3,
##     "_row": "Merc 450SLC"
##   },
##   {
##     "mpg": 10.4,
##     "cyl": 8,
##     "disp": 472,
##     "hp": 205,
##     "drat": 2.93,
##     "wt": 5.25,
##     "qsec": 17.98,
##     "vs": 0,
##     "am": 0,
##     "gear": 3,
##     "carb": 4,
##     "_row": "Cadillac Fleetwood"
##   },
##   {
##     "mpg": 10.4,
##     "cyl": 8,
##     "disp": 460,
##     "hp": 215,
##     "drat": 3,
##     "wt": 5.424,
##     "qsec": 17.82,
##     "vs": 0,
##     "am": 0,
##     "gear": 3,
##     "carb": 4,
##     "_row": "Lincoln Continental"
##   },
##   {
##     "mpg": 14.7,
##     "cyl": 8,
##     "disp": 440,
##     "hp": 230,
##     "drat": 3.23,
##     "wt": 5.345,
##     "qsec": 17.42,
##     "vs": 0,
##     "am": 0,
##     "gear": 3,
##     "carb": 4,
##     "_row": "Chrysler Imperial"
##   },
##   {
##     "mpg": 32.4,
##     "cyl": 4,
##     "disp": 78.7,
##     "hp": 66,
##     "drat": 4.08,
##     "wt": 2.2,
##     "qsec": 19.47,
##     "vs": 1,
##     "am": 1,
##     "gear": 4,
##     "carb": 1,
##     "_row": "Fiat 128"
##   },
##   {
##     "mpg": 30.4,
##     "cyl": 4,
##     "disp": 75.7,
##     "hp": 52,
##     "drat": 4.93,
##     "wt": 1.615,
##     "qsec": 18.52,
##     "vs": 1,
##     "am": 1,
##     "gear": 4,
##     "carb": 2,
##     "_row": "Honda Civic"
##   },
##   {
##     "mpg": 33.9,
##     "cyl": 4,
##     "disp": 71.1,
##     "hp": 65,
##     "drat": 4.22,
##     "wt": 1.835,
##     "qsec": 19.9,
##     "vs": 1,
##     "am": 1,
##     "gear": 4,
##     "carb": 1,
##     "_row": "Toyota Corolla"
##   },
##   {
##     "mpg": 21.5,
##     "cyl": 4,
##     "disp": 120.1,
##     "hp": 97,
##     "drat": 3.7,
##     "wt": 2.465,
##     "qsec": 20.01,
##     "vs": 1,
##     "am": 0,
##     "gear": 3,
##     "carb": 1,
##     "_row": "Toyota Corona"
##   },
##   {
##     "mpg": 15.5,
##     "cyl": 8,
##     "disp": 318,
##     "hp": 150,
##     "drat": 2.76,
##     "wt": 3.52,
##     "qsec": 16.87,
##     "vs": 0,
##     "am": 0,
##     "gear": 3,
##     "carb": 2,
##     "_row": "Dodge Challenger"
##   },
##   {
##     "mpg": 15.2,
##     "cyl": 8,
##     "disp": 304,
##     "hp": 150,
##     "drat": 3.15,
##     "wt": 3.435,
##     "qsec": 17.3,
##     "vs": 0,
##     "am": 0,
##     "gear": 3,
##     "carb": 2,
##     "_row": "AMC Javelin"
##   },
##   {
##     "mpg": 13.3,
##     "cyl": 8,
##     "disp": 350,
##     "hp": 245,
##     "drat": 3.73,
##     "wt": 3.84,
##     "qsec": 15.41,
##     "vs": 0,
##     "am": 0,
##     "gear": 3,
##     "carb": 4,
##     "_row": "Camaro Z28"
##   },
##   {
##     "mpg": 19.2,
##     "cyl": 8,
##     "disp": 400,
##     "hp": 175,
##     "drat": 3.08,
##     "wt": 3.845,
##     "qsec": 17.05,
##     "vs": 0,
##     "am": 0,
##     "gear": 3,
##     "carb": 2,
##     "_row": "Pontiac Firebird"
##   },
##   {
##     "mpg": 27.3,
##     "cyl": 4,
##     "disp": 79,
##     "hp": 66,
##     "drat": 4.08,
##     "wt": 1.935,
##     "qsec": 18.9,
##     "vs": 1,
##     "am": 1,
##     "gear": 4,
##     "carb": 1,
##     "_row": "Fiat X1-9"
##   },
##   {
##     "mpg": 26,
##     "cyl": 4,
##     "disp": 120.3,
##     "hp": 91,
##     "drat": 4.43,
##     "wt": 2.14,
##     "qsec": 16.7,
##     "vs": 0,
##     "am": 1,
##     "gear": 5,
##     "carb": 2,
##     "_row": "Porsche 914-2"
##   },
##   {
##     "mpg": 30.4,
##     "cyl": 4,
##     "disp": 95.1,
##     "hp": 113,
##     "drat": 3.77,
##     "wt": 1.513,
##     "qsec": 16.9,
##     "vs": 1,
##     "am": 1,
##     "gear": 5,
##     "carb": 2,
##     "_row": "Lotus Europa"
##   },
##   {
##     "mpg": 15.8,
##     "cyl": 8,
##     "disp": 351,
##     "hp": 264,
##     "drat": 4.22,
##     "wt": 3.17,
##     "qsec": 14.5,
##     "vs": 0,
##     "am": 1,
##     "gear": 5,
##     "carb": 4,
##     "_row": "Ford Pantera L"
##   },
##   {
##     "mpg": 19.7,
##     "cyl": 6,
##     "disp": 145,
##     "hp": 175,
##     "drat": 3.62,
##     "wt": 2.77,
##     "qsec": 15.5,
##     "vs": 0,
##     "am": 1,
##     "gear": 5,
##     "carb": 6,
##     "_row": "Ferrari Dino"
##   },
##   {
##     "mpg": 15,
##     "cyl": 8,
##     "disp": 301,
##     "hp": 335,
##     "drat": 3.54,
##     "wt": 3.57,
##     "qsec": 14.6,
##     "vs": 0,
##     "am": 1,
##     "gear": 5,
##     "carb": 8,
##     "_row": "Maserati Bora"
##   },
##   {
##     "mpg": 21.4,
##     "cyl": 4,
##     "disp": 121,
##     "hp": 109,
##     "drat": 4.11,
##     "wt": 2.78,
##     "qsec": 18.6,
##     "vs": 1,
##     "am": 1,
##     "gear": 4,
##     "carb": 2,
##     "_row": "Volvo 142E"
##   }
## ]
```

```r
# Minify pretty_json: mini_json
mini_json <- minify(pretty_json)

# Print mini_json
mini_json
```

```
## [{"mpg":21,"cyl":6,"disp":160,"hp":110,"drat":3.9,"wt":2.62,"qsec":16.46,"vs":0,"am":1,"gear":4,"carb":4,"_row":"Mazda RX4"},{"mpg":21,"cyl":6,"disp":160,"hp":110,"drat":3.9,"wt":2.875,"qsec":17.02,"vs":0,"am":1,"gear":4,"carb":4,"_row":"Mazda RX4 Wag"},{"mpg":22.8,"cyl":4,"disp":108,"hp":93,"drat":3.85,"wt":2.32,"qsec":18.61,"vs":1,"am":1,"gear":4,"carb":1,"_row":"Datsun 710"},{"mpg":21.4,"cyl":6,"disp":258,"hp":110,"drat":3.08,"wt":3.215,"qsec":19.44,"vs":1,"am":0,"gear":3,"carb":1,"_row":"Hornet 4 Drive"},{"mpg":18.7,"cyl":8,"disp":360,"hp":175,"drat":3.15,"wt":3.44,"qsec":17.02,"vs":0,"am":0,"gear":3,"carb":2,"_row":"Hornet Sportabout"},{"mpg":18.1,"cyl":6,"disp":225,"hp":105,"drat":2.76,"wt":3.46,"qsec":20.22,"vs":1,"am":0,"gear":3,"carb":1,"_row":"Valiant"},{"mpg":14.3,"cyl":8,"disp":360,"hp":245,"drat":3.21,"wt":3.57,"qsec":15.84,"vs":0,"am":0,"gear":3,"carb":4,"_row":"Duster 360"},{"mpg":24.4,"cyl":4,"disp":146.7,"hp":62,"drat":3.69,"wt":3.19,"qsec":20,"vs":1,"am":0,"gear":4,"carb":2,"_row":"Merc 240D"},{"mpg":22.8,"cyl":4,"disp":140.8,"hp":95,"drat":3.92,"wt":3.15,"qsec":22.9,"vs":1,"am":0,"gear":4,"carb":2,"_row":"Merc 230"},{"mpg":19.2,"cyl":6,"disp":167.6,"hp":123,"drat":3.92,"wt":3.44,"qsec":18.3,"vs":1,"am":0,"gear":4,"carb":4,"_row":"Merc 280"},{"mpg":17.8,"cyl":6,"disp":167.6,"hp":123,"drat":3.92,"wt":3.44,"qsec":18.9,"vs":1,"am":0,"gear":4,"carb":4,"_row":"Merc 280C"},{"mpg":16.4,"cyl":8,"disp":275.8,"hp":180,"drat":3.07,"wt":4.07,"qsec":17.4,"vs":0,"am":0,"gear":3,"carb":3,"_row":"Merc 450SE"},{"mpg":17.3,"cyl":8,"disp":275.8,"hp":180,"drat":3.07,"wt":3.73,"qsec":17.6,"vs":0,"am":0,"gear":3,"carb":3,"_row":"Merc 450SL"},{"mpg":15.2,"cyl":8,"disp":275.8,"hp":180,"drat":3.07,"wt":3.78,"qsec":18,"vs":0,"am":0,"gear":3,"carb":3,"_row":"Merc 450SLC"},{"mpg":10.4,"cyl":8,"disp":472,"hp":205,"drat":2.93,"wt":5.25,"qsec":17.98,"vs":0,"am":0,"gear":3,"carb":4,"_row":"Cadillac Fleetwood"},{"mpg":10.4,"cyl":8,"disp":460,"hp":215,"drat":3,"wt":5.424,"qsec":17.82,"vs":0,"am":0,"gear":3,"carb":4,"_row":"Lincoln Continental"},{"mpg":14.7,"cyl":8,"disp":440,"hp":230,"drat":3.23,"wt":5.345,"qsec":17.42,"vs":0,"am":0,"gear":3,"carb":4,"_row":"Chrysler Imperial"},{"mpg":32.4,"cyl":4,"disp":78.7,"hp":66,"drat":4.08,"wt":2.2,"qsec":19.47,"vs":1,"am":1,"gear":4,"carb":1,"_row":"Fiat 128"},{"mpg":30.4,"cyl":4,"disp":75.7,"hp":52,"drat":4.93,"wt":1.615,"qsec":18.52,"vs":1,"am":1,"gear":4,"carb":2,"_row":"Honda Civic"},{"mpg":33.9,"cyl":4,"disp":71.1,"hp":65,"drat":4.22,"wt":1.835,"qsec":19.9,"vs":1,"am":1,"gear":4,"carb":1,"_row":"Toyota Corolla"},{"mpg":21.5,"cyl":4,"disp":120.1,"hp":97,"drat":3.7,"wt":2.465,"qsec":20.01,"vs":1,"am":0,"gear":3,"carb":1,"_row":"Toyota Corona"},{"mpg":15.5,"cyl":8,"disp":318,"hp":150,"drat":2.76,"wt":3.52,"qsec":16.87,"vs":0,"am":0,"gear":3,"carb":2,"_row":"Dodge Challenger"},{"mpg":15.2,"cyl":8,"disp":304,"hp":150,"drat":3.15,"wt":3.435,"qsec":17.3,"vs":0,"am":0,"gear":3,"carb":2,"_row":"AMC Javelin"},{"mpg":13.3,"cyl":8,"disp":350,"hp":245,"drat":3.73,"wt":3.84,"qsec":15.41,"vs":0,"am":0,"gear":3,"carb":4,"_row":"Camaro Z28"},{"mpg":19.2,"cyl":8,"disp":400,"hp":175,"drat":3.08,"wt":3.845,"qsec":17.05,"vs":0,"am":0,"gear":3,"carb":2,"_row":"Pontiac Firebird"},{"mpg":27.3,"cyl":4,"disp":79,"hp":66,"drat":4.08,"wt":1.935,"qsec":18.9,"vs":1,"am":1,"gear":4,"carb":1,"_row":"Fiat X1-9"},{"mpg":26,"cyl":4,"disp":120.3,"hp":91,"drat":4.43,"wt":2.14,"qsec":16.7,"vs":0,"am":1,"gear":5,"carb":2,"_row":"Porsche 914-2"},{"mpg":30.4,"cyl":4,"disp":95.1,"hp":113,"drat":3.77,"wt":1.513,"qsec":16.9,"vs":1,"am":1,"gear":5,"carb":2,"_row":"Lotus Europa"},{"mpg":15.8,"cyl":8,"disp":351,"hp":264,"drat":4.22,"wt":3.17,"qsec":14.5,"vs":0,"am":1,"gear":5,"carb":4,"_row":"Ford Pantera L"},{"mpg":19.7,"cyl":6,"disp":145,"hp":175,"drat":3.62,"wt":2.77,"qsec":15.5,"vs":0,"am":1,"gear":5,"carb":6,"_row":"Ferrari Dino"},{"mpg":15,"cyl":8,"disp":301,"hp":335,"drat":3.54,"wt":3.57,"qsec":14.6,"vs":0,"am":1,"gear":5,"carb":8,"_row":"Maserati Bora"},{"mpg":21.4,"cyl":4,"disp":121,"hp":109,"drat":4.11,"wt":2.78,"qsec":18.6,"vs":1,"am":1,"gear":4,"carb":2,"_row":"Volvo 142E"}]
```


# References {-}
