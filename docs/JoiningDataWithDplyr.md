# Joining Data in R with dplyr
***
Notes taken during/inspired by the Datacamp course 'Joining Data in R with dplyr' by Garrett Grolemund.

Other useful info:

* [R for Data Science Book] (http://r4ds.had.co.nz/)
* Data Wrangling Cheatsheet
* dplyr two table verbs vignette
* dbplyr vignette

## Mutating joins

Data is best used in R when in a single data table.  This course introduces a number of techniques to achieve this.  Dplyr also has connectors to a range of different databases, so can be used to both extract and manipulate data in databases.

### Keys

We often want to join two tables together, adding a set of values or variables from a second table(s).  For this to happen, we need a key, whereby we have a key in the initial table (a primary key) that is uniquely identifies rows in that within that table or dataset (we don't have duplicates) and we then use this key to add in data from a secondary table (the foreign key to that table).  The foreign key in the secondary table may be duplicated or not appear at all.  Sometimes no single variable acts as a primary key in a dataset. Instead, it takes a combination of variables to uniquely identify each row, for example a table of addresses with different columns representing sections of the address - house number, street name, postcode/zip code.

When working with dplyr, it works with the following tables for the purposes of joining data

* **Tables** in dplyr are one of the following:
* **data frames**
* **tibbles (tbl_df)** - similar to data frame but only what fits in to your R console window will be displayed but you can use View() all the table if needed 
* **tbl references**

### Left Joins

left_join() is the basic join function in dplyr. You can use it whenever you want to augment a data frame with information from another data frame.

For example, left_join(x, y) joins y to x. The second dataset you specify is joined to the first dataset. 



