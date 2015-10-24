# NYC Subway Data Analysis

This project look at the NYC Subway data and figure out if more people ride the subway when it is raining versus when it is not raining.

The data can be found at:

* Original data set at: <https://www.dropbox.com/s/meyki2wl9xfa7yk/turnstile_data_master_with_weather.csv>
* Improved data set at: <https://www.dropbox.com/s/1lpoeh2w6px4diu/improved-dataset.zip?dl=0>


## 0. References

* Frost, Jim. Choosing Between a Nonparametric Test and a Parametric Test. <http://blog.minitab.com/blog/adventures-in-statistics/choosing-between-a-nonparametric-test-and-a-parametric-test>
* SPSS tutorial. Mann-Whitney U Test using SPS. <https://statistics.laerd.com/spss-tutorials/mann-whitney-u-test-using-spss-statistics.php>
* Frost, Jim. Regression Analysis: How Do I Interpret R-squared and Assess the Goodness-of-Fit? <http://blog.minitab.com/blog/adventures-in-statistics/regression-analysis-how-do-i-interpret-r-squared-and-assess-the-goodness-of-fit>
* NIST. Are the model residuals well-behaved? <http://www.itl.nist.gov/div898/handbook/pri/section2/pri24.htm>
* OriginLab. Graphic Residual Analysis. <http://www.originlab.com/doc/Origin-Help/Residual-Plot-Analysis>
* Graphpad. One-tail vs. two-tail P values. <http://graphpad.com/guides/prism/6/statistics/index.htm?one-tail_vs__two-tail_p_values.htm>
* Skymark.com. Normal Test Plot. <http://www.skymark.com/resources/tools/normal_test_plot.asp>
* Nau, Robert. Regression diagnostics:  testing the assumptions of linear regression. <http://people.duke.edu/~rnau/testing.htm>
* Frost, Jim. How to Interpret Regression Analysis Results: P-values and Coefficients. <http://blog.minitab.com/blog/adventures-in-statistics/how-to-interpret-regression-analysis-results-p-values-and-coefficients>


## 1. Statistical Analysis

I divided the dataset into two samples, entries per hour when it is raining vs when it is not raining and then perform inferential statistic on them.

### 1.1 Statistical test

#### Statistical test

I perform Mann-Whitney U test on the two samples.

#### One-tail vs Two-tail

I am performing a two-tail P value.

#### Hypothesis

*The null hypothesis is:*

The distributions of both groups are identical, so that there is a 50% probability that an observation from a value randomly selected from one population exceeds an observation randomly selected from the other population.

P(x > y) = 0.5

*Alternative hypothesis:*

The distributions of both groups are not identical, the probability that an observation from a value randomly selected from one population exceeds an observation randomly selected from the other population is not 50%.

P(x > y) != 0.5 

#### Critical value

I use 0.05 as p-critical value.


### 1.2 Reasoning

I am using this test because the samples is non-normal. Generally, we use Mann-Whitney U test, if it follows several assumptions:

* The dependent variable is ordinal or continous
* The independent variable (with rain, without rain) is 2 categorical data
* There is no relationship between the observations in each group or between the groups themselves
* The two variables are not normaly distributed


### 1.3 Result

Calculating the Mann-Whitney U test I got:

U value of **1924409167.0** and p-value of  **0.049999825587**

Some descriptive statistics regarding our two samples.

           With rain summary  Without rain summary
    count       44104.000000          87847.000000
    mean         1105.446377           1090.278780
    std          2370.527674           2320.004938
    min             0.000000              0.000000
    25%            41.000000             38.000000
    50%           282.000000            278.000000
    75%          1103.250000           1111.000000
    max         51839.000000          43199.000000
    
### 1.4 Intrepretation

Since the p-value is < 0.05, we conclude, the Mann-Whitney test indicates that the ridership is significantly different for when it is raining than when it is not raining, U = **1924409167.0**, p = **0.049999825587**.

## 2. Linear regression

### 2.1 Approach
To compute the coefficients theta and produce prediction for ENTRIESn_hourly in the regression model I choose to use OLS using Statsmodels.

### 2.2 Features
I use the following features:

**rain, precipi, meantempi, fog, meanwindspi, Hour**

In addition to this, I have added dummy variables 'UNIT' for the features.

### 2.3 Reasoning behind features selection

I am interested in several features and I choose to play around with combination of several of the following features:

* rain : I think people will decide to ride the subway if it is raining
* precipi : same with the above, it includes snow, drizzle, hail, etc.
* meantempi : I think people will ride the subway more if it is cold
* fog : I think people will ride the subway more if it is fogging, it is probably not fun to drive
* meanwindspi: If it is windy, people who use bike or walk might opt to take subway
* Hour : People tend to ride subway on certain hour, for example to get to or back from work.

Using Python itertool.combination module I generate list of all possible combination from the above feature list (e.g. ['rain'], ['rain', 'precipi'], ['rain', 'fog'] , ['rain', 'fog', 'Hour'], ['rain', 'fog', 'Hour', 'precipi'] and so on). In total there are 63 combinations.

I compute the predictions and the R^2 value for each combination and use the combination with the best R^2 value (R^2 must at least be 0.40). 

In the end I got **rain, precipi, meantempi, fog, meanwindspi, Hour** as the features with the best R^2.


### 2.4 Parameters

The coefficients for the model is:
 
* Intercept: **1081.87869019**
* Non-dummy Parameters: 

    - rain: **-32.26725174**
    - meantempi: **-5.9122**
    - fog: **120.27580878** 
    - meandwindspdi: **26.27992382**
    - precipi: **-22.61805652**
    - Hour: **67.39739472**

### 2.5 R^2

The R^2 value is **0.458621326664**

### 2.6 Intrepretation

The 'rain' coefficient from our model turns out to be negative.
The P value for this coefficient (**0.011**) is greater than 0.05, which indicates that is not statistically significant. The 'precipi' coefficient also has a P value that is greater then 0.05.

To check if the R^2 is fit for the regresion model. I also do some analysis on the residual.
First I plot the residuals histogram as follow:

![png](img/residual_histogram.png)

The histogram has long tails, which suggests that there are some very large residuals. To make sure I plot normal probably plot.

![png](img/residual_normal_probability_plot.png)

The plot confirms that we have long tails problem which means we are seeing more variance than we would expect in a normal distribution.

In fact if we simply plot the residual values we got:

![png](img/residual_plot.png)

It shows that there are some non-linearity in the data.
And this should be addressed by designing a non linear model.

In conclusion, we achieve a R^2 value that we set (> 0.40), but on further inspection we find out that the linear model is **not** appropriate to predict ridership.

The 'rain' coefficient which we initially thought as an important feature, turns out not to be not so important. In fact, we probably should remove it from our model.


## 3. Visualization

I present the following visualization for this project below.

### 3.1 Rainy days vs Non-rainy days histogram

![png](img/rainy_vs_non_rainy.png)

### 3.2 Ridership throughout the day

![png](img/ridership_throughout_day.png)


## 4. Conclusion
Our statistical analysis indicates that there is significant difference in ridership when it is raining vs when it is not raining.

We develop a model to predict the ridership. We used linear model (OLS) and achieve R^2 value that we set as the goal. However after looking at the coefficients and  analyzing the residual, we find that the model is not appropriate to predict the ridership. 

## 5. Reflection

#### Dataset
The dataset although big (131951 records) only covers 30 days of data.
The data was collected from 2011-05-01 to 2011-05-30
This is probably not enough. It will be interesting to see data from multiple months.
If for instance on April it rains a lot, can we see that the ridership on April is significantly different then the ridership on May.

#### Analysis
We used linear regression method to make our models. But the resulting model does not seems to be appropriate to predict the ridership of NYC subway. We need to address the non-linearity in the data. 

Initially I thought maybe if I remove the Hour from the model features list (looking at figure 3.2 I conclude that this is what causes the cycle) the model will work, but I got the same result.