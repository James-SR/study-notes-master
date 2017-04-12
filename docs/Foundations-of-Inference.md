# Foundations of Inference
***
By Jo Hardin. Collaborators; Nick Carchedi and Tom Jeon.

## Introduction to Inference

Classical statistical inference is the process of making claims about a population based on a sample of information.  We are making an inference from a small group (sample) to a much larger one (population).  We typically have:

* **Null Hypothesis $H_{0}$**: What we are researching has no effect
* **Alternate Hypothesis $H_{A}$**: What we are researching does have an effect

Under the null hypothesis, chance alone is responsible for the results.  Under the alternate hypothesis, we reject the null hypothesis, by using statistical techniques that indicate that chance is not responsible for our findings.  Statistical inference largely grew out of the work by Pearson and Fisher, the following summarises the history of significance testing succinctly:

>*"Significance testing is largely the product of Karl Pearson (p-value, Pearson's chi-squared test), William Sealy Gosset (Student's t-distribution), and Ronald Fisher ("null hypothesis", analysis of variance, "significance test"), while hypothesis testing was developed by Jerzy Neyman and Egon Pearson (son of Karl)...Modern hypothesis testing is an inconsistent hybrid of the Fisher vs Neyman/Pearson formulation, methods and terminology developed in the early 20th century. While hypothesis testing was popularized early in the 20th century, evidence of its use can be found much earlier [Such as Laplace in the 1770s]"*. [@wiki:HypothesisTesting]

The idea behind statistical inference is to understand samples from a hypothetical population, where the null hypothesis is true - there is no difference between two groups. We can do this by calculating one statistic - for instance the proportion (mean) of a test group who show a positive response when testing a new drug, compared to a placebo control group - for each repeated sample from a population, then work out the difference between these two groups means. With each sample, the mean will change, resulting in a changing difference for each sample.

We can then generate a distribution (histogram) of differences, assuming the null hypothesis - that there is no link between drug effectiveness between a test group and a control group - is true. *"Generating a distribution of the statistic from the null population gives information about whether the observed data are inconsistent with the null hypothesis"*. That is to say, by taking repeated samples creating a distribution, we can then say whether our observed difference is consistent (within an acceptable value range due to chance) to the null hypothesis. The null samples consist of randomly shuffled drug effectiveness variables (permuted samples from the population), so that the samples don't have any dependency between the two groups and effectiveness. 
