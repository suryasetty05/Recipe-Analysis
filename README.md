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

