---
layout: post
title: "Visualizing Model Search"
date: 2012-07-09 09:37
comments: true
categories: 
- analytics
author: David Franke, Ph.D.
---

Model search is a general name for a set of techniques that address the problem of selecting a subset of variables for regression.  See (1) for an in depth discussion of various model search methods, and (2) for guidance on using implementations in R of some of these techniques. Here we are interested in visualizing the variable selection order determined using [lasso regression](http://www-stat.stanford.edu/~tibs/lasso.html) as implemented by the ["lars" package in R](http://www.stanford.edu/~hastie/Papers/LARS/). The goal is to get a sense of the impact of selecting a variable subset of a particular size, and the impact of decreasing or increasing the number of variables in order as determined by the lasso technique.

<!-- more -->
To do this, we first determine a variable ordering using the lasso technique.  With this variable ordering, we then generate the series of linear regression models using all possible prefixes of the variable ordering.  That is, the first model contains the first variable in the ordering, the second model contains the first two variables in the ordering, and so forth.

For each of these models, we collect several "goodness of fit" metrics, specifically the [AIC](http://en.wikipedia.org/wiki/Akaike_information_criterion), R-Squared, and the mean absolute error. Below is an example using a data set with 53 variables, plotting the three "goodness of fit" metrics for the 53 linear regression models.

![AIC](/images/2012-07-09-visualizing-aic.png)

![R-Squared](/images/2012-07-09-visualizing-rsq.png)

![Mean Absolute Error](/images/2012-07-09-visualizing-mae.png)

The impact in terms of "goodness of fit" for adding additional variables to the regression model is easily judged with these graphs. Additional "goodness of fit" or other model metrics are easily added.

The example data set used here included categorical variables, but the "lars" package as of our usage did not support categorical variables. We generated the appropriate dummy variables and modified the feature vector for each data point in order to apply the lasso technique.  For the variable ordering for the series of linear regression models, each categorical variable was positioned at the first occurrence of a dummy variable associated with the categorical variable.

(1) Miller, Alan (2002), Subset Selection in Regression, Chapman & Hall/CRC, Boca Raton, FL.

(2) Wright, Daniel B. and London, Kamala (2009), Modern Regression Techinques Using R: A Practical Guide for Students and Researchers, SAGE Publications Ltd, London.
