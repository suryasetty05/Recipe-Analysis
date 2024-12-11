# Stirring Up Insights Through Data Analysis and Prediction
This is a project for DSC80 about recipes.

# Introduction

The general rise of obesity in recent years has been attributed by the National Institute of Health (NIH) to a suboptimal diet that maximizes added sugars, saturated and trans fats, and sodium. These diets implicate that most of the food we eat contains a disproportionate amount of sodium, added sugar, and bad fats compared to our bodily needs. Much of the rise of sodium can be attributed to salty foods, while bad fats can be traced to partially hydrogenated vegetable oils, common in fried foods, margarine, and shortening for baked goods. Saturated fats often come from fatty meats, cheese, and butter.
https://www.nhlbi.nih.gov/news/2016/new-dietary-guidelines-urge-americans-eat-less-added-sugars-saturated-fat-and-sodium

The rise of these dietary trends should be correlated with consumer preference, which is what makes it difficult to change eating habits or preferences. As a result, it could be evident that diets that contain more bad fats sodium are related to how good consumers would rate it. Taking data of recipes and their rating, I will answer the initial question of: 

**Do recipes with high sodium content or recipes that contain bad fats are more complicated than the average recipe?**

The dataset recipe, from food.com, has 83782 rows and 12 columns. Relevant columns include "nutrition", starting with the number of calories, followed by the percent daily value of total fat, sugar, sodium, protein, saturated fat, and carbohydrates. 

| Columns     | Info    |
|-------------|-------------|
| name | name of recipe (str) |
| id | unique id corresponding to each recipe (int) |
| minutes | how long the recipe takes to make (int) |
| nutrition | as described above (str) |
| n_steps | number of steps to make recipe (int) |
| steps | steps needed to make recipe (str) |
| ingredients | steps needed to make recipe (str) |
| n_ingredients | number of ingredients needed to make recipe (int) |
| submitted | date submitted (str) |

The dataset interactions, from food.com, has 731927 entries with relevant columns below:

| Columns     | Info    |
|-------------|-------------|
| user_id | id of user submitting review (int) |
| recipe_id | id of corresponding recipe (int) |
| rating | star rating (1-5) (int) |
| review | number of steps to make recipe (int) |
| date | date review submitted (str) |

The strategy I will use in this analysis is to connect sodium or fats in the nutrition column to the number of ingredients, steps, and minutes. In doing so, we may find correlations between unhealthy foods and the health effect.

# Exploratory Data Analysis, Data Cleaning

Since we're working with this dataframe, my goal is to "match" reviews to their recipes, and in this process I'll end up comparing the nutritional values of sodium and saturated fats for each recipe, and plot them by their average review.

Some columns, like 'nutrition', contain values that look like lists, but are actually strings that look like lists. You may want to turn the strings into actual lists, or create columns for every unique value in those lists. For instance, per the data dictionary, each value in the 'nutrition' column contains information in the form "[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), and carbohydrates (PDV)]"; you could create individual columns in your dataset titled 'calories', 'total fat', etc. I'll need to extract the related data for data analysis.

I'll start with preliminary data cleaning.

1. Left merge recipes and interactions to connect recipes with their respective reviews, such that recipes have meaning.
2. Replace the 0s in the rating column to be np.nan. It was used as a placeholder value as the data was generated, but you can't rate a recipe 0 stars on food.com.
3. Create an average rating column by using groupby and means to create an average rating column (so we know how good a recipe is relatively to other recipes), placing the results into recipes.
4. Convert the nutrition values to become floats and have their own corresponding columns.
5. Convert the submitted column to datetimes, to be used to see changes over time.
6. Filter recipes to only include columns whose 'calories(#)' are between the 0.15 and 0.85 quantile, as extreme outliers may heavily skew the data.
7. Drop columns that I won't use in this analysis. Some columns like tags, are subjective and may depend on the user's input. Other columns like contributer_id and user_id are nominal, and would result in poor data quality.

Here's a view of the cleaned dataframe of recipes:
| name                                 |   minutes | submitted           | nutrition                                    |   n_steps |
|:-------------------------------------|----------:|:--------------------|:---------------------------------------------|----------:|
| 1 brownies in the world    best ever |        40 | 2008-10-27 00:00:00 | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0]     |        10 |
| 1 in canada chocolate chip cookies   |        45 | 2011-04-11 00:00:00 | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0] |        12 |
| 412 broccoli casserole               |        40 | 2008-05-30 00:00:00 | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0]    |         6 |

|   n_ingredients |   average_rating |   calories(#) |   total_fat |   sugar |   sodium |   protein |   saturated fat |   carbohydrates |
|----------------:|-----------------:|--------------:|------------:|--------:|---------:|----------:|----------------:|----------------:|
|               9 |                4 |         138.4 |          10 |      50 |        3 |         3 |              19 |               6 |
|              11 |                5 |         595.1 |          46 |     211 |       22 |        13 |              51 |              26 |
|               9 |                5 |         194.8 |          20 |       6 |       32 |        22 |              36 |               3 |

An example of an element in steps, which is too large to display: "['heat the oven to 350f and arrange the rack in the middle', 'line an 8-by-8-inch glass baking dish with aluminum foil', 'combine chocolate and butter in a medium saucepan and cook over medium-low heat , stirring frequently , until evenly melted', 'remove from heat and let cool to room temperature', 'combine eggs , sugar , cocoa powder , vanilla extract , espresso , and salt in a large bowl and briefly stir until just evenly incorporated', 'add cooled chocolate and mix until uniform in color', 'add flour and stir until just incorporated', 'transfer batter to the prepared baking dish', 'bake until a tester inserted in the center of the brownies comes out clean , about 25 to 30 minutes', 'remove from the oven and cool completely before cutting']"

An example of an element in ingredients, which is too large to display: ['bittersweet chocolate', 'unsalted butter', 'eggs', 'granulated sugar', 'unsweetened cocoa powder', 'vanilla extract', 'brewed espresso', 'kosher salt', 'all-purpose flour']

# Exploratory Data Analysis

<iframe
  src="assets/num_ingredients.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This graph showing the distribution of the number of ingredients shows that most ingredients are fairly simple, concentrated around 6-11 elements. I'll test if this is related to sodium in the hypothesis testing section.

<iframe
  src="assets/avg_sat_fat.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

This tracks the rise of saturated fats and total fats, which include trans fats. This shows that the NIH was right about the rise of the unhealthy diet!

<iframe
  src="assets/steps.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

This trend is suspicious, it suggests the rise of fats is heavily correlated with the increase of steps. Let's check out if sodium has the same trend.

<iframe
  src="assets/high_sodium.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

For this plot, I assigned high_sodium to recipes with sodium levels that are higher than the 75th quartile of sodium, which is a daily value intake of 29%. This shows that recipes with high sodium
might have started shifting the average steps up.

# Aggregates

I created the column sat_fat_prop, created by convert saturated fat to calories (by multiplying by 9) and then dividing by calories for every recipe. The values are n_ingredients and n_steps. We can see that smaller sat_fat_props correspond to having more steps and ingredients. This implies that the more saturated fat a recipe has, the more likely that it has less ingredients it has (or the converse). 

| sat_fat_prop | n_ingredients | n_steps  |
|---------------|----------------|----------|
| 0             | 6.82355        | 7.19078  |
| 0.1           | 9.71026        | 9.36697  |
| 0.2           | 10.0299        | 9.71636  |
| 0.3           | 9.90993        | 9.74334  |
| 0.4           | 10.0542        | 9.8965   |
| ...           | ...            | ...     |
| 4.1           | 3              | 7       |
| 4.2           | 4              | 6       |
| 4.3           | 4.5            | 7.75    |

# Assessment of Missingness

| column       |    Number Missing   |
|:-------------|-------:|
| name         |     1  |
| description  |   114  |
| user_id      |     1  |
| recipe_id    |     1  |
| date         |     1  |
| rating       | 15036  |
| review       |    58  |
| year         |     1  |

The dataframe above shows the number of missing values for the reviews (the merged dataframe from earlier) dataframe. As we can see, the rating, description, and review column have a significant number of missing values.

| year | review_missing |
|------|----------------|
| 2008 |              0 |
| 2009 |              0 |
| 2010 |              0 |
| 2011 |              0 |
| 2012 |              0 |
| 2013 |              4 |
| 2014 |              0 |
| 2015 |              0 |
| 2016 |              4 |
| 2017 |             33 |
| 2018 |             16 |

The column 'review' may be Not Missing At Random (NMAR) because its missingness depends on when the review was submitted, or which moderators might delete reviews. For example, the years 2017 and 2018 had a disproportionate amount of missing values, which may depend on the discretion of the moderation team to remove reviews, which may have become more strict in recent years.

In these next steps, I'm going to test if the missingness of the 'rating' column is dependent on certain features.



Driving Question: Is the missingness of rating dependent on the number of ingredients of the recipe? If people have more steps, they may be more likely to get lost.

Null Hypothesis: The missingness of rating is not dependent on the number of ingredients of the recipe. The missingness of rating depends on random chance.

Alternative Hypothesis: Ratings are more likely to be missing for recipes with too many ingredients. People might get lost, forget ingredients, or substitute ingredients so they feel like their version they made is different than the food.com recipe.

Significance Level: 0.05

Test statistic: Difference in means, missing[True] - missing[False]

<iframe
  src="assets/n_ingredients_permute.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

The p_value is 0, and the observed mean difference shows that recipes with missing ratings have more ingredients on average. Since the p_value is less than the significance level, we reject the null hypothesis.



Driving Question: Is the missingness of rating dependent on the how long the recipe takes? If people aren't patient, they might feel like they don't want to spend more time on anything related to the recipe.

Null Hypothesis: The missingness of rating is not dependent on how long the recipe takes. The missingness of rating depends on random chance.

Alternative Hypothesis: Ratings are more likely to be missing for recipes that take a longer period of time.

Significance Level: 0.05

Test statistic: Difference in means, missing[True] - missing[False]

<iframe
  src="assets/minutes_permute.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

Our observed mean difference is within the range of the histogram, and this is shown in the p_value of 0.135, which is greater than the significance level 0.05. This test fails to reject the null hypothesis.

# Hypothesis Testing

Let's check if high sodium foods have more steps and ingredients.

Null Hypothesis: The distribution of the number of steps is not dependent on if the food has more sodium in it.

Alternative Hypothesis: Foods with more sodium have more steps, on average, such that the number of steps is correlated or dependent on sodium.

Test Statistic: Difference in means, high_sodium[True] - high_sodium[False] -> permute high_sodium

Significance Level: 0.05

<iframe
  src="assets/sodium_steps.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

The observed value is 0.305, greater than the mode of 0 and the distribution of the difference of means. The p_value is 0.0, lower than the significance level of 0.05. This rejects the null hypothesis.


Null Hypothesis: The distribution of the number of ingredients is not dependent on if the food has more sodium in it.

Alternative Hypothesis: Foods with more sodium have more ingredients, on average, such that the number of ingredients is correlated or dependent on sodium.

Test Statistic: Difference in means, high_sodium[True] - high_sodium[False] -> permute high_sodium

<iframe
  src="assets/sodium_ingredients.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

The observed statistic is 1.346, greater than the average near 0 and the p_value is 0.0. This is lower than our significance level of 0.05. This rejects the null hypothesis.

# Framing a Prediction Problem

As we've seen in previous sections, the number of steps seems correlated with our nutrition data. Though n_steps is a discrete random variable, the continuous form of n_steps that results from regression still represents numerical data the best. It can also be converted easily back to a discrete variable for use through rounding. The reason I'm choosing a linear regression classifier over a decision tree regressor is because the linear regression classifier can also take advantage of categorical data. For example, the word carrot in ingredients might mean that the carrots should be sliced or grated. I used name so that we can predict that 'cakes' likely take a longer time to make than 'apples' contributing to a higher score.

The metric that I'm using is R^2 or equivalently the score method on the linear regression qualifier. It minimizes residuals. Compared to ridge regression, the model should not care about whether the coefficients are large or small, as the data is not all standardized. Compared to lasso regression, the default linear regressor is more interpretable and makes more sense if I'm selecting features versus whether the model selects features.

For this section, the model will not have access to the steps (str) column from earlier that contains a list of steps, or n_steps, our y-value.

# Baseline Model

In the baseline model, I used LinearRegression() from sklearn. I used 3 quantitative columns, 'sodium', 'saturated fat' and 'n_ingredients', where the first two are discrete and the latter is continuous. I used 2 qualitative columns, ingredients and name. Ingredients and name are nominal by technicality. I used a CountVectorizer() independently on ingredients (after transforming it outside the function to be easier to parse) and name. I attempted to use a Binarizer(threshold = 29), where 29 is the high_sodium classifier from earlier, but this made the model worse, losing a significant amount of score. The score I received on this model is 0.25, which shows that my data is likely scattered and doesn't have enough features.

This model isn't great. It predictions seem too dependent on the correlations determined previously, which we see now as weak. As a result, I attempted the use of a DecisionTreeRegressor(), but the score for that model at depth = 15 was roughly 0.18, which is not an improvement for the extra computing time needed for the method. The score implies the model is overly simplistic and recommends either I use more features, polynomials, or classification. As a result, I'll use more features.

# Final Model

I judged which features to use in this model initially by using the spearmanr function and then using the pearsonr function to find correlations. Spearman uses ranking while Pearson uses linearity. Using iteration, I found which features are most similar to my data. As a result, I'm using the five features mentioned in Baseline Model, along with: 
1. minutes (quantitative, discrete), since steps take time
2. calories('#') (quantitative, discrete) since the more food you're making, you take more time
3. total fat (quantiative, discrete) since it's related to saturated fats and all other fats, avoiding collinearity and adding a significant features
4. carbohydrates (quantitative, discrete) since it's a minor correlated feature, makes the model better at getting closer values
5. protein (quantitative, discrete) since meat usually takes more steps to cook than vegetables

I applied the log function transformer to total fat and saturated fat because there was a marginal benefit to using np.log. It's a minor chance that likely occurs due to smoothening out the fat curves, making the model slightly better at capturing noise. I also applied the StandardScaler() to n_ingredients() as it can improve the interpretability of the model. I used a QuantileTransformer() on carbohydrates to reduce the impact of outliers. 

Most of the advantage in score came from the use of adding more features. Despite the correlations that the transformed columns had with its pearsonr, I found that the transformations hampered interpretability and reduced the score that had accrued. The resulting score from this refined model was 0.394.

After the use of gridsearchCV with hyperparameters linearregression__fit_intercept (to add an intercept or not) and 'columntransformer__scaler__copy': [True, False], which says to add a copy of the original data back into X. This improved my score to 0.47. 


