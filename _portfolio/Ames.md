---
title: "Ames housing"
excerpt: "Should I renovate this house?"
header:
  image: /assets/images/ames/house.jpg
  teaser: assets/images/ames/house.jpg
---
[See source code and slide deck](https://github.com/cyan-sunset/ames){: .btn .btn--info}

This is a data science project based on the [Ames, Iowa housing dataset].

The dataset includes 79 explanatory variables describing many aspects of residential homes in Ames, Iowa.

![quicklook data]({{ site.url }}{{ site.baseurl }}/assets/images/ames/quicklook.png){: .align-center}

Here is a quick look at the first 5 rows of this dataset, showing a few of the (bajillion) columns. The very first thing I noticed was that the column names are rather cryptic. Luckily the dataset came with a data dictionary - and yes, unfortunately I did have to read every single line of it for this project.

The main takeaways for me after completing this project were skills in advanced regressions, residual analysis, data cleaning, EDA and feature engineering. Data cleaning and feature engineering were particularly important tasks for this dataset.

## Goals for this project

1. Develop an algorithm to reliably estimate the value of residential houses based on fixed characteristics.
2. Identify characteristics of houses that the company can cost-effectively change/renovate with their construction team.

But before we even think about any of that...

## Data cleaning and feature engineering

I needed to take care of the NA values first. After examining the variables closely to determine what an appropriate approach is here, I decided to split up the dataset into numerical and categorical columns and fill numerical NA's with zeros and categorical NA's with "none" (as a string). For my purposes, this solution worked just fine EXCEPT for one discrepancy I found, which was this pesky little column:

![garageyrblt]({{ site.url }}{{ site.baseurl }}/assets/images/ames/garageyrblt.png){: .align-center}

The NA values in this column indicated that the property had no garage. So, if I wanted to use the information here as a continuous variable in the regression analysis it doesn't make sense to fill the NA values with zeros, because the rest of the values were year values and it would skew the data heavily to the left (also it would make no sense - year zero is clearly not realistic). But, filling in with "none" as if it were a categorical variable would've been less than ideal too, as i'd have to (at a later step) dummify far too many year variables. 

I had an idea to then engineer the GarageYrBlt column into some evaluation of how in need of renovation it is, since age is likely related to quality, which is then related to need for renovation.

To do this, I first had to check that garage age (also engineered: latest year in the dataset mimus GarageYrblt to obtain age) is in fact related to garage quality. Since quality was categorical (Poor, Fair, Average, Good, Excellent), the easiest way to have a quick look at this information is a bar graph.

![garage quality and age]({{ site.url }}{{ site.baseurl }}/assets/images/ames/garage-qual-age.png){: .align-center}

From this graph, it's fairly obvious that the newer garages were better quality while the fair and poor quality garages are rather old. However, the red bar representing "excellent" garages has an enormous error bar, which means some of these garages are older but high quality, which means they were probably recently renovated or very well cared for. But further investigation revealed that there were only 3 excellent garages, so we can safely disregard this small discrepancy in the (generally sound) relationship between garage age and quality as there are a total of 1371 garages and 3 garages make up such a small percentage of this.

To evaluate how much in need of a renovation the garage is, I first did some research into garages. I found that the major renovation tasks involved in a garage would be the **garage door opener, and the actual garage door itself.**

I also found out that the average garage door opener lifespan is **10-15 years**, and the garage door itself should last around **30 years**. 

I decided to give the garage door opener the benefit of the doubt, assume it's a good quality product and that it will last 15 years. I then averaged the lifespans of the garage door opener and the garage door, which gave me 22 years average lifespan of products crucial to garage operation. 

By this logic, garages aged:

- Less than 22 years can be considered newly renovated (N)
- More than 22 years and less than 44 years can be considered moderately in need of renovation (M)
- More than 44 years can be considered in immediate need of renovation (I)

Assuming that garages have not been renovated since it was first built (which is likely the case from the chart above, with the possible exception of 3 garages out of 1371).

I used these age bounds to create a new column called "GarageReno" with the labels "N", "M", and "I" to add as a predictor for my regression analysis. 

I've been told that this was probably overkill for this project (and it might be) but at the time I thought it was important and I may as well do a thorough job of it if i've decided on this approach. I think even if it's just a project i'm doing for myself, I should take it seriously and do it as best I can. Also, it would've bothered me to no end if I didn't set out a clear logical approach to doing this.

Next, I noticed that the column "MSSubClass" was numerical but the numbers in the column actually refer to codes (and therefore should be categorical), so I changed the data type to string so that it would dummify correctly.

Since the questions I was trying to answer were split into fixed and renovatable (non-fixed) characteristics, I had to then decide whether each column of data constituted as fixed or non-fixed. This actually isn't as easy as it sounds because many features of a house could be "renovatable" or changeable but at great expense such as "utilities". 

I made dummies for the categorical columns for both fixed and non fixed features, defined the fixed features as the predictor matrix (X), and "SalePrice" as the target (y) to address my first goal:

## Develop an algorithm to reliably estimate the value of residential houses based on fixed characteristics.

I fitted a regular Linear Regression for this data because it had higher train and test scores compared to Lasso and Ridge regression methods. For this scenario, instead of a random train test split based on only proportions, the training data was pre-2010 data and the testing was 2010 data. I did this because it's a better representation of real life problems (predicting what the latest year's sale price should be) than a random test train split. 

### Results

My model had a test score of 0.8503 and a train score of 0.8527.

This is a fairly accurate model. The very similar test and train scores also indicate that the model is not very variant (it performs almost just as well on unseen data as the data it was trained on), which is good as it means the model is not overfit. Judging from the high test and train scores combined with the small difference in scores, it looks like there is a good bias-variance tradeoff in this model (it's not overfit or underfit). 

![best fixed predictors]({{ site.url }}{{ site.baseurl }}/assets/images/ames/best-fixed-predictors.png){: .align-center}

Using SelectKBest, I found the top 10 fixed features that best predict house price. In hindsight, recursive feature elimination probably would've been a good idea to both build the model and find best features and I probably would try that for my next regression problem.

## Identify characteristics of houses that the company can cost-effectively change/renovate with their construction team.

To answer this question, I had to first find out how much of the variance in price that was not explained by the fixed features (approximately 15% since score of previous model was 0.85) can be explained by the non-fixed features. This means that the residuals from the first model for both the training and testing portions of the dataset now become the target of this model, with the predictor matrix being the non-fixed features this time.

Lasso regression gave me the highest score for this, so that's the model I went with using the same test train split as described above.

### Results

This model had a test score of 0.0943 and a train score of 0.0816. Again, the scores are very similar which indicates that the model is not overfit.

![coefficients non fixed]({{ site.url }}{{ site.baseurl }}/assets/images/ames/coefs-nfixed.png){: .align-center}

This is a plot of the coefficients of the top 30 features. Since the target was SalePrice in dollars, the coefficients here represent the number of dollars that correspond to an increase of 1 unit of the predictor variable.

The non-fixed house features that have the most effect on sale price are the overall quality and the overall condition, followed by the masonry veneer area.

![top 3]({{ site.url }}{{ site.baseurl }}/assets/images/ames/top3.png){: .align-center}

This means that the characteristics of houses that the company can cost-effectively renovate would be any renovations that would increase the overall quality by 1 point (10%) without spending more than $3549.15. Similarly with overall condition - spending less than $3244.83 for each 1 unit increase. However, the way i've done this has some inconsistencies. The next top feature (masonry veneer area) has a coefficient of 0.73, but because I didn't standardise the predictor matrix this coefficient actually refers to $0.73 per square foot of masonry veneer area. Compared to the other two top features which were both evaluations out of 10, this is admittedly a little messy and could be improved by standardising first.

## Concluding thoughts

Overall, I really enjoyed the challenges of feature engineering and data cleaning in this project. I think the skills I learnt are very valuable as data cleaning/feature engineering is such a crucial part of tackling every data problem. It was also of course immensely satisfying to find an answer to these problems after so much hard work.

Thanks for reading :)


[Ames, Iowa housing dataset]: https://www.kaggle.com/c/house-prices-advanced-regression-techniques