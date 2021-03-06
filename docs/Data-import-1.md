# Importing data - Part 1
***
Notes taken during/inspired by the Datacamp course 'Importing Data in R (Part 1)' by Filip Schouwenaars.

## Introduction 

Data often comes from many different sources and formats, including

* Flat files - simple tables e.g. csv
* Excel
* Databases - MySQL, Postgres
* Websites - APIs, JSON, scraping
* Other statistical software - SPSS, STATA, SAS

## Reading CSV files

Reading csv files can be achived with simple code like


```r
read.csv("file.csv", stringsAsFactors = FALSE)
```

We may want to import strings as categorical variables, in which case we would set stringsAsFactors = TRUE which is also the default option, if not stated.

When working across different machines or operating systems, problems can arise due to different ways of addressing file locations and differing file locations.  Therefore, it can be easier to set a relative path to the users home directory, which would be achieved with the following code.


```r
path <- file.path("~", "datasets", "file.csv")
path
```

```
## [1] "~/datasets/file.csv"
```

Then use the file path as before, assigning to a dataframe.


```r
df <- read.csv(path, stringsAsFactors = FALSE)
```

## Reading tab deliminated files or other table formats

In a similar way to before, we add the path to the file and if we want strings as strings, for instance


```r
read.delim("file.csv", stringsAsFactors = FALSE)
```

However, if the file comes in another format perhaps due to the system encoding or setup, it is still possible to try and read the file as a tabular formatting converting it to a data frame.  To do so, we use the read.table() command which has a lot of arguments that can be customised.  You can specify column names and types for instance.  If for instance we have a file format where the objects are separated by a / rather than a comma or tab as before, we could use


```r
read.table("file.txt",
           header = TRUE,
           sep = "/",
           stringsAsFactors = FALSE)
```

Or, if you have a file which has no column/variable names and tabs as spaces, you would read the file as:


```r
# Path to the file.txt file: path
path <- file.path("data", "file.txt")

# Import the file.txt file: hotdogs
file <- read.table(path, 
                      sep = "\t",                                        # specify seperator - tab in this instance
                      col.names = c("VarName1", "VarName2", "VarName3"), # specifiy variable names
                      colClasses = c("factor", "NULL", "numeric"))       # specify the column/variable classes
```

Both read.csv and read.delim are wrapper functions of read.table(), both use read.table but have different default options depending on the file type.  There are two further wrapper functions - read.csv2 and read.delim2 - which deal with regional differences in formatting, notably that some areas use full stops as decimal places, whereas other areas use commas for decimal places.

## Readr and data.table

These two packages are other ways of reading in files.  Readr uses the tibble, so will be compatible with other tidyverse packages such as dplyr.  It is faster than utils, the r default and also prints out the column classes, depending on what other packages are loaded. It is not necessary to specify stringsAsFactors = FALSE.  


```r
library(readr)
read_csv("file.csv")    #read comma seperated
read_tsv("file2.txt")   #read tab seperated files

#If there are no row heads, you can create a vector then read it in using the col_names argument

#specify the vector for column names
properties <- c("area", "temp", "size", "storage", "method",
                "texture", "flavor", "moistness")
#read in the vector
df <- read_tsv("file3.txt", col_names = properties)
```

Like the utils package, these are wrapper functions, with the base function being read_delim().  Unlike the utils package, read_delim() expects the first row to contain headers, so this doesn't need to be explicit.  As mentioned previously, it is also not necessary to specify the we don't want strings as factors.  You can specify col_names using a vector as before, or we can read them directly at the time.  If we also want to explicitly state the column types, perahps because the automatically assigned variable is not correct, we can do so with col_type using abbreviations:

* c = character
* d = double
* i = integer
* n = number
* l = logical
* D = date
* T = date time
* t = time
* ? = guess
* _ = skip column (underscore)

Finally, we can use skip and n_max to specify how many rows to skip at the beginning of a file, perhaps due to a large header, and the maximum now of rows to read, perhaps due to a very large file with many rows. 


```r
read_delim("file4.txt", delim = "/", col_names = c("var1", "var2", "var3"))

read_delim("file5.txt", delim = "/", col_types = "ccid")

read_delim("file6.txt", delim = "\t", col_names = c("var1", "var2", "var3"), 
           skip = 12, n_max = 50000)
```

Another way of setting the types of the imported columns is using collectors. Collector functions can be passed in a list() to the col_types argument of read_ functions to tell them how to interpret values in a column.  Look at the collector documentation for more details.  Two examples are shown below, one for columns to be interpreted as integers and one for a column with factors.


```r
# The collectors needed for importing
fac <- col_factor(levels = c("Beef", "Meat", "Poultry"))
int <- col_integer()

# Edit the col_types argument with the specified collectors
hotdogs_factor <- read_tsv("hotdogs.txt",
                           col_names = c("type", "calories", "sodium"),
                           col_types = list(fac, int, int))
```

### data.table fread

data.table is a tool for doing [fast data analysis](https://github.com/Rdatatable/data.table/wiki/Benchmarks-%3A-Grouping), particularly on large datasets.  It also has a function to read data using the fread() command.  It can automatically infer column names, types and separators.  You can also drop or select columns at read time.


```r
df <- fread("file7.csv", select = c("colname1", "colname2"))
```

The readr package fill create different dataframe types or object classes - 'tbl_df', 'tbl' and 'data.frame' - which can be useful for different purposes, such as for use in dplyr.  Fread creates a data.table object class.

## Reading Excel files

There are many packages for reading Excel files,  one package is the readxl package by Hadley Wickham.  There are to main functions

* **excel_sheets()**: lists the sheets within an excel file or workbook
* **read_excel()**: import the data, unless specified the first sheet is read, this can either be done with sheet = 7, or sheet = "name".

So to read an urbanpop.xlsx file containing three sheets of urban populations, for different time frames, our code would look similar to that below.


```r
library(readxl)

#list the sheerts in the file
excel_sheets("urbanpop.xlsx")

# Read the sheets, one by one
pop_1 <- read_excel("urbanpop.xlsx", sheet = 1)
pop_2 <- read_excel("urbanpop.xlsx", sheet = 2)
pop_3 <- read_excel("urbanpop.xlsx", sheet = 3)

# Put pop_1, pop_2 and pop_3 in a list: pop_list
pop_list <- list(pop_1, pop_2, pop_3)

# IF we want to read all the files, a more efficient way to read all the files in the file uses lapply
pop_list <- lapply(excel_sheets("urbanpop.xlsx"),
  read_excel,
  path = "urbanpop.xlsx")
```

There are other arguments that can be used with the read_excel() function:

* **col_names**: If true, the first row is read, if false R will assign it's own names or you specify a charecter vector manually
* **col_types**: If NULL, R gueses the data types of the columns.  Alternatively, they can be specified e.g. text, numeric, date, blank (which ignores the col)
* **skip**: Speficies the number of rows to ignore


```r
# Some examples

# Import the the first Excel sheet of urbanpop_nonames.xlsx (R gives names): pop_a
pop_a <- read_excel("urbanpop_nonames.xlsx", col_names = FALSE)

# Import the the first Excel sheet of urbanpop_nonames.xlsx (specify col_names): pop_b
cols <- c("country", paste0("year_", 1960:1966))
pop_b <- read_excel("urbanpop_nonames.xlsx", col_names = cols)

# Import the second sheet of urbanpop.xlsx, skipping the first 21 rows: urbanpop_sel
urbanpop_sel <- read_excel("urbanpop.xlsx", sheet = 2, col_names = FALSE, skip = 21)

# Print out the first observation from urbanpop_sel
urbanpop_sel[1,]
```

### Alternatives for importing Excel files

One alternative is the gdata package, which is a suite of tools for data.  There is a read.xls() function which only, currently, supports XLS files although xlsx could be supported with a driver.  The data is interpreted by the read.xls file using perl into a csv file, which is then read using the read.csv function - itself a offshoot of read.table, in to an R data frame. Hadley's readxl package is faster, but is quite early in it's development so some of the functions may change.  For gdata, as it is an offshoot of read.table(), all of the same arguments can be used by read.xls().

## XLConnect - read and write to excel

Most of the Excel tools can become accessible but inside R, using XLConnect.  It is possible to use XLS and XLSX and it will create a 'workbook' object in R, but it does require Java to work.


```r
library(XLConnect)

#create a connect to a file and list the sheets
book <- loadWorkbook("file.xlsx")
getSheets(book)

#read in the specific sheet but only the columns we are interested in
wardData <- readWorksheet(book, sheet = "sheet_1", startCol = 3, endCol = 5)

# read in the names column, previoulsy excluded
wardNames <- readWorksheet(my_book, sheet = 2, startCol = 1, endCol = 1)

#cbind the data and names together
selection <- cbind(wardNames, wardData)
```

XLConnect has more features than simply reading sheets.  It is possible to write data back to the Excel file also.  We can add sheets, write or add data to sheets, rename and remove sheets.


```r
# Add a worksheet to my_book, named "summary"
createSheet(my_book, "summary")

# Add data in summ to "data_summary" sheet
writeWorksheet(my_book, summ, "summary")

# Save workbook as summary.xlsx
saveWorkbook(my_book, "summary.xlsx")

# Rename "summary" sheet to "data_summary"
renameSheet(my_book, sheet = 4, "data_summary")

# Remove the third sheet
removeSheet(my_book, sheet = 3)
```

