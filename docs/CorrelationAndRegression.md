# Correlation and Regression
***
Notes taken during/inspired by the Datacamp course 'Correlation and Regression' by Ben Baumer.

## Visualizing two variables

Some common terminology of data includes

* Response variable a.k.a. y, dependent (usually on the vertical axis if using a scatter plot)
* Explanatory variable, something you think might be related to the response a.k.a. x, independent, predictor (usually on the horizontal axis)


```r
library(openintro)
library(ggplot2)
library(dplyr)

# load the data
data(ncbirths)

# Scatterplot of weight vs. weeks
ggplot(ncbirths, aes(weeks, weight)) +
  geom_point()
```

```
## Warning: Removed 2 rows containing missing values (geom_point).
```

<img src="CorrelationAndRegression_files/figure-html/unnamed-chunk-1-1.png" width="672" />

If it is helpful, you can think of boxplots as scatterplots for which the variable on the x-axis has been discretized.

The cut() function takes two arguments: the continuous variable you want to discretize and the number of breaks that you want to make in that continuous variable in order to discretize it.


```r
# Boxplot of weight vs. weeks
ggplot(data = ncbirths, 
       aes(x = cut(weeks, breaks = 5), y = weight)) + 
  geom_boxplot()
```

<img src="CorrelationAndRegression_files/figure-html/unnamed-chunk-2-1.png" width="672" />

### Transformations

Here the relationship is hard to see.


```r
data(mammals)
# Mammals scatterplot
ggplot(mammals, aes(BodyWt, BrainWt)) +
  geom_point()
```

<img src="CorrelationAndRegression_files/figure-html/unnamed-chunk-3-1.png" width="672" />

The relationship between two variables may not be linear. In these cases we can sometimes see strange and even inscrutable patterns in a scatterplot of the data. Sometimes there really is no meaningful relationship between the two variables. Other times, a careful transformation of one or both of the variables can reveal a clear relationship.

ggplot2 provides several different mechanisms for viewing transformed relationships. The coord_trans() function transforms the coordinates of the plot. Alternatively, the scale_x_log10() and scale_y_log10() functions perform a base-10 log transformation of each axis. Note the differences in the appearance of the axes.


```r
# Scatterplot with coord_trans()
ggplot(data = mammals, aes(x = BodyWt, y = BrainWt)) + 
  geom_point() + 
  coord_trans(x = "log10", y = "log10")
```

<img src="CorrelationAndRegression_files/figure-html/unnamed-chunk-4-1.png" width="672" />

```r
# Scatterplot with scale_x_log10() and scale_y_log10()
ggplot(data = mammals, aes(x = BodyWt, y = BrainWt)) +
  geom_point() +
  scale_x_log10() + scale_y_log10()
```

<img src="CorrelationAndRegression_files/figure-html/unnamed-chunk-4-2.png" width="672" />

### Identifying Outliers

It is clear here, using the Using the mlbBat10 dataset,  a scatterplot illustrates how the slugging percentage (SLG) of a player varies as a function of his on-base percentage (OBP).


```r
data("mlbBat10")
# Baseball player scatterplot
ggplot(mlbBat10, aes(OBP, SLG)) + 
  geom_point()
```

<img src="CorrelationAndRegression_files/figure-html/unnamed-chunk-5-1.png" width="672" />

Most of the points are clustered in the lower left corner of the plot, making it difficult to see the general pattern of the majority of the data. This difficulty is caused by a few outlying players whose on-base percentages (OBPs) were exceptionally high. These values are present in our dataset only because these players had very few batting opportunities.

Both OBP and SLG are known as rate statistics, since they measure the frequency of certain events (as opposed to their count). In order to compare these rates sensibly, it makes sense to include only players with a reasonable number of opportunities, so that these observed rates have the chance to approach their long-run frequencies.

In Major League Baseball, batters qualify for the batting title only if they have 3.1 plate appearances per game. This translates into roughly 502 plate appearances in a 162-game season. The mlbBat10 dataset does not include plate appearances as a variable, but we can use at-bats (AB) -- which constitute a subset of plate appearances -- as a proxy.


```r
# Scatterplot of SLG vs. OBP
mlbBat10 %>%
  filter(AB >= 200) %>%
  ggplot(aes(x = OBP, y = SLG)) +
  geom_point()
```

<img src="CorrelationAndRegression_files/figure-html/unnamed-chunk-6-1.png" width="672" />

```r
# Identify the outlying player
mlbBat10 %>%
  filter(AB >= 200, OBP < 0.2)
```

```
##     name team position  G  AB  R  H 2B 3B HR RBI TB BB SO SB CS   OBP
## 1 B Wood  LAA       3B 81 226 20 33  2  0  4  14 47  6 71  1  0 0.174
##     SLG   AVG
## 1 0.208 0.146
```
