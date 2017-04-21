# Foundations of Inference
***
Notes taken during/inspired by the Datacamp course 'Foundations of Inference' by Jo Hardin, collaborators; Nick Carchedi and Tom Jeon.

## Introduction to Inference

Classical statistical inference is the process of making claims about a population based on a sample of information.  We are making an inference from a small group (sample) to a much larger one (population).  We typically have:

* **Null Hypothesis $H_{0}$**: What we are researching has no effect
* **Alternate Hypothesis $H_{A}$**: What we are researching does have an effect

Under the null hypothesis, chance alone is responsible for the results.  Under the alternate hypothesis, we reject the null hypothesis, by using statistical techniques that indicate that chance is not responsible for our findings.  Hypothesis or statistical testing goes back over 300 years, with the first recorded use by John Arbuthnot: 

Table: (\#tab:simple-table) Statistical Testing Applications 

 Year     Person        Context   
--------  ------------  -------------  
1710      Arbuthnot     Sex ratio at birth          
1767      Michelle      Distribution of  stars 
1823      Laplace       Moon phase and barometric changes 
1900      K. Pearson    Goodness of  fit 
1908      Gosset        A single mean 

Source: [@Huberty1993, pg 318]

Contemporary statistical testing is a usually that of either Fisher or Neyman-Pearson approaches. Fisher tends to use a single hypothesis test and a p-value strength of evidence test, where as the Neyman-Pearson test will set a critical alpha value and compare the null hypothesis against an alternative hypothesis, rejecting the null if the test statistic is high enough [@Huberty1993, pg 318]. 

The course goes on to say that idea behind statistical inference is to understand samples from a hypothetical population, where the null hypothesis is true - there is no difference between two groups. We can do this by calculating one statistic - for instance the proportion (mean) of a test group who show a positive response when testing a new drug, compared to a placebo control group - for each repeated sample from a population, then work out the difference between these two groups means. With each sample, the mean will change, resulting in a changing difference for each sample.

We can then generate a distribution (histogram) of differences, assuming the null hypothesis - that there is no link between drug effectiveness between a test group and a control group - is true. *"Generating a distribution of the statistic from the null population gives information about whether the observed data are inconsistent with the null hypothesis"*. That is to say, by taking repeated samples and creating a distribution, we can then say whether our observed difference is consistent (within an acceptable value range due to chance) to the null hypothesis. The null samples consist of randomly shuffled drug effectiveness variables (permuted samples from the population), so that the samples don't have any dependency between the two groups and effectiveness. 

##Home Ownership by Gender

Data used in the exercises are from NHANES 2009-2012 With Adjusted Weighting.

This is survey data collected by the US National Center for Health Statistics (NCHS) which has conducted a series of health and nutrition surveys since the early 1960's. Since 1999 approximately 5,000 individuals of all ages are interviewed in their homes every year and complete the health examination component of the survey. The health examination is conducted in a mobile examination centre (MEC).

The NHANES target population is "the non-institutionalized civilian resident population of the United States". NHANES, (American National Health and Nutrition Examination surveys), use complex survey designs (see http://www.cdc.gov/nchs/data/series/sr_02/sr02_162.pdf) that oversample certain subpopulations like racial minorities. 


```r
# Load packages
library("dplyr")
library("ggplot2")
library("NHANES")
library("oilabs")
```



```r
# Create bar plot for Home Ownership by Gender
ggplot(NHANES, aes(x = Gender, fill = HomeOwn)) + 
  geom_bar(position = "fill") +
  ylab("Relative frequencies")
```

<img src="Foundations-of-Inference_files/figure-html/unnamed-chunk-2-1.png" width="672" />


```r
# Density for SleepHrsNight coloured by SleepTrouble, faceted by HealthGen
ggplot(NHANES, aes(x = SleepHrsNight, col = SleepTrouble)) + 
  geom_density(adjust = 2) + 
  facet_wrap(~ HealthGen)
```

<img src="Foundations-of-Inference_files/figure-html/unnamed-chunk-3-1.png" width="672" />

Next we want to create a selection for just our variables of interest - rent and owner occupation.


```r
# Subset the data: homes
homes <- NHANES %>%
  select(Gender, HomeOwn) %>%
  filter(HomeOwn %in% c("Own", "Rent"))
```

We build a distribution of differences assuming the null hypothesis - that there is no link between gender and home ownership - is true. 

In this first step, we just do a single iteration, or permutation from the true values.  The null (permuted) version here will create a randomly shuffled home ownership variable, so that the permuted version does not have any dependency between gender and homeownership.  We effectively have the same gender split variables as per the original, with the same owned and rented proportions, but disassociated from the gender variable - just randomly shuffled.


```r
# Perform one permutation 
homes %>%
  mutate(HomeOwn_perm = sample(HomeOwn)) %>%
  group_by(Gender) %>%
  summarize(prop_own_perm = mean(HomeOwn_perm == "Own"), 
            prop_own = mean(HomeOwn == "Own")) %>%
  summarize(diff_perm = diff(prop_own),
            diff_orig = diff(prop_own_perm))
```

```
## # A tibble: 1 × 2
##      diff_perm    diff_orig
##          <dbl>        <dbl>
## 1 -0.007828723 -0.002062378
```

It is easier to see what is going on by breaking the results down iteratively.  Our selected and filtered homes dataset looks like. 


```r
head(homes)
```

```
## # A tibble: 6 × 2
##   Gender HomeOwn
##   <fctr>  <fctr>
## 1   male     Own
## 2   male     Own
## 3   male     Own
## 4   male     Own
## 5 female    Rent
## 6   male    Rent
```

Next we shuffle this data, let's call it homes 2. we can then check the total number of owns and rents are the same using the summary function, which confirms the data is just randomly shuffled.


```r
homes2 <- homes %>%
  mutate(HomeOwn_perm = sample(HomeOwn)) %>%
  group_by(Gender)
tail(homes2)
```

```
## Source: local data frame [6 x 3]
## Groups: Gender [2]
## 
##   Gender HomeOwn HomeOwn_perm
##   <fctr>  <fctr>       <fctr>
## 1   male    Rent          Own
## 2   male    Rent          Own
## 3 female     Own         Rent
## 4   male     Own          Own
## 5   male     Own          Own
## 6   male     Own          Own
```

```r
summary(homes2)
```

```
##     Gender      HomeOwn     HomeOwn_perm
##  female:4890   Own  :6425   Own  :6425  
##  male  :4822   Rent :3287   Rent :3287  
##                Other:   0   Other:   0
```

Then we calculate the mean value of home ownership (Own) across our original and shuffled (permutated) data


```r
homes3 <- homes2 %>% 
  summarize(prop_own_perm = mean(HomeOwn_perm == "Own"), 
             prop_own = mean(HomeOwn == "Own"))
homes3
```

```
## # A tibble: 2 × 3
##   Gender prop_own_perm  prop_own
##   <fctr>         <dbl>     <dbl>
## 1 female     0.6660532 0.6654397
## 2   male     0.6569888 0.6576109
```

FFinally we calculate the differences in ownership - note that the difference for the permuted value here may be different from the full code above, as it a new random permutation and we have used the set.seed() function which would create an identical permutation.


```r
homes4 <- homes3 %>% 
  summarize(diff_perm = diff(prop_own),
  diff_orig = diff(prop_own_perm))
homes4
```

```
## # A tibble: 1 × 2
##      diff_perm    diff_orig
##          <dbl>        <dbl>
## 1 -0.007828723 -0.009064368
```

##Density Plots
Next we can make multiple permutations using the rep_sample_n from the oilabs package.  We specify  the data (tbl), the sample size, the number of samples to take (reps), and whether sampling should be done with or without replacement (replace). The output includes a new column, replicate, which indicates the sample number. We can create 100 permutations and create a dot plot of the results.


```r
# Perform 100 permutations
homeown_perm <- homes %>%
  rep_sample_n(size = nrow(homes), reps = 100) %>%
  mutate(HomeOwn_perm = sample(HomeOwn)) %>%
  group_by(replicate, Gender) %>%
  summarize(prop_own_perm = mean(HomeOwn_perm == "Own"), 
            prop_own = mean(HomeOwn == "Own")) %>%
  summarize(diff_perm = diff(prop_own_perm),
            diff_orig = diff(prop_own)) # male - female

# Dotplot of 100 permuted differences in proportions
ggplot(homeown_perm, aes(x = diff_perm)) + 
  geom_dotplot(binwidth = .001)
```

<img src="Foundations-of-Inference_files/figure-html/unnamed-chunk-10-1.png" width="672" />

We can go further and run 1000 permutations and create a density chart.


```r
set.seed(666)
# Perform 1000 permutations
homeown_perm <- homes %>%
  rep_sample_n(size = nrow(homes), reps = 1000) %>%
  mutate(HomeOwn_perm = sample(HomeOwn)) %>%
  group_by(replicate, Gender) %>%
  summarize(prop_own_perm = mean(HomeOwn_perm == "Own"), 
            prop_own = mean(HomeOwn == "Own")) %>%
  summarize(diff_perm = diff(prop_own_perm),
            diff_orig = diff(prop_own)) # male - female

# Density plot of 1000 permuted differences in proportions
ggplot(homeown_perm, aes(x = diff_perm)) + 
  geom_density()
```

<img src="Foundations-of-Inference_files/figure-html/unnamed-chunk-11-1.png" width="672" />

Now we have our density plot of the null hypothesis - randomly permuted samples - we can see where our actual observed difference lies, plus how many other randomly permuted differences were less than the observed difference.


```r
  # Plot permuted differences
ggplot(homeown_perm, aes(x = diff_perm)) + 
  geom_density() +
  geom_vline(aes(xintercept = diff_orig),
          col = "red")
```

<img src="Foundations-of-Inference_files/figure-html/unnamed-chunk-12-1.png" width="672" />

```r
# Compare permuted differences to observed difference and calculate the percent of differences
homeown_perm %>%
  summarize(sum(diff_orig >= diff_perm)) /1000 * 100
```

```
##   sum(diff_orig >= diff_perm)
## 1                        20.5
```

So in this instance, when we set the seed of 666 we end up with 20.5% of randomly shuffled (permuted) differences being greater than the observed difference, so the observed difference is consistent with the null hypothesis.  That it to say it is within the range we may expect by chance alone, were we to repeat the exercise, although we should specify a distribtion we are comparing against, in this which is inferred as being the normal distribution in this instance.  __We can therefore say that there is no statistically significant difference between gender and home ownership__.  Or put more formally

>__We fail to reject the null hypothesis:__
> There is no evidence that our data are inconsistent with the null hypothesis

##Gender Discrimination

In this example we use data from @Rosen1974, where 48 male bank supervisors were given personal files and asked if they should be promoted to Branch Manager. All files were identical, but half (24) were named as female, and the other half (24) were named male.  The results showed 21 males were promoted and 14 females, meaning 35  of the total 48 were promoted. Do we know statistically if there is significant?  

* **Null Hypothesis $H_{0}$**: Gender and promotion are unrelated variables
* **Alternate Hypothesis $H_{A}$**: Men are more likely to be promoted

# References {-}
