## Overview

In this data science project, I will be exploring the relationship between recipes with high protein and cooking time to determine the feasibility of healthy eating.

## Introduction

If you want to prioritize your health, nutrition is one of the first things you should be looking towards. Overall, one of the biggest improvements you can make to your diet is increaing the amount of protein you eat. According to a National Health and Nutrition Examination Survey, up to 46% of the oldest participants did not consume enough protein. This is concerning, as protein is extremely important for muscle protein synthesis, satiety, metabolism support, and immune function. However, for some people, eating higher protein meals can seem like a daunting task, as cooking meats may take longer than other recipes, or just simply eating out. **Therefore, my focus is going to be on the relationship between high protein recipes and cooking time.** I will also examine ratings on the side to see if it is feasible to eat both healthy and delicious food. To analyze this, I will be utilizing two datasets consisting of recipes and ratings posted on [food.com](https://www.food.com/).

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

**Given thse datasets, my goal is to see if the distribution of cooking times for higher protein recipes is the same as lower protein recipes.** To explain protein in greater depth, I used the information in the `'nutrition'` column to create `'protein_prop'` and `'is_high_protein'` columns. I will go into further depth on how this was achieved in the data cleaning section.

Through this analysis, I will be able to debunk whether or not it is harder to eat healthier and just how much harder it is. I hope this exploration can motivate more people to eat healthier, since health is wealth!

## Data Cleaning and Exploratory Data Analysis

To make analysis possible, the data underwent these steps for cleaning.

1. Left merge the recipes and interactions datasets on id and recipe_id.

   - This step is essential to matching all ratings with their corresponding recipes.

2. Fill all ratings of 0 with np.nan.

   - The dataset's designated 0's for missing ratings. However, a 0 can cause confusion when computing the mean of the ratings for a recipe, as a missing value doesn't equate to a low rating.

3. Add column `'avg_rating'` containing average rating per recipe.

   - Creating a more comprehensive way to analyze a given recipes rating. Note that np.nans are not included in this mean computation.

4. Change values in the nutrition column to lists of floats.

   - The values of the nutrition column are actually strings that are formatted like lists. So, I converted them to actual lists with float values in them as to make the nutritional data easier to access.

5. Drop submitted column.

   - These two columns are both stored as objects initially, so we converted them into datetime to allow us conduct analysis on trends over time if needed.

6. Create `'protein_prop'` column and add it to the dataframe.

    - `'protein_prop'` contains the proportion of calories that come from protein in a recipe. This was calculated using the nutrition PDV's. The recommended amount of protein is 50 grams, so I multiplied the PDV divided by 100 by 50 to get the grams of protein. Each gram of protein is 4 calories, so then I multiplied by 4 to get the number of calories from protein, and then divided that by the total calories which was also in the nutrition column.

7. Create `'is_high_protein'` column and add it to the dataframe.

    - Used the mean protein proportion from the `'protein_prop'` column as the threshold and split rows into high protein and low protein, with boolean values in the column.

8. Add `'calories'` column to the dataframe.

   - Used the `'nutrition'` column to add calories.

9. Reformat `'tags'` and `'steps'` columns into a lists of strings.

    - Similar the the `'nutrition'` column, the `'tags'` and `'steps'` columns didn't contain only lists. I had to parse through and reformat both lists and strings into the list of string format I wanted.

10. Create `'is_easy'` column by looking through `'tags'` column.

    - Parsed through the tags of each recipe to see if there was the tag "easy". This column contains boolean values.

11. Removed outliers using IQR.

    - Found that there wer unreasonable outliers in both the `'minutes'` and `'calories'` columns. To remove outliers, I calculated the interquartile range of each column, and dropped any values 2.5 IQRs above Q3 and any values 2.5 IQRs below Q1.

12. Dropping unnecessary columns.

    - For the analysis, I don't need `'user_id'` as it is the same as id, or `'contributor_id'`. I also don't use the date submitted for any of my analyses, so I dropped the `'submitted'` and `'date'` columns as well.


#### Result
Here are all the columns of the cleaned dataframe.

| Column                  | Description    |
| :---------------------- | :------------- |
| `'name'`                | object         |
| `'id'`                  | int64          |
| `'minutes'`             | int64          |
| `'tags'`                | object         |
| `'nutrition'`           | object         |
| `'n_steps'`             | int64          |
| `'steps'`               | object         |
| `'description'`         | object         |
| `'ingredients'`         | object         |
| `'n_ingredients'`       | int64          |
| `'rating'`              | float64        |
| `'review'`              | object         |
| `'avg_rating'`          | object         |
| `'protein_prop'`        | float64        |
| `'calories'`            | float64        |
| `'is_easy'`             | bool           |
| `'is_high_protein'`     | bool           |


My cleaned dataframe ended up with 209621 rows and 17 columns. Here are the first 5 rows of ~unique recipes of the cleaned dataframe for illustration. I selected a few columns of interest to be displayed.

| name                                 |   minutes |   avg_rating |   calories |   protein_prop |
|:-------------------------------------|----------:|-------------:|-----------:|---------------:|
| 1 brownies in the world    best ever |        40 |            4 |      138.4 |      0.0433526 |
| 1 in canada chocolate chip cookies   |        45 |            5 |      595.1 |      0.0436901 |
| 412 broccoli casserole               |        40 |            5 |      194.8 |      0.225873  |
| millionaire pound cake               |       120 |            5 |      878.3 |      0.0455425 |
| 2000 meatloaf                        |        90 |            5 |      267   |      0.217228  |

### Univariate Analysis

For this analysis, I plotted the distribution of protein proportion among all recipes, making sure to drop any duplicate columns from rating. The plot shows that a majority of recipes are generally lower in protein, as it is skewed right. We can see that the mean protein proportion is around 0.15 or 15%.

<iframe
  src="assets/protein_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

For this analysis, I examined the relationship between a recipe being high protein and it's rating. Overall, you can see there is a disproportionate amount of ratings, with there mainly being five star ratings. However, you can see that in those five star ratings, there are significantly less 5 star ratings for recipes with higher protein. So it is possible that recipes with higher protein may not be as tasty.

<iframe
  src="assets/rating_protein_barplot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

For this section, I further investigated the relationship and amount of protein in a recipe. I generated a pivot table of protein proportion statistics conditioned on their rating. Overall, you can see differences in protein proportions among ratings, but it is still difficult to determine anything due to the larger number of 4 and 5 star ratings. 5 star recipes typically have a lower protein proportion than 4 and 3 stars.

|   rating |   ('mean', 'protein_prop') |   ('median', 'protein_prop') |   ('count', 'protein_prop') |   ('std', 'protein_prop') |
|---------:|---------------------------:|-----------------------------:|----------------------------:|--------------------------:|
|        1 |                   0.137561 |                    0.0996116 |                        2406 |                  0.120587 |
|        2 |                   0.14933  |                    0.113636  |                        2023 |                  0.124265 |
|        3 |                   0.161463 |                    0.131393  |                        6328 |                  0.128332 |
|        4 |                   0.16567  |                    0.136054  |                       33487 |                  0.129738 |
|        5 |                   0.153861 |                    0.120573  |                      152673 |                  0.127425 |

## Assessment of Missingness

Only three columns, `'description'`, `'rating'`, and `'review'`, in the cleaned dataset have a significant number of missing values.

### NMAR Analysis

Looking at the `'review'` column, I believe that its missingness is NMAR, which means its missingness depends on the value itself. The review column will be missing when people feel indifferent about a recipe and don't want to take the time to write out a review. However, we are unable to see if the person actually feels indifferent because we don't have the review itself. A way to prove whether or not this is the case is a mood score. If reviewers also included their mood after trying the recipe, we could truly determine if it is NMAR or MAR.

### Missingness Dependency

Next, I evaluated whether or not the missingness of `'rating'` in the merged DataFrame is MAR by testing the dependency of its missingness on two other columnes `'protein_prop'`, which is the proportion of protein in a recipe, and the column `'minutes'`, which is the number of minutes a recipe takes.

#### Sidenote

At first, I tried to determine missingness dependency doing permutation tests on the entire dataframe of values. I used absolute difference in means, and for every single column I tried, it would tell me that there was significance, even though the plotted KDE's showed their similarity. This is due the fact that there are around 12700 missing ratings and around 200000 total points. To remedy this, I used subsampling, where I subsampled 500 values from rating missing, and 500 from rating not missing, and computed the p-value of that KS statistic for 50 different sub samplings. The final p-value was the average of those 50 p-values.

> Protein Proportion and Rating

**Null Hypothesis:** The missingness of ratings doesn't depend on the proportion of protein in a recipe.

**Alternate Hypothesis:** The missingness of ratings does depend on the proportion of protein in a recipe.

**Test Statistic:** The average p value of 50 KS Statistics from subsamples.

**Significance Level:** 0.10
<iframe
  src="assets/missing_protein_kde.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of the 50 p-values I calculated.

<iframe
  src="assets/pvals_protein.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **average p-value** of **0.347** is indicated by the red vertical line on the graph. Since the **p_value** is greater than the significance value, we **fail to reject the null hypothesis**. It cannot be determined if the missingness of `'rating'` depends on `'protein_prop'`.

> Minutes and Rating

**Null Hypothesis:** The missingness of the rating column doesn't depend on the minutes a recipe takes.

**Alternate Hypothesis:** The missingness of the rating column does depend on the minutes a recipe takes.

**Test Statistic:** The average p value of 50 KS Statistics from subsamples.

**Significance Level:** 0.10

<iframe
  src="assets/missing_minutes_kde.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of the 50 p-values I calculated. Notice how, overall, p-values are very low, and the mean is affected by a few outliers, which is why I chose a significance level of 0.10.

<iframe
  src="assets/pvals_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **average p-value** of **0.090** is indicated by the red vertical line on the graph. Since the **p_value** is less than the significance value, we **reject the null hypothesis**. The missingness of `'rating'` depends on `'minutes'`, and thus is MAR.

## Hypothesis Testing

The main goal of this project is to see if cooking high protein recipes take longer. From the previous analyses, we can see that it is possible that higher protien foods aren't as tasty due to lower ratings. Now, let's see if they take longer to cook using a **permutation test** by shuffling the `'is_high_protein'` column.

**Null Hypothesis:** All meals come from the same distribution of minutes to cook.

**Alternative Hypothesis:** Higher protein meals take longer to cook.

**Test Statistic:** The difference in mean between minutes to cook of high protein recipes and low protein recipes.

**Significance Level:** 0.05

The reason I'm using a permutation test is because we do not have any information of any population, and we want to check if the two distributions look like they come from the same population. I assume that **higher protein recipes take longer to cook** because cooking meat takes longer than other recipes because you have to ensure that it is fully cooked. For the test statistic, I'm using the difference in mean of the ratings of two groups of recipes instead of absolute difference in mean. This is because the hypothesis is directional, as we inferred that higher protein recipes take longer. Using this test statistic, we can clearly see, on average, how much longer protein recipes take.

To run the test, I split the data points into two groups by using the `'is_high_protein'` column. The **observed statistic** is **5.768**.

Then we shuffled the `'is_high_protein'` column 1000 times to collect 1000 simulated mean differences in the two distributions as described in the test statistic. As you can see, the **p-value** is **0.000**.

<iframe
  src="assets/main_perm_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Conclusion of Permutation Test

Since the **p-value** was **(0.000)**, it is less than the significance level of 0.05. Therfore, we **reject the null hypothesis**. Overall, higher protein recipes take longer to cook than lower protein recipes.

## Framing a Prediction Problem

I plan to **predict the minutes a recipe takes to cook** which would be a **linear regression problem** since minutes is a continuous variable. To achieve this, I will build a linear regression model to predict the minutes using features of my choice.

The variable I'm trying to predict, minutes, is a good measure of how time consuming cooking healthy meals is. Also, from the previous permutation test, I have seen that minutes and protein proportion are highly correlated, with higher protein proportion correlating to longer minutes to cook. As such, it is possible to predict minutes to cook using protein proportion.

To test the model's accuracy, I will be using the root mean squared error, because it is the best measure of loss between an actual and predicted set of continuous variables.

The information we have prior is all columns of our dataset except `'rating'`. Also, in the `'steps'` column, some recipes include cook times, but I will not be using those to predict cooking time, as I just want to use the characteristics of a recipe.

## Baseline Model

For the baseline model, I used a Linear Regression model trained on data points split into training and test sets using `train_test_split`. The two features I used are `'protein_prop'` and `'calories'`, both columns with quantitative values.

I did not do any transforming of `'protein_prop'`, but I did use the FunctionTransformer on `'calories'`, applying a log(x+1) transformation because the data was right skewed.

The metric, **RMSE**, of this model is **28.66**. This is not good, as the average value of the `'minutes'` column is around 40. Therefore, the model's predictions are typically off by 75% of the mean.

## Final Model

For the final model, I used `'is_high_protein'`, `'n_steps'`, `'calories'`, `'n_ingredients'`, `'is_easy'`, and `'cooking_method'` as the features.

`'is_high_protein'`

This column uses `'protein_prop'` to categorize a recipe whether or not it's protein proportion is above or below the mean. I used this variable because we saw significant correlation between protein and minutes to cook in the hypothesis testing section. This column was OneHotEncoded in the pipeline. It is important to note that `'protein_prop'` performs exactly the same as this column, I just decided to go with this one.

`'n_steps'`

This column is the number of steps a recipe is. It is reasonable to assume that a recipe with more steps will take longer than a recipe with less steps, which is why I chose it for my model. I did not use any transformations on this column.

`'calories'`

The column contains the total calories of the recipe. It is reasonable to assume that the more calories a recipe contains, the more food it is, and thus a longer cooking time. Also, more calorie dense foods, such as deserts and meats require longer cooking times. I used the same FunctionTransformer as the baseline, applying log(1+x) to reduce the effect of the right skew.

`'n_ingredients'`

This column is the number of ingredients a recipe contains. It is reasonable to assume that a recipe with more ingredients will take longer than a recipe less ingredients, as you have to combine or prep more items. I did not use any transformations on this column.

`'is_easy'`

This column is a boolean for whether or not the tag "easy" was in the recipe tags. It is reasonable to assume that easier recipes involve less moving parts, and easier techniques of cooking. As such, easier recipes should take a shorter time to cook than hard. Since the values are nominal, I used OneHotEncoding for this column.

`'cooking_method'`

I created this column just for the linear regression model. I thought it would be a good idea to include the primary cooking method of the recipe, as recipes with ovens often take longer due to having to preheat and cook. Other methods include boiling, and using a pan/skillet/grill. The columns's values are nominal, so this was also OneHotEncoded.

I used `LinearRegression` as my modeling algorithm and manually created different combinations of features to test out which was best. I created mulitple pipes and used a cross validation level of 5 to estimate RMSE for each combination of features, and my final model did best will all of the aforementioned features.

The metric, **RMSE**, of the final model is **25.99**, which is a 2.67 improvement from the RMSE of the baseline model. This model performs better than the baseline, however, overall, it's predictions are still pretty far form optimal.

## Fairness Analysis

For the fairness analysis, I split the recipes into two groups, is dessert and is not dessert by parsing through the `'tags'` column. I chose to evaluate the **RMSE** for the two groups. 

**Null Hypothesis**: The model is fair. It's RMSE for predicting the cooking time for dessert recipes and non desert recipes is roughly the same.

**Alternative Hypothesis**: The model is unfair. It performs worse at predicitng cooking time for dessert recipes.

**Test Statistic**: Difference in RMSE (is dessert - is not desert)

**Significance Level**: 0.05

<iframe
  src="assets/fairness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

To run the permutation test, I created a new column `'is_dessert'` to differentiate between the dessert and non dessert recipes. I then used the final model to predict minutes to cook each recipe, and then computed an **observed statistic** of **1.218**. Then I shuffled the `'is_dessert'` column 1000 times, computing the difference in RMSE's to build a null distribution. The **p-value** was **0.048** which is less than our significane level of 0.05, so re reject the null hypothesis. The model performs worse for deserts, which makes since, as they are lower in protein and often use ovens.