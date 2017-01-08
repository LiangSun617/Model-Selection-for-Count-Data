
### Model Selection for Count Data

#### Liang Sun
#### December 17, 2016

*This is a summary of what I found when I tried to select appropriate statistical models that fit count data for my dissertation.*

In social sciences, many researchers treat discrete count data as a continuous variable and simply apply an OLS regression model. Especially in educational research, dependent variables can be number of students, number of teachers, number of institutions, etc. Generally, this does not cause a big issue if the dependent variable is normally or approximately normally distributed, most likely, after logarithm transformation. Sometimes, however, it may be really hard for your count data to resemble anything close to a normal distribution.

From the perspective of statistics, regular linear regression model assumes that the dependent variable is continuous without any boundary, which means it can be negative or positive, can be integer or decimal, while count data are always bounded by zero, or even concentrated at zero. In this case, regular regression is probably not a good fit, so statisticians offer alternative models that can address the problem. The most commonly mentioned include Poisson regression, negative binomial regression, zero-inflated Poisson or negative binomial regression.
(You can find more detailed data analysis examples using the above count models in Stata here: http://www.ats.ucla.edu/stat/dae/)

Here I am going to describe how I dealt with count data in my research, the problems I have come across in the process, and how I tested and selected models.

#### Example 1:

The first example is about the number of international students in the United States and how it has been affected by the characteristics of country of origin.

            Figure 1. Histogram of the number of international students before and after transformation

![png](/image/1.png)

Fortunately, it turns into a quite normal distribution after the natural logarithm transformation, and I could simply use OLS. Out of curiosity, I also checked how Poisson regression would differ from OLS:

![png](/image/2.png)

![png](/image/3.png)

In the Poisson model, the dependent variable is “the number of international students” instead of its natural logarithm form because it has to be integers rather than decimals in Poisson model. The Poisson regression command already calculates the Poisson formula on its own where the relationship between the natural logarithm of dependent variable and independent variables is linear, so there is no need to transform the dependent variable.

In comparison of the results of OLS and Poisson regression, the significance and coefficients of some variables seem to differ vastly. The main reason, I think, is that Poisson regression works well only when the dependent variable has equal or almost equal mean and variance, while my data have a much larger standard deviation (12449.35) than mean (3697.632), which is called over-dispersion.

![png](/image/4.png)

Negative binomial regression is usually for modeling over-dispersed count outcome variables, so I turned to the negative binomial regression:

![png](/image/5.png)

In spite of a couple of variables which have different significance and directions of coefficients, the negative binomial regression models gives closer results to the linear regression’s. However, it seems that negative binomial regression produces reasonable result here, but how can we decide which is better, OLS or negative binomial regreesion?

I agree with some people’s opinion that it does not make sense to use Poisson regression because Poisson distribution is based on the probability of the occurrence of an event. It can be hard to simulate or explain that the number of international students who came to study in the United States is due to probability.

In addition, I found out the negative binomial regression cannot converge when I controlled for country fixed effects. It is important for my models to consider the country endogeneity problem, because in panel data countries repeat yearly. If we do not control for countries themselves, we cannot control for unobserved country’s characteristics, if any, which may actually remain stable over time and are correlated with the error term. Then, it violates the assumption of regression that the error term should be random. Moreover, we cannot get a persuasive causal inference between dependent and independent variables, since the variation in the dependent variable may be explained by the unobserved country’s characteristics.

With these considerations, I decided to use OLS for the analysis of the number of international students in the United States.

#### Example 2:

The second example is about the number of collaborative research publications between the United States and a particular country, and how it is affected by researchers from this country who work in the United States. The purpose is to explore whether expatriate researchers contribute to the academic collaboration between their home country and host country.

The dependent variable here, unlike the one in the first example, cannot be normalized even after logarithm transformation:

    Figure 2. Histogram of number of collaborative publications before and after transformation

![png](/image/6.png)

Clearly, it is a zero-inflated distribution; only the values larger than 0 are approximately normally distributed. Apparently, OLS should not be used here, but can I apply those probability-based count models here? I boldly think, yes. Although it is not a typical probability case, it can be treated so if we consider the number of collaborative publications among total number of publication in a country is by chance.

The next question is which model I should select among all the ones that are mentioned above: Poisson, zero-inflated Poisson, negative binomial, or zero-inflated Poisson?

I tested each of them, and here are my findings:

- Poisson vs zero-inflated Poisson

The graph above suggested that the data have excess zero values, but I needed more evidence to believe zero-inflated Poisson works better than regular Poisson regression. So I conducted Vuong test which compares the fitness of regular and zero-inflated Poisson regression. A large test statistic (z = 41.16, Prob>z = 0.000) suggests that zero-inflated Poisson is preferred. However, there are two issues about using zero-inflated Poisson regression here.

First, zero-inflated Poisson assumes that a group of observations do not have probability of getting values larger than zero (absolute zero group). This does not make sense with my data because countries are not likely to have zero chance of having collaborative publications with the United States unless there are no researchers at all in the country.

Second, zero-inflated Poisson regression has difficulty running country fixed-effect model, which will be a big problem for my analysis.

Therefore, I turned to negative binomial regression models.

- Negative binomial vs. zero-inflated negative binomial

As mentioned in the first example, negative binomial works better with over-dispersed data than Poisson regression. The dependent variable, number of collaborative publications, has much larger variance than mean.

![png](/image/7.png)

Then do we really need zero-inflated model here?

No. The Vuong test I performed was insignificant, which suggested that regular negative binomial regression is preferred over zero-inflated one. Paul Allison also suggests that zero-inflated negative binomial regression is not always necessary, and the difference in model fitness between these two is also trivial ([Paul Allison's article](http://statisticalhorizons.com/zero-inflated-models)).

Fortunately, the negative binomial regression model could work with both year and country fixed effects.

![png](/image/8.png)

Finally, I could decide negative binomial regression model is the best fit for my data.

The only issue of using this model is that it may be hard to say that collaborative publications can be considered as a negative binomial process, although social science researchers always ignore the practical mechanism or assumptions of the statistical model they use.

A more straightforward and simple way to deal with zero-inflated data is two-part model, which is common in many social science studies. In my case, I just need to run a logistic regression model of a binary outcome variable which is whether the number of collaborative publication is zero or above zero, and apply another OLS model to observations with non-zero values. Unfortunately, the logit part could not converge when country fixed effects are taken into account.
