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

##Section 1 Exercises

Data used in the exercises are from NHANES 2009-2012 With Adjusted Weighting.

This is survey data collected by the US National Center for Health Statistics (NCHS) which has conducted a series of health and nutrition surveys since the early 1960's. Since 1999 approximately 5,000 individuals of all ages are interviewed in their homes every year and complete the health examination component of the survey. The health examination is conducted in a mobile examination centre (MEC).

The NHANES target population is "the non-institutionalized civilian resident population of the United States". NHANES, (American National Health and Nutrition Examination surveys), use complex survey designs (see http://www.cdc.gov/nchs/data/series/sr_02/sr02_162.pdf) that oversample certain subpopulations like racial minorities. 


```r
# Load packages
library("dplyr")
library("ggplot2")
library("NHANES")
```



```r
# Create bar plot for Home Ownership by Gender
ggplot(NHANES, aes(x = Gender, fill = HomeOwn)) + 
  geom_bar(position = "fill") +
  ylab("Relative frequencies")
```

<img src="Foundations-of-Inference_files/figure-html/unnamed-chunk-2-1.png" width="672" />


```r
# Density for SleepHrsNight colored by SleepTrouble, faceted by HealthGen
ggplot(NHANES, aes(x = SleepHrsNight, col = SleepTrouble)) + 
  geom_density(adjust = 2) + 
  facet_wrap(~ HealthGen)
```

<img src="Foundations-of-Inference_files/figure-html/unnamed-chunk-3-1.png" width="672" />





```r
# Subset the data: homes
homes <- NHANES %>%
  select(Gender, HomeOwn) %>%
  filter(HomeOwn %in% c("Own", "Rent"))
```

We build a distribution of differences assuming the null hypothesis - that there is no link between gender and home ownership - is true. 

In this first step, we just do a single iteration, or permutation from the true values.  The null (permuted) version here will create a randomly shuffled home ownership variable, so that the permuted version does not have any dependency between gender and homeownership.  We effectively have the same gender split variables as per the original, with the same owned and rented proportions, but disassociated from the gender variable - just randomly perturbed.



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
##      diff_perm   diff_orig
##          <dbl>       <dbl>
## 1 -0.007828723 0.006999022
```



