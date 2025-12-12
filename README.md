# Analyzing Recipe Ratings

by Salina Tang

---

## Introduction

As with many others, I am an avid food enjoyer and have recently been interested in learning how to cook and bake. However, as I become more conscious of my health, I start to look at the nutrition facts of different recipes and how various ingredients contribute to each statistic. People should be more aware of what they are consuming for the health of their bodies and well-being, especially with possible harmful consequences of excess fat or sodium. For this data science project, I intend on exploring two datasets from [food.com](https://www.food.com/): one containing recipes and their nutritional information, and the other containing user ratings for those recipes. **In particular, I hope to examine whether people rate high-fat and low-fat recipes on the same scale or rather high-fat recipes higher.** By finding an answer to this question, I aim to better understand user preferences in relation to nutritional content.

The first dataset `recipes` contains 83782 observations and 12 columns:

| Column             | Description |
|:-------------------|:------------|
| `'name'`           | Recipe name |
| `'id'`             | Recipe ID |
| `'minutes'`        | Minutes to prepare recipe |
| `'contributor_id'` | User ID who submitted this recipe |
| `'submitted'`      | Date recipe was submitted |
| `'tags'`           | Food.com tags for recipe |
| `'nutrition'`      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for "percentage of daily value" |
| `'n_steps'`        | Number of steps in recipe |
| `'steps'`          | Text for recipe steps, in order |
| `'description'`    | User-provided description |


The second dataset `interacts` contains 731927 observations and 5 columns:

| Column         | Description |
|:---------------|:------------|
| `'user_id'`    | User ID |
| `'recipe_id'`  | Recipe ID |
| `'date'`       | Date of interaction |
| `'rating'`     | Rating given |
| `'review'`     | Review text |

For our data analysis, we will separate the `'nutrition'` column to make use of the different nutritional information the recipe provides. In particular, we are interested in the `'calories (#)'` and `'total fat (PDV)'` values. Since we are interested in the trends of how people rate different recipes, we are also interested in the `'rating'` column.

Through researching these trends, we hope to bring awareness to our readers about the nutritional aspects of the food and recipes they may enjoy. Our goal is not to promote any specific dietary opinions or push particular movements, but instead to provide information to empower readers to make choices that align with their preferences and well-being. Food should be enjoyable, and for many people, cooking and baking is a fun hobby as a way to relax and experiment. We want to encourage a balanced relationship with food, so that our readers can feel informed what goes in their meals.

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning
For the purpose of analysis, we have conducted the following data cleaning steps:

1. Left merge the recipes and interaction datasets together
    - We performed a left merge with `recipes` as the left dataset and `interacts` as the right dataset, using `id` and `recipe_id` as the join key from each dataset, respectively. We named this merged dataset `recipe_ratings`.

2. Fill all ratings of 0 with `np.nan`
    - We decided to replace ratings of 0 with `NaN`, as in many cases, viewers can leave a comment without assigning a rating. These comments would automatically be stored as a rating of 0. However, including these as real ratings would wrongfully drag down the average, so by replacing them with `NaN`, we ensure that the computed average rating reflects real user ratings.

3. Compute the average rating per recipe
    - In order to have one comprehensive rating per recipe, as the observations from `interacts` may have multiple reviews per recipe, we computed an average rating by grouping the reviews by `'recipe_id'` and taking the mean. We named this column `'avg_rating'`.

4. Breaking down the `'nutrition'` column
    - As we know from the column description, `'nutrition'` contains the nutritional facts of the recipe. The column's values are of type `str`, but resembles the structure of a list. To split up the values, we applied string functions to the column with the `.str` accessor and separated the values into seven separate columns. For numerical calculations later on in this project, we converted the values to floats.

5. Adding other useful variables for our analysis to the dataset
    - `'prop_fat'` is the proportion of calories in a recipe that come from fat. To compute this, we first computed the amount of calories associated with fat in a recipe. We took the `'total fat (PDV)'` and divided by 100 to change the value to a percentage, and then multiplied by 78 grams, which is the FDA daily value (DV) for total fat for adults. Since a gram of fat contains 9 calories, we multiplied the fat grams by 9 to obtain the calories from fat. Finally, we divided by `'calories (#)'` to compute the proportion of the recipe's fat calories to total calories.
    - `'fat_category'` separates each observation into two values: `'high-fat'` and `'low-fat`'. To decide which value attributes to each recipe, we computed the threshold by taking the mean of `'prop_fat'`. Recipes with a value greater than or equal to the threshold were categorized as `'high-fat'`, and recipes with a value less than the threshold were categorized as `'low-fat'`. 

#### Result
Our cleaned dataframe resulted in a total of 234429 observations and 26 columns. We have attached the first 5 rows of our `'recipe_ratings'` dataframe to give an idea of the values each column contains. Since there are many columns, we have selected from them the columns that may be relevant to our analysis.

| name                                 |   recipe_id |   minutes |   n_steps |   rating |   avg_rating |   calories (#) |   total fat (PDV) |   prop_fat | fat_category   |
|:-------------------------------------|------------:|----------:|----------:|---------:|-------------:|---------------:|------------------:|-----------:|:---------------|
| 1 brownies in the world    best ever |      333281 |        40 |        10 |        4 |            4 |          138.4 |                10 |   0.507225 | high-fat       |
| 1 in canada chocolate chip cookies   |      453467 |        45 |        12 |        5 |            5 |          595.1 |                46 |   0.542631 | high-fat       |
| 412 broccoli casserole               |      306168 |        40 |         6 |        5 |            5 |          194.8 |                20 |   0.720739 | high-fat       |
| 412 broccoli casserole               |      306168 |        40 |         6 |        5 |            5 |          194.8 |                20 |   0.720739 | high-fat       |
| 412 broccoli casserole               |      306168 |        40 |         6 |        5 |            5 |          194.8 |                20 |   0.720739 | high-fat       |


### Univarate Analysis

For this analysis, we decided to view the distribution of fat density, or the column `'prop_fat'`. As shown in the plot below, the distribution is concentrated between 0.4 and 0.6, indicating that the proportion of fat for the recipes on food.com are clustered within this range. The bump in the first bin suggests that there may be some recipes that are listed as zero fat.

<iframe
  src="recipe-ratings-analysis/assets/univariate-fig.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


### Bivariate Analysis

For this analysis, we examined if there was a significance difference in distribution of ratings for low-fat and high-fat recipes. As shown in the graph below, the overall trend is that most recipes are rated on the higher end. For the ratings of 1-4, recipes were more likely to be low-fat, while in the rating of 5, recipes were more likely to be high-fat. In our later tests, we will see if this difference is significant.

<iframe
  src="recipe-ratings-analysis/assets/bivariate-fig.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

## Assessment of Missingness

---

## Hypothesis Testing

---

## Framing a Prediction Problem

---

## Baseline Model

---

## Final Model

---

## Fairness Analysis