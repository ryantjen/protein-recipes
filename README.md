Author: Ryan Tjenalooi

## Overview

In this data science project, I will be exploring the relationship between recipes with high protein and cooking time to determine the feasibility of healthy eating.

## Introduction

Food is a critical component of our daily lives, and cooking is a hobby that brings joy and satisfaction for many. While there are many of us who have a sweet tooth and love sugary food, the health risks associated with them, such as diabetes, are not negligible. According to the National Diabetes Statistics Report from Centers for Disease Control and Prevention, 11.6% of the US population have diabetes and over 38.0% of people over 18 years old have prediabetes. With this information in our mind, we want to investigate the relationship between rating and the amount of sugar present in the recipe. **We wonder if people would rate a sugary recipe lower due to the health concerns relating to it**. To do so, we are analyzing two dataset consisting of recipes and ratings posted since 2008 on [food.com](https://www.food.com/). The original purpose of the datasets is for the recommender system research paper, [Generating Personalized Recipes from Historical User Preferences](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) by Majumder et al.

The first dataset, `recipe`, contains 83782 rows, indicating 83782 unique recipes, with 10 columns recording the following information:

| Column             | Description                                                                                                                                                                                       |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `'name'`           | Recipe name                                                                                                                                                                                       |
| `'id'`             | Recipe ID                                                                                                                                                                                         |
| `'minutes'`        | Minutes to prepare recipe                                                                                                                                                                         |
| `'contributor_id'` | User ID who submitted this recipe                                                                                                                                                                 |
| `'submitted'`      | Date recipe was submitted                                                                                                                                                                         |
| `'tags'`           | Food.com tags for recipe                                                                                                                                                                          |
| `'nutrition'`      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'`        | Number of steps in recipe                                                                                                                                                                         |
| `'steps'`          | Text for recipe steps, in order                                                                                                                                                                   |
| `'description'`    | User-provided description                                                                                                                                                                         |
| `'ingredients'`    | Text for recipe ingredients                                                                                                                                                                       |
| `'n_ingredients'`  | Number of ingredients in recipe                                                                                                                                                                   |

The second dataset, `interactions`, contains 731927 rows and each row contains a review from the user on a specific recipe. The columns it includes are:

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |

**Given the datasets, we are investigating whether people rate sugary recipes and the non-sugary recipes on the same scale.** To facilitate the investigation of our question, we separated the values in the `'nutrition'` columns into the corresponding columns,`'calories (#)'`, `'total fat (PDV)'`, `'sugar (PDV)'`, etc. PDV, or percent daily value shows how much a nutrient in a serving of food contributes to a total daily diet. Moreover, we calculated the proportion of sugar in terms of calories out of the total calories of a given recipe and stored the information in a new column, `'prop_sugar'`. because sugary in here will be referring to the recipes with value `'prop_sugar'` higher than the average `'prop_sugar'`. The most relevant columns to answer our question are `'calories(#)'`, `'sugar (PDV)'`, `'prop_sugar'`, described above, `'rating'`, which is the rating that user gave on a recipe, and `'average rating'`, which are the average of the ratings on each unique recipes.

By seeking an answer to our question, we would have an insight on people’s preference on sugary recipes, which could help contributors on Food.com revise and improve their recipes to align with the public’s interests. In addition, the new pieces of information could lead to future work on diving deeper into how much awareness people have on the negative health effects of sweets.

## Data Cleaning and Exploratory Data Analysis

To make our analysis of the dataset more efficient and convenient, we conducted the following data cleaning steps.

1. Left merge the recipes and interactions datasets on id and recipe_id.

   - This step helps match the unique recipes with their rating and review.

1. Check data types of all the columns.

   - This step helps us evaluate what data cleaning steps are appropriate for the dataset and if we need to conduct data type conversion.
   - | Column             | Description |
     | :----------------- | :---------- |
     | `'name'`           | object      |
     | `'id'`             | int64       |
     | `'minutes'`        | int64       |
     | `'contributor_id'` | int64       |
     | `'submitted'`      | object      |
     | `'tags'`           | object      |
     | `'nutrition'`      | object      |
     | `'n_steps'`        | int64       |
     | `'steps'`          | object      |
     | `'description'`    | object      |
     | `'ingredients'`    | object      |
     | `'n_ingredients'`  | int64       |
     | `'user_id'`        | float64     |
     | `'recipe_id'`      | float64     |
     | `'date'`           | object      |
     | `'rating'`         | float64     |
     | `'review'`         | object      |

1. Fill all ratings of 0 with np.nan.

   - Rating is generally on a scale from 1 to 5, 1 meaning the lowest rating while 5 means the highest rating. With that being said, a rating of 0 indicates missing values in rating. Thus, to avoid bias in the ratings, we filled the value 0 with np.nan.

1. Add column `'average_rating'` containing average rating per recipe.

   - Since a recipe can have numerous ratings from different users, we take an average of all the ratings to get a more comprehensive understanding of the rating of a given recipe.

1. Split values in the nutrition column to individual columns of floats.

   - Even though the values in the nutrition column look like a list, they are actually objects, which act like strings. Given by the description of the columns of the recipe dataset, we know what each individual values inside the brackets mean. To split up the values, we applied a lambda function then converted the columns to floats. It will allow us to conduct numerical calculations on the columns.

1. Convert submitted and date to datetime.

   - These two columns are both stored as objects initially, so we converted them into datetime to allow us conduct analysis on trends over time if needed.

1. Add `'is_dessert'` to the dataframe

   - `'is_dessert'` is a boolean column checking if the tags of recipes contain 'dessert' since desserts typically have a higher amount of sugar. This step separates the recipes into two groups, ones that are dessert and ones that are not. This provides us another way to compare ratings of recipes with more sugar and ones with less sugar.

1. Add `'prop_sugar'` to the dataframe
   - prop_sugar is the proportion of sugar of the total calories in a recipe. To calculate this, we use the values in the sugar (PDV) column to divide by 100% to get it in the decimal form. Then, we multiply by 25 to convert the values to grams of sugar since 25 grams of sugar is the 100% daily value (PDV). We got this value of 25 grams from experimenting on food.com with different amounts of sugar in a recipe. The experimentation allows us to understand the nutrition formula used on the website for recipes. Lastly, we multiply by 4 since there are 4 calories in 1 gram of sugar. After all these conversions, we end up with the number of calories of sugar, so we can divide by the total amount of calories in the recipe to get the proportion of sugar of the total calories. This data cleaning step is critical to allow us to make parallel comparisons on the amount of sugar in a recipe without concerns of extremely large values since all the values will be between 0 and 1.

#### Result
Here are all the columns of the cleaned df.

| Column                  | Description    |
| :---------------------- | :------------- |
| `'name'`                | object         |
| `'id'`                  | int64          |
| `'minutes'`             | int64          |
| `'contributor_id'`      | int64          |
| `'submitted'`           | datetime64[ns] |
| `'tags'`                | object         |
| `'nutrition'`           | object         |
| `'n_steps'`             | int64          |
| `'steps'`               | object         |
| `'description'`         | object         |
| `'ingredients'`         | object         |
| `'n_ingredients'`       | int64          |
| `'user_id'`             | float64        |
| `'recipe_id'`           | float64        |
| `'date'`                | datetime64[ns] |
| `'rating'`              | float64        |
| `'review'`              | object         |
| `'average rating'`      | object         |
| `'calories (#)'`        | float64        |
| `'total fat (PDV)'`     | float64        |
| `sugar (PDV)'`          | float64        |
| `'sodium (PDV)'`        | float64        |
| `'protein (PDV)'`       | float64        |
| `'saturated fat (PDV)'` | float64        |
| `'carbohydrates (PDV)'` | float64        |
| `'is_dessert'`          | bool           |
| `'prop_sugar'`          | float64        |


Our cleaned dataframe ended up with 234429 rows and 27 columns. Here are the first 5 rows of ~unique recipes of our cleaned dataframe for illustration. Since there is a lot of columns for the merged dataframe, we selected the columns that are most relevant to our questions for display. Scroll right to view more columns.

| name                                 |     id |   minutes | submitted           |   rating |   average rating |   calories (#) |   sugar (PDV) | is_dessert   |   prop_sugar |
|:-------------------------------------|-------:|----------:|:--------------------|---------:|-----------------:|---------------:|--------------:|:-------------|-------------:|
| 1 brownies in the world    best ever | 333281 |        40 | 2008-10-27 00:00:00 |        4 |                4 |          138.4 |            50 | True         |    0.361272  |
| 1 in canada chocolate chip cookies   | 453467 |        45 | 2011-04-11 00:00:00 |        5 |                5 |          595.1 |           211 | False        |    0.354562  |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30 00:00:00 |        5 |                5 |          194.8 |             6 | False        |    0.0308008 |
| millionaire pound cake               | 286009 |       120 | 2008-02-12 00:00:00 |        5 |                5 |          878.3 |           326 | True         |    0.371172  |
| 2000 meatloaf                        | 475785 |        90 | 2012-03-06 00:00:00 |        5 |                5 |          267   |            12 | False        |    0.0449438 |


### Univariate Analysis

For this analysis, we examined the distribution of the proportion of sugar in a recipe. As the plot below shows, the distribution skewed to the right, indicating that most of the recipes on food.com have a low proportion of sugar. There is also a decreasing trend, indicating that as the proportion of sugar in recipes gets higher, there are less of those recipes on food.com.

<iframe
  src="assets/univariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

For this analysis, we examined the distribution of the rating of the recipe conditioned between the sugary recipes and non-sugary recipes. The graph below shows that recipes with rating of 3, 4 and 5 are more likely to be non-sugary recipes while the recipes with rating of 1 and 2 are more likely to be sugary recipes. We would dive deeper to see if the difference in these proportions are significant in later sections.

<iframe
  src="assets/bivariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

For this section, we investigated the relationship between the cooking time in minutes and proportion of sugar of the recipes. First, we created a small dataframe, `'filter_df'` to store the cooking time in minutes without outliers. We identified the outliers using the IQR method. After grouping the cooking time and proportion of sugar in a pivot table shown below, we created a data visualization to understand it better.

| minutes | ('mean', 'prop_sugar') | ('median', 'prop_sugar') | ('min', 'prop_sugar') | ('max', 'prop_sugar') |
| ------: | ---------------------: | -----------------------: | --------------------: | --------------------: |
|       0 |              0.0137804 |                0.0137804 |             0.0137804 |             0.0137804 |
|       1 |                0.29681 |                 0.212177 |                     0 |               1.02985 |
|       2 |               0.316258 |                  0.25641 |                     0 |               1.06358 |
|       3 |               0.279901 |                  0.19305 |                     0 |               1.03192 |
|       4 |               0.276908 |                 0.235205 |                     0 |               1.04322 |
|     ... |                    ... |                      ... |                   ... |                   ... |
|     115 |               0.132994 |                0.0705617 |                     0 |              0.955342 |
|     116 |                 0.2303 |                   0.2303 |              0.133949 |              0.326652 |
|     117 |              0.0412412 |                0.0412412 |             0.0412412 |             0.0412412 |
|     118 |               0.378571 |                 0.378571 |              0.378571 |              0.378571 |
|     120 |               0.145882 |                0.0628323 |                     0 |               1.01558 |

Interestingly, the graph shows that as the cooking time increases the proportion of sugar in a recipe fluctuates more and more. According to the plot, tecipes take a long time could be either sugary or savory dishes. Also, the shapes of the line for mean and median looks very similar, especially for the recipes with shorter cooking time.

<iframe
  src="assets/interesting_agg.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Assessment of Missingness

Three columns, `'date'`, `'rating'`, and `'review'`, in the merged dataset have a significant amount of missing values, so we decided to assess the missingness on the dataframe.

### NMAR Analysis

We believe that the missingness of the `'review'` column is NMAR, because if people feel indifferent about the recipe, they are less likely to leave a review for it since they would feel like they have nothing significant to talk about. People usually will leave a review only if they have stronger emotions towards the recipe. Their emotions would motivate them to go onto the page, click multiple buttons to leave, and take some time out of their day to write a review. For example, people who enjoyed the recipe would be willing to do all the work to leave a good review for the recipe.

### Missingness Dependency

We moved on to examine the missingness of `'rating'` in the merged DataFrame by testing the dependency of its missingness. We are investigating whether the missiness in the `'rating'` column depends on the column `'prop_sugar'`, which is the proportion of sugar out of the total calories, or the column `'n_steps'`, which is the number of steps of the recipe.

> Proportion of Sugar and Rating

**Null Hypothesis:** The missingness of ratings does not depend on the proportion of sugar in the recipe.

**Alternate Hypothesis:** The missingness of ratings does depend on the proportion of sugar in the recipe.

**Test Statistic:** The absolute difference of mean in the proportion of sugar of the distribution of the group without missing ratings and the distribution of the group without missing ratings.

**Significance Level:** 0.05

<iframe
  src="assets/distr_rating_sugar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We ran a permutation test by shuffling the missingness of rating for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic.

<iframe
  src="assets/empirical_diff_sugar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** of **0.0063** is indicated by the red vertical line on the graph. Since the **p_value** that we found **(0.0)** is < 0.05 which is the significance level that we set, we **reject the null hypothesis**. The missingness of `'rating'` does depend on the `'prop_sugar'`, which is proportion of sugar in the recipe.

> Minutes and Rating

**Null Hypothesis:** The missingness of ratings does not depend on the cooking time of the recipe in minutes.

**Alternate Hypothesis:** The missingness of ratings does depend on the cooking time of the recipe in minutes.

**Test Statistic:** The absolute difference of mean in cooking time of the recipe in minutes of the distribution of the group without missing ratings and the distribution of the group without missing ratings.

**Significance Level:** 0.05

<iframe
  src="assets/empirical_diff_prescale.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Due to the outliers in cooking time, it is difficult to identify the shapes of the two distributions, so we update the scale to take a closer look.

<iframe
  src="assets/distr_rating_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We ran another permutation test by shuffling the missingness of rating for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic.

<iframe
  src="assets/empirical_diff_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** of **51.4524** is indicated by the red vertical line on the graph. Since the **p-value** that we found **(0.123)** is > 0.05 which is the significance level that we set, we **fail to reject the null hypothesis**. The missingness of rating does not depend on the cooking time in minutes of the recipe.

## Hypothesis Testing

As mentioned in the introduction, we are curious about whether people rate sugary recipes and non-sugary recipes on the same scale. By sugary recipes, we are talking about recipes with a proportion of sugar higher than the average proportion of sugar. Proportion of sugar is referring to the values in `'prop_sugar'`, which are the proportion of sugar in calories out of the total calories of the recipe.

To investigate the question, we ran a **permutation test** with the following hypotheses, test statistic, and significance level.

**Null Hypothesis:** People rate all the recipes on the same scale.

**Alternative Hypothesis:** People rate sugary recipes lower than non-sugary recipes.

**Test Statistic:** The difference in mean between rating of sugary recipes and non-sugary recipes.

**Significance Level:** 0.05

The reason we chose to run a permutation test is because we do not have any information of any population, and we want to check if the two distributions look like they come from the same population. We proposed that **people rate the sugary recipes lower** because people might be concerned with the negative health risks relating to the recipe, and we would like to know all the opinions from the users, so we used rating instead of average rating of the recipes. For the test statistic, we chose the difference in mean of the ratings of two groups of recipes instead of absolute difference in mean. This is because we have a directional hypothesis, which is that people rate sugary recipes lower than other recipes. By looking at the difference in mean between the two groups, we can see what type of recipes typically have a higher rating, which answers our question.

To run the test, we first split the data points into two groups, sugary, which are recipes with proportion of sugar higher than the mean proportion of sugar, and the rest of the data points are in the non-sugary group. The **observed statistic** is **-0.0097**.

Then we shuffled the ratings for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic. We got a **p-value** of **0.001**.

<iframe
  src="assets/empirical_diff_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Conclusion of Permutation Test

Since the **p-value** that we found **(0.002)** is less than the significance level of 0.05, we **reject the null hypothesis**. People do not rate all the recipes on the same scale, and they tend to rate sugary recipes lower. One plausible explanation for this founding could be that people are concerned with health risks relating to sugary recipes, such as diabetes.

## Framing a Prediction Problem

We plan to **predict average rating of a recipe** which would be a **classification problem** since we can treat rating as a ordinal categorical variable if we round the average rating so that we only have [1, 2, 3, 4, 5] as possible values. To address our prediction problem, we will build a multi-class classifier since our average ratings have 5 possible values that the model will predict from.

We chose the average rating of a recipe as a response variable because it is a good representation of the overall rating of a recipe. We have also previously found significant correlation between rating and sugary recipes, which are recipes with proportion of sugar higher than the average proportion of sugar, so we may be able to predict the rating through the proportion of sugar.

To evaluate our model, we will use the f1 score instead of accuracy, because the distribution for the ratings are heavily skewed left with most ratings concentrated in the higher ratings (4-5). This means that there are more recipes with higher average ratings. If we use accuracy, the model's performance may be misleading due to the imbalanced classes.

The information we have prior making our prediction are all the columns in the `rating` dataset, which are listed in the introduction section. All those columns are features relating to the recipes themselves, thus we would have access to it even though no one has made a rating and review on them.

## Baseline Model

For our baseline model, we are utilizing a random forest classifier and split the data points into training and test sets. The features we are using for this model is `'prop_sugar'`, a column containing quantitative numerical values, and `'is_dessert'`, a column containing nominal values since they are boolean values.

We one hot encoded the boolean values in `'is_dessert'` with the corresponding 0 and 1 values and dropped one of the encoded columns. This step allows us to train the model appropriately.

The metric, **F1 score**, of this model is **0.87**. The F1 score for each rating categoires are 0.20, 0.47, 0.50, 0.74, and 0.92 for rating of 1s, 2s, 3s, 4s, and 5s respectively. The metrics let us know that the model predicts better for rating of 4s and 5s and not as accurate for the lower ratings. The reason for this could be that there are more recipes with rating 4s and 5s in the dataset compared to other ratings. With the more data points, the model predicted better for higher ratings.

## Final Model

For the final model, we used `'is_dessert'`, `'minutes'`, `'calories (#)'`, `'submitted'`, and `'prop_sugar'` as the features.

`'is_dessert'`

The column categorizes the data as dessert or not dessert by checking if the recipe's tags contain 'dessert'. We chose this feature because based on the the bar graph we construsted between `'average rating'` and `'is_dessert'`, we saw that for the higher ratings (4 and 5) there are less dessert recipes. This trend might be useful in helping the model predict the average rating of a recipe. We one hot encoded this column like we did for the baseline model.

`'minutes'`

The column is the cooking time of the recipe in minutes. By constructing a bivariate table of the `'minutes'` and `'average rating'`, we learned that the recipes took longer to cook tend to have medicore ratings, like ratings of 2 or 3. The differences between the mean minutes among the different ratings makes us believe that it could help with our predicition model. It is also reasonable that a recipe that takes long to make would lead to lower rating since people are busy nowadays and lack patience. We used `StandardScaler` to standardize the `'minutes'` feature to guarantee that the cooking time are in a comparable range since some recipes has extremely long cooking times.

`'calories (#)'`

The column contains the total calories of the recipe. By constructing a bivariate table of the `'calories (#)'` and `'average rating'`, we learned that recipes with higher rating typically has less calories. A lower calories generally indicates that the recipe is healthier, thus it is logical that it has a higher rating. Knowing this, we think the relationship between `'calories (#)'` and `'average rating'` would help our model predict better. To transform the `'calories (#)'` feature, we used `RobustScaler`, which scales the numerical features while handling outliers effectively. From the EDA, we learned that the columns contains many outliers, and it might introduces bias to our model, so we engineered the feature to minimize the ourlier effects.

`'submitted'`

The column contains information of the date that the recipe was submitted. In data cleaning process, we converted the column to be `datetime[ns]` and now we pulled out only the year using `FunctionTransformer`. When we created a table of `'submitted'` and `'average rating'`, we noticed that recipes submitted in recent years has a lower ratings. This could be due to the lack of novelity of newer recipes since most of the classic recipes might already posted on the website. The trend between `'submitted'` and `'average rating'` could be also useful in improving our model.

`'prop_sugar'`

As mentioned numerous time in earlier sections, this column contains information of the proportion of sugar in calories out of the total calories of the recipe. According to our hypothesis testing, people seems more likely to rate a sugary recipe lower than recipes that are not sugary. This takeaway motivates us use this feature since the relationship could be a significant deciding factor when making predicition on the `'average rating'`. For this feature, we will leave the column as it is.

We used `RandomForestClassifier` as our modeling algorithm and conducted `GridSearchCV` to tune the hyperparameters of `max_depth` and `n_estimators` of the `RandomForestClassifier`. Decision trees are prone to high variance, and the two hyperparameters we chose serve a way to control the variance and avoid overfitting the training set. The best combination of the hyperparameters is 42 for the `max_depth` and 142 for the `n_estimators`.

The metric, **F1 Score**, of the final model is **0.92**, which is a 0.05 increase from the F1 Score of the baseline model. Moreover, the F1 score of each of the rating also improved. The F1 score for each rating categoires are now 0.36, 0.66, 0.68, 0.85, and 0.95 for rating of 1s, 2s, 3s, 4s, and 5s respectively.

## Fairness Analysis

For our fairness analysis, we split the recipes into two groups: high calories and low calories. We designated high calorie recipes to be ones with calories > 301.1 and low calorie recipes to be ones with calories <= 301.1. We found that the median calories for our data set is **301.1** which is why we chose it as the threshold. We used median instead of mean, because we previously found that calories had many high outliers which can skew our results. We chose to evaluate the **precision parity** of the model for the two groups, because we think it’s more important for the model to correctly identify the rating of a recipe among all instances of that rating. False positives would not be good since it would mislead users with the incorrectly labeled ratings. False positives would not be good since it would mislead users with the incorrectly labeled ratings. For example, if we predicted recipes with lower calories to have a bad rating, people would be discouraged to try them. For recipes with lower calories, it wouldn’t be good to mislabel them, as low calorie recipes may be healthier for people.

**Null Hypothesis**: Our model is fair. Its precision for recipes with higher calories and lower calories are roughly the same, and any differences are due to random chance.

**Alternative Hypothesis**: Our model is unfair. Its precision for recipes with lower calories is lower than its precision for recipes with higher calories.

**Test Statistic**: Difference in precision (low calories - high calories)

**Significance Level**: 0.05

<iframe
  src="assets/empirical_precision.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

To run the permutation test, we created a new column `is_high_calories` to differentiate between the low and high calorie recipes. When we took the difference in their precision, we got an observed test statistic of **-0.023**. We shuffled the `is_high_calories` column for 1000 times to collect 1000 simulating differences in the two distributions as described in the test statistic. After running our permutation test, we got a p-value of **0.0**. Since the p-value of 0.0 is less than 0.05, we reject the null hypothesis that our model is fair. The model's precision for recipes with lower calories is lower than its precision for recipes with higher calories.
