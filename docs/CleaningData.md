# Cleaning Data
***
Notes taken during/inspired by the Datacamp course 'Cleaning Data in R' by Nick Carchedi.

## Tidying data

In Hadley's paper on tidy data, he talked about how columns in a data frame should be variables or attributes and rows should be observations - this somestimes does not happen if there are things like dummy variables as columns, that could be collpased in to a single column.  The entire data table (data frame) should be about one particular set of data i.e. we have countries without an embedded table about cats.  

Hadley introduced the tidyr package to try and help clean some data.  There are two fundamental verbs of data tidying:

* **gather()** takes multiple columns, and gathers them into key-value pairs: it makes “wide” data longer

* **spread()** takes two columns (key & value) and spreads in to multiple columns, it makes “long” data wider

> gather(data, key, value ...)

**data**: is a data frame

**key**: the name of the new key column

**value**: the name of the new value column

**...**: names of columns to gather or not (if not, state -col e.g. -time to not include the time column in the gathered table)


> spread(data, key, value)

**data**: is a data frame

**key**: the name containing the key column

**value**: the name containing the value column


```r
# Apply gather() to bmi and save the result as bmi_long
bmi_long <- gather(bmi, year, bmi_value, -Country)

# Apply spread() to bmi_long
bmi_wide <- spread(bmi_long, year, bmi_val)
```

Another useful feature is separate().  This takes a single variable and separates it into two separate columns or variable, for instance converting a year-month (2015-10) into a separate column for year and month.

> separate(data, col, into, sep = "")

**data**: a data frame

**col**: bare name of column to separate

**into**: charecter vector of new column names

**Optional sep = ""**: in the separate command you can designate on what item (/, @ etc) to break the data by.  This is optional and can depend on the column type (numeric vs char)


```r
# separate year-mo into two columns
separate(treatments, year_mo, c("year", "month"))
```

We can also use the unite function to combine two columns together

> unite(data, col, ...)

**data**: a data frame

**col**: name of the new column

**...**: columns to unite

The default seperator within the new column is an underscore, however we can specify something different

**Optional sep = "-"**: would add the seperator as a hyphen

head(bmi_cc)
             Country_ISO  year  bmi_val
1         Afghanistan/AF Y1980 21.48678
2             Albania/AL Y1980 25.22533
3             Algeria/DZ Y1980 22.25703
4             Andorra/AD Y1980 25.66652
5              Angola/AO Y1980 20.94876
6 Antigua and Barbuda/AG Y1980 23.31424

So to separate Country_ISO into two columns


```r
# Apply separate() to bmi_cc
bmi_cc_clean <- separate(bmi_cc, col = Country_ISO, into = c("Country", "ISO"), sep = "/")

# Apply unite() to bmi_cc_clean aand reverse
bmi_cc <- unite(bmi_cc_clean, Country_ISO, Country, ISO, sep = "-")
```

## Preparing data for analysis

Often we need to convert, or in the case of raw data, create the appropriate data type for each variable prior to analysis.  Some common data types include

* **character**: "treatment", "123", "A"
* **numeric**: 23.44, 120, NaN, Inf
* **integer**: 4L, 1123L
* **factor**: factor("Hello"), factor(8)
* **logical**: TRUE, FALSE, NA

We can use the class() function to detmine the variable type, or we can also include a value to determine the appropriate type e.g. class(77L) will return [1] "integer".  

We can also use the coercion functions to change the types, such as as.numeric, as.factor() and as.character().

For dates and times, we can use the lubridate package.


```r
# Load the lubridate package
library(lubridate)

# Parse as date
dmy("17 Sep 2015")

# Parse as date and time (with no seconds!)
mdy_hm("July 15, 2012 12:56")

# Coerce dob to a date (with no time)
students2$dob <- ymd(students2$dob)

# Coerce nurse_visit to a date and time
students2$nurse_visit <- ymd_hms(students2$nurse_visit)
```

## String manipulation

Another useful package is stringr, which like lubridate and other Hadley packages has a consistent interface, providing a range of functions for dealing with strings.  Some functions include

* **str_trim()** - Trim leading and trailing white space
* **str_pad()** - Pad with additional characters
* **str_detect()** - Detect a pattern
* **str_replace()** - Find and replace a pattern


```r
# Load the stringr package
library(stringr)

# Trim all leading and trailing whitespace
str_trim(c("   Filip ", "Nick  ", " Jonathan"))

# Pad these strings with leading zeros
str_pad(c("23485W", "8823453Q", "994Z"), width = 9, side = "left", pad = 0)

# Detect all dates of birth (dob) in 1997
str_detect(students2$dob, "1997")

# In the sex column, replace "F" with "Female"...
students2$sex <-  str_replace(students2$sex, "F", "Female")

# ...And "M" with "Male"
students2$sex <- str_replace(students2$sex, "M", "Male")
```

R {base} also has some handy features for strings, including toupper() and tolower().


## Missing, Specials and Outliers 

Generally missing values in R are represented by NA.  However, if the data has been imported from other systems, the values can be different, such as a . (dot) if imported from SPSS.  

We can use the is.na(df) to return a TRUE/FALSE array of where there are NA values in a data frame.  Or, for large datasets, we can use the any(is.na(df)) to return a true or false if there is an NA anywhere in the data frame.  

Alternatively we can use the sum(is.na(df)) to count how many NAs are in the dataframe.  Use complete.cases() to see which rows have no missing values.

Special values include inf for infinite value, NaN for Not a number.

Outliers are best detected by measures such as the IQR or other nuemrical measures (see the EDA section), by using a boxplot or a histogram/density plot.  There are a number of likely reasons for an outlier:

*Valid measurements
*Variability in measurement
*Experimental error
*Data entry error

May be discarded or retained depending on cause.  In some instances we may want to cap, or put a limit on, the maximum number the outlier can.  Looking at the actual values and considering possible values can help, for instance negative age values or a perons age above 200 are not plausible values.  However, they may be data entry errors or in the case of negative numbers, represent a deliberately coded missing value.

## Examples

The weather dataset suffers from one of the five most common symptoms of messy data: column names are values. In particular, the column names X1-X31 represent days of the month, which should really be values of a new variable called day.

head(weather)

X year month  measure           X1 X2 X3 X4 X5 X6 X7 X8 X9 X10 X11 X12 X13 X14 
- ---- -----  ----------------- -- -- -- -- -- -- -- -- -- --- --- --- --- ---
1 2014 12     Max.TemperatureF  64 42 51 43 42 45 38 29 49 48  39  39  42  45
2 2014 12     Mean.TemperatureF 52 38 44 37 34 42 30 24 39 43  36  35  37  39
3 2014 12     Min.TemperatureF  39 33 37 30 26 38 21 18 29 38  32  31  32  33
4 2014 12     Max.Dew.PointF    46 40 49 24 37 45 36 28 49 45  37  28  28  29


```r
# Load the tidyr package
library(tidyr)

# Gather the columns
weather2 <- gather(weather, day, value, X1:X31, na.rm = TRUE)
```

becomes

X year month measure           day value 
- ---- ----- ----------------- --- -----
1 2014 12    Max.TemperatureF  X1    64
2 2014 12    Mean.TemperatureF X1    52
3 2014 12    Min.TemperatureF  X1    39
4 2014 12    Max.Dew.PointF    X1    46
5 2014 12    MeanDew.PointF    X1    40
6 2014 12    Min.DewpointF     X1    26

Our data suffer from a second common symptom of messy data: values are variable names. Specifically, values in the measure column should be variables (i.e. column names) in our dataset.  WE also have an additional column (X) which is not needed as it is just the row number.


```r
# First remove column of row names
weather2 <- weather2[, -1]

# Spread the data
weather3 <- spread(weather2, measure, value)
```


Table: (\#tab:simple-table) 

year  month   day CloudCover Events   Max.Dew.PointF Max.Gust.SpeedMPH  
----  ------  --- --------- -------   -------------- -----------------
2014    12    X1  6         Rain      46             29
2014    12    X2  7         Rain-Snow 40             29
2014    12    X3  8         Rain      49             38
2014    12    X4  3                   24             33
2014    12    X5  5         Rain      37             26
2014    12    X6  8         Rain      45             25
...

Now that the weather dataset adheres to tidy data principles, the next step is to prepare it for analysis. We'll start by combining the year, month, and day columns and recoding the resulting character column as a date. We can use a combination of base R, stringr, and lubridate to accomplish this task.


```r
# Remove X's from day column
weather3$day <- str_replace(weather3$day, "X", "")

# Unite the year, month, and day columns
weather4 <- unite(weather3, date, year, month, day, sep = "-")

# Convert date column to proper date format using lubridates's ymd()
weather4$date <- ymd(weather4$date)

# Rearrange columns using dplyr's select()
weather5 <- select(weather4, date, Events, CloudCover:WindDirDegrees)
```

It's important for analysis that variables are coded appropriately. This is not yet the case with our weather data. Recall that functions such as as.numeric() and as.character() can be used to coerce variables into different types.

It's important to keep in mind that coercions are not always successful, particularly if there's some data in a column that you don't expect. For example, the following will cause problems:

as.numeric(c(4, 6.44, "some string", 222))

So you can use the str_replace function to change character values to something else.

If we have missing data, we can use indices and is.na function to identify then only see those rows with NA values on a variable of interest.


```r
# Count missing values
sum(is.na(weather6))

# Find missing values
summary(weather6)

# Find indices of NAs in Max.Gust.SpeedMPH
ind <- which(is.na(weather6$Max.Gust.SpeedMPH))

# Look at the full rows for records missing Max.Gust.SpeedMPH
weather6[ind, ]
```

Besides missing values, we want to know if there are values in the data that are too extreme or bizarre to be plausible. A great way to start the search for these values is with summary().

Once implausible values are identified, they must be dealt with in an intelligent and informed way. Sometimes the best way forward is obvious and other times it may require some research and/or discussions with the original collectors of the data.


```r
# Find row with Max.Humidity of 1000
ind <- which(weather6$Max.Humidity == 1000)

# Look at the data for that day
weather6[ind, ]

# Change 1000 to 100
weather6$Max.Humidity[ind] <- 100
```

Before officially calling our weather data clean, we want to put a couple of finishing touches on the data. These are a bit more subjective and may not be necessary for analysis, but they will make the data easier for others to interpret, which is generally a good thing.

There are a number of stylistic conventions in the R language. Depending on who you ask, these conventions may vary. Because the period (.) has special meaning in certain situations, we generally recommend using underscores (_) to separate words in variable names. We also prefer all lowercase letters so that no one has to remember which letters are uppercase or lowercase.
