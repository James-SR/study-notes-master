# Exploratory Data Analysis
***
Notes taken during/inspired by the Datacamp course 'Exploratory Data Analysis' by Andrew Bray.

## Categorical Data

Common functions when looking at categorical, aka factors variables, are levels(df$var) and to get a contigency or xtab table the table(df$var1, df$var2).  We can also create bar charts to visually represent the data using ggplot.


```r
# Read in our dataset thanks to fivethirtyeight https://github.com/fivethirtyeight/data/tree/master/comic-characters

comics <- read.csv("https://raw.githubusercontent.com/fivethirtyeight/data/master/comic-characters/dc-wikia-data.csv", stringsAsFactors = TRUE)
comics$name <- as.character(comics$name)

# Check levels of align
levels(comics$ALIGN)
```

```
## [1] ""                   "Bad Characters"     "Good Characters"   
## [4] "Neutral Characters" "Reformed Criminals"
```

```r
# Check the levels of gender
levels(comics$SEX)
```

```
## [1] ""                       "Female Characters"     
## [3] "Genderless Characters"  "Male Characters"       
## [5] "Transgender Characters"
```

```r
# Create a 2-way contingency table
table(comics$ALIGN, comics$SEX)
```

```
##                     
##                           Female Characters Genderless Characters
##                        25               220                     0
##   Bad Characters       63               597                    11
##   Good Characters      30               953                     6
##   Neutral Characters    7               196                     3
##   Reformed Criminals    0                 1                     0
##                     
##                      Male Characters Transgender Characters
##                                  356                      0
##   Bad Characters                2223                      1
##   Good Characters               1843                      0
##   Neutral Characters             359                      0
##   Reformed Criminals               2                      0
```

To simplify an analysis, it often helps to drop levels with small amounts of data. In R, this requires two steps: first filtering out any rows with the levels that have very low counts, then removing these levels from the factor variable with droplevels(). This is because the droplevels() function would keep levels that have just 1 or 2 counts; it only drops levels that don’t exist in a dataset.


```r
# Load dplyr
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
# Remove align level
comics <- comics %>%
  filter(ALIGN != "Reformed Criminals") %>%
  droplevels()
```

While a contingency table represents the counts numerically, it's often more useful to represent them graphically.

Here you'll construct two side-by-side barcharts of the comics data. This shows that there can often be two or more options for presenting the same data. Passing the argument position = "dodge" to geom_bar() says that you want a side-by-side (i.e. not stacked) barchart.


```r
# Load ggplot2
library(ggplot2)

# Create side-by-side barchart of gender by alignment
ggplot(comics, aes(x = ALIGN, fill = SEX)) + 
  geom_bar(position = "dodge")
```

<img src="EDA_files/figure-html/unnamed-chunk-3-1.png" width="672" />

```r
# Create side-by-side barchart of alignment by gender
ggplot(comics, aes(x = SEX, fill = ALIGN)) + 
  geom_bar(position = "dodge") +
  theme(axis.text.x = element_text(angle = 90))
```

<img src="EDA_files/figure-html/unnamed-chunk-3-2.png" width="672" />

When creatign tables, it is often easier to look at proportions for patterns rather than counts.  We can do this using conditional proportions, by using the prop.table(df_counts, n) where n is the number we want to condition our frequency/count table by, 1 = rows and 2 = columns.


```r
tab <- table(comics$ALIGN, comics$SEX)
options(scipen = 999, digits = 2) # Print fewer digits
prop.table(tab)     # Joint proportions (totals in the entire table)
```

```
##                     
##                              Female Characters Genderless Characters
##                      0.00363           0.03192               0.00000
##   Bad Characters     0.00914           0.08661               0.00160
##   Good Characters    0.00435           0.13826               0.00087
##   Neutral Characters 0.00102           0.02843               0.00044
##                     
##                      Male Characters Transgender Characters
##                              0.05165                0.00000
##   Bad Characters             0.32250                0.00015
##   Good Characters            0.26737                0.00000
##   Neutral Characters         0.05208                0.00000
```

```r
prop.table(tab, 2)  # Conditional on columns (column totals)
```

```
##                     
##                            Female Characters Genderless Characters
##                      0.200             0.112                 0.000
##   Bad Characters     0.504             0.304                 0.550
##   Good Characters    0.240             0.485                 0.300
##   Neutral Characters 0.056             0.100                 0.150
##                     
##                      Male Characters Transgender Characters
##                                0.074                  0.000
##   Bad Characters               0.465                  1.000
##   Good Characters              0.385                  0.000
##   Neutral Characters           0.075                  0.000
```

Here we see that approx. 49% of female characters are good, compared to 39% for males.

Bar charts can tell dramatically different stories depending on whether they represent counts or proportions and, if proportions, what the proportions are conditioned on. To demonstrate this difference, you'll construct two barcharts in this exercise: one of counts and one of proportions.

```r
# Plot of gender by align
ggplot(comics, aes(x = ALIGN, fill = SEX)) +
  geom_bar()
```

<img src="EDA_files/figure-html/unnamed-chunk-5-1.png" width="672" />

```r
# Plot proportion of gender, conditional on align
ggplot(comics, aes(x = ALIGN, fill = SEX)) + 
  geom_bar(position = "fill")
```

<img src="EDA_files/figure-html/unnamed-chunk-5-2.png" width="672" />
