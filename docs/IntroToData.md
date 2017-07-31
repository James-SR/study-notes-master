# Introduction to Data
***
Notes taken during/inspired by the Datacamp course 'Introduction to Data' by Mine Cetinkaya-Rundel.  The supporting textbook is @OS3.

## Language of Data

The course makes use of the openintro package, accompanying the textbook.  Let's load the package and our first dataset, email50.


```r
# Load packages
library("openintro")
library("dplyr")

# Load data
data(email50)
# View its structure
str(email50)
```

```
## 'data.frame':	50 obs. of  21 variables:
##  $ spam        : num  0 0 1 0 0 0 0 0 0 0 ...
##  $ to_multiple : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ from        : num  1 1 1 1 1 1 1 1 1 1 ...
##  $ cc          : int  0 0 4 0 0 0 0 0 1 0 ...
##  $ sent_email  : num  1 0 0 0 0 0 0 1 1 0 ...
##  $ time        : POSIXct, format: "2012-01-04 13:19:16" "2012-02-16 20:10:06" ...
##  $ image       : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ attach      : num  0 0 2 0 0 0 0 0 0 0 ...
##  $ dollar      : num  0 0 0 0 9 0 0 0 0 23 ...
##  $ winner      : Factor w/ 2 levels "no","yes": 1 1 1 1 1 1 1 1 1 1 ...
##  $ inherit     : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ viagra      : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ password    : num  0 0 0 0 1 0 0 0 0 0 ...
##  $ num_char    : num  21.705 7.011 0.631 2.454 41.623 ...
##  $ line_breaks : int  551 183 28 61 1088 5 17 88 242 578 ...
##  $ format      : num  1 1 0 0 1 0 0 1 1 1 ...
##  $ re_subj     : num  1 0 0 0 0 0 0 1 1 0 ...
##  $ exclaim_subj: num  0 0 0 0 0 0 0 0 1 0 ...
##  $ urgent_subj : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ exclaim_mess: num  8 1 2 1 43 0 0 2 22 3 ...
##  $ number      : Factor w/ 3 levels "none","small",..: 2 3 1 2 2 2 2 2 2 2 ...
```

```r
#glimpse the first few items using dplyr
glimpse(email50)
```

```
## Observations: 50
## Variables: 21
## $ spam         <dbl> 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0...
## $ to_multiple  <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0...
## $ from         <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
## $ cc           <int> 0, 0, 4, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0...
## $ sent_email   <dbl> 1, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 1, 1...
## $ time         <dttm> 2012-01-04 13:19:16, 2012-02-16 20:10:06, 2012-0...
## $ image        <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
## $ attach       <dbl> 0, 0, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2, 0, 0, 0...
## $ dollar       <dbl> 0, 0, 0, 0, 9, 0, 0, 0, 0, 23, 4, 0, 3, 2, 0, 0, ...
## $ winner       <fctr> no, no, no, no, no, no, no, no, no, no, no, no, ...
## $ inherit      <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
## $ viagra       <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
## $ password     <dbl> 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 2, 0, 0, 0, 0, 0...
## $ num_char     <dbl> 21.705, 7.011, 0.631, 2.454, 41.623, 0.057, 0.809...
## $ line_breaks  <int> 551, 183, 28, 61, 1088, 5, 17, 88, 242, 578, 1167...
## $ format       <dbl> 1, 1, 0, 0, 1, 0, 0, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1...
## $ re_subj      <dbl> 1, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 1, 1...
## $ exclaim_subj <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0...
## $ urgent_subj  <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
## $ exclaim_mess <dbl> 8, 1, 2, 1, 43, 0, 0, 2, 22, 3, 13, 1, 2, 2, 21, ...
## $ number       <fctr> small, big, none, small, small, small, small, sm...
```

When using certain functions, such as filters on categorical variables, the way R handles the filtered out variables is to leave the items in as place holders (empty containers), even though the place holder is empty.  This can have undesirable effects, particularly if using the filtered object for modelling.  We then end up with zero values which are actually filtered out factors.

```r
# Subset of emails with big numbers: email50_big
email50_big <- email50 %>%
  filter(number == "big")

# Glimpse the subset
glimpse(email50_big)
```

```
## Observations: 7
## Variables: 21
## $ spam         <dbl> 0, 0, 1, 0, 0, 0, 0
## $ to_multiple  <dbl> 0, 0, 0, 0, 0, 0, 0
## $ from         <dbl> 1, 1, 1, 1, 1, 1, 1
## $ cc           <int> 0, 0, 0, 0, 0, 0, 0
## $ sent_email   <dbl> 0, 0, 0, 0, 0, 1, 0
## $ time         <dttm> 2012-02-16 20:10:06, 2012-02-04 23:26:09, 2012-0...
## $ image        <dbl> 0, 0, 0, 0, 0, 0, 0
## $ attach       <dbl> 0, 0, 0, 0, 0, 0, 0
## $ dollar       <dbl> 0, 0, 3, 2, 0, 0, 0
## $ winner       <fctr> no, no, yes, no, no, no, no
## $ inherit      <dbl> 0, 0, 0, 0, 0, 0, 0
## $ viagra       <dbl> 0, 0, 0, 0, 0, 0, 0
## $ password     <dbl> 0, 2, 0, 0, 0, 0, 8
## $ num_char     <dbl> 7.011, 10.368, 42.793, 26.520, 6.563, 11.223, 10.613
## $ line_breaks  <int> 183, 198, 712, 692, 140, 512, 225
## $ format       <dbl> 1, 1, 1, 1, 1, 1, 1
## $ re_subj      <dbl> 0, 0, 0, 0, 0, 0, 0
## $ exclaim_subj <dbl> 0, 0, 0, 1, 0, 0, 0
## $ urgent_subj  <dbl> 0, 0, 0, 0, 0, 0, 0
## $ exclaim_mess <dbl> 1, 1, 2, 7, 2, 9, 9
## $ number       <fctr> big, big, big, big, big, big, big
```

```r
# Table of number variable - now we have just 7 values
table(email50_big$number)
```

```
## 
##  none small   big 
##     0     0     7
```

```r
# Drop levels
email50_big$number <- droplevels(email50_big$number)

# Another table of number variable
table(email50_big$number)
```

```
## 
## big 
##   7
```

In some instance you want to create a discreet function from a numeric value.  That is to say we want to create a categorical value based on some groups of numbers.  This can be achived as shown below. Note that when calculating a function R will typically either:

* Assign a value e.g. med_num_char <- median(email50$num_char)
* Print a result e.g. median(email50$num_char)
* But we can do both by adding brackets (med_num_char <- median(email50$num_char))


```r
# Calculate median number of characters: med_num_char
(med_num_char <- median(email50$num_char))
```

```
## [1] 6.8895
```

```r
# Create num_char_cat variable in email50
email50 <- email50 %>%
  mutate(num_char_cat = ifelse(num_char < med_num_char, "below median", "at or above median"))
  
# Count emails in each category
table(email50$num_char_cat)
```

```
## 
## at or above median       below median 
##                 25                 25
```

We can also use the mutate function from dplyr to create a new variable from categorical variables


```r
# Create number_yn column in email50
email50 <- email50 %>%
  mutate(number_yn, ifelse(number == "none", "no", "yes"))

# Visualize number_yn
ggplot(email50, aes(x = number_yn)) +
  geom_bar()
```

We often want to compare two or three variables, which is most easily done using the ggplot package


```r
# Load ggplot2
library(ggplot2)

# Scatterplot of exclaim_mess vs. num_char
ggplot(email50, aes(x = num_char, y = exclaim_mess, color = factor(spam))) +
  geom_point()
```

<img src="IntroToData_files/figure-html/unnamed-chunk-5-1.png" width="672" />

## Observational studies and Experiments

Typically there are two types of study, if we are interested in whether variable Y is caused by some factors (X) we could have two types of studies.
 
* **Observational Study**: We are observing, rather than specifically interfere or direct how the data is collected - only correlation can be inferred.  In this case, we might survey people and look for patterns in their characteristics (X) and the outcome variable (Y)
* **Experimental Study**: We randomly assign subjects to various treatments - causation can be inferred. In this case, we would get a group of individuals together then randomly assign them to a group of interest (X), removing the decision from the subjects of the study, we often have a control group also.




# References {-}
