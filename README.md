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
  src="assets/univariate-fig.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

### Bivariate Analysis
For this analysis, we examined if there was a significance difference in distribution of ratings for low-fat and high-fat recipes. As shown in the graph below, the overall trend is that most recipes are rated on the higher end. For the ratings of 1-4, recipes were more likely to be low-fat, while in the rating of 5, recipes were more likely to be high-fat. In our later tests, we will see if this difference is significant.

<iframe
  src="assets/bivariate-fig.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

### Interesting Aggregates
For this section, we investigated the relationship between the total calories of a recipe and its proportion of fat. Since there are some outliers in `'calories (#)'`, we created separate dataframe `filtered3_agg` to store the observations without outliers. We used the IQR method to identify these outliers. We then created six calories bins to compare the aggregates for proportion of fat between each. 

| cal_bin         |   ('mean', 'prop_fat') |   ('median', 'prop_fat') |   ('min', 'prop_fat') |   ('max', 'prop_fat') |   ('std', 'prop_fat') |
|:----------------|-----------------------:|-------------------------:|----------------------:|----------------------:|----------------------:|
| (-0.001, 125.0] |               0.370237 |                 0.36     |                     0 |               1.21483 |              0.327676 |
| (125.0, 201.8]  |               0.438575 |                 0.428286 |                     0 |               1.21575 |              0.282432 |
| (201.8, 284.7]  |               0.470528 |                 0.456056 |                     0 |               1.21935 |              0.254268 |
| (284.7, 385.1]  |               0.495173 |                 0.495197 |                     0 |               1.21575 |              0.23282  |
| (385.1, 536.7]  |               0.525576 |                 0.534382 |                     0 |               1.21923 |              0.217301 |
| (536.7, 971.7]  |               0.567884 |                 0.578701 |                     0 |               1.2477  |              0.220006 |

The results are as expected, with proportion of fat increasing as the calories increase. Since each gram of fat equates to 9 calories, it is reasonable that recipes with high amounts of calories may be inflated so by the amount of fat in the recipe. 


---

## Assessment of Missingness

To assess missingness, we sorted and viewed the value counts of missing values in the columns of `recipe_ratings`. From this, we deduced that `'rating'`, `'description'`, and `'review'` have significant amounts of missing values. Thus, we will proceed to assess the missingness in the dataframe.

### NMAR Analysis
I believe that the column that is most likely to be NMAR is the 'rating' column. People are less likely to leave a rating if the score they would have given is low, which means that missingness depends on the value itself. Additional data that could potentially change this missingness to MAR would be an analysis of the users' rating patterns--such as if they typically leave a rating or just a review--or possibilities of technical issues that erased the ratings. Such factors could explain the missingness of 'rating' values.

### Missingness Dependency
To decide if our missingness is dependent on other columns in the dataset, we performed some permutation tests to investigate the dependency of the missingness of `'rating'` on a few other columns. 

#### Permutation Test #1: total fat (PDV)
**Null Hypothesis:** The missingness of rating does not depend on the total fat in the recipe.  
**Alternate Hypothesis:** The missingness of rating does depend on the total fat in the recipe.  
**Test Statistic:** The absolute difference in means of the total fat in recipes with and without missing ratings.  
**Significance Level:** 0.05

<iframe
  src="assets/missing-perm-fig1.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

For our first permutation test, the **observed statistic** is **5.7406** and is denoted by the red line on the graph above. Since the **p-value** is **0.0**, we **reject the null hypothesis** on a confidence level of 0.05. The missingness of 'rating' does depend on the total fat (PDV) of the recipe.

#### Permutation Test #2: calories (#)
**Null Hypothesis:** The missingness of rating does not depend on the amount of calories in the recipe.  
**Alternate Hypothesis:** The missingness of rating does depend on the amount of calories in the recipe.  
**Test Statistic:** The absolute difference in means of the calories in recipes with and without missing ratings.
**Significance Level:** 0.05

<iframe
  src="assets/missing-perm-fig2.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

For our second permutation test, we found the **observed statistic** to be **51.4234** and is denoted by the red line on the graph above. Since the **p-value** was calculated to be **0.131**, we **fail to reject the null hypothesis** on a confidence level of 0.05. Thus, we cannot conclude that the missingness of the 'rating' column depends on the time (in minutes) to prepare the recipe. 


---

## Hypothesis Testing

As mentioned previously, in our project we intend to analyze whether people rate high-fat and low-fat recipes on the same scale. To investigate this, we ran a **permutation test** on the 0.05 confidence level with the following hypotheses.

**Null Hypothesis:** People rate high-fat and low-fat recipes on the same scale.  
**Alternative Hypothesis:** People rate high-fat recipes higher than low-fat recipes.  
**Test Statistic:** Difference in mean between high-fat and low-fat recipes.  
**Significance Level:** 0.05

We chose to run a permutation because we wanted to see if the two distributions look like they come from the same population. We suggest that **people rate high-fat recipes higher** because many people rate recipes based on the taste of the final product. In general, high-fat recipes often taste better than low-fat recipes due to high amounts of sugar and fats that elevate the taste of the dish. Since direction matters in our test, we chose to use the difference in mean rather than absolute difference so that we may see which group of recipes have a higher rating. We chose to use a significance level of 0.05, as it is the common benchmark for hypothesis tests.

<iframe
  src="assets/hypotest_fig.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

### Conclusion
After performing our permutation test with 1000 permutations, we compared the results to our **observed statistic** of **0.0328**. Our resulting **p-value** was **0.0**, which, when compared to our significance level of 0.05, means that we **reject the null hypothesis**. In conclusion, people do not rate high-fat and low-fat recipes on the same scale, and tend to rate high-fat recipes higher. A possible explanation for this is, as we mentioned earlier, that high-fat recipes could taste better than low-fat recipes for the general reviewers.


---

## Framing a Prediction Problem
For this project, we plan on **predicting the average rating of a recipe** - a **regression problem**. The response variable for our model is average rating of a recipe, which we chose because we believe it is representative of user satisfaction and the overall quality of the recipe. Furthermore,  we have previously found that people do not rate high-fat and low-fat recipes on the same scale, so we can account for that in our prediction mdoel.

The metric we will be using to evaluate our model is **mean absolute error**, which we chose over accuracy, as it is easier to interpret for how many points on average we are off by in our prediction model. Furthermore, unlike mean squared error, MAE does not disproportionally penalize large but rare errors. 

Since the ratings are created after the commenters have tried the recipes, we have access to all the columns from the `'recipes'` dataset for our prediction. The features included in the dataset have been listed and described in the first section.

---

## Baseline Model
For our baseline model, we utilized a random forest regressor and split the data points into training and test sets using sklearn's `train_test_split` method. As there are a few outliers and missing values in the columns we are using, we used the IQR method to exclude outliers from the `'minutes'` column and named our filtered dataframe `filtered_agg`. Since `avg_rating` has missing values that we cannot impute, as it would introduce more noise and skew our predictions, we decided to drop the missing values as it is a small percentage of the observations. 

Thus, the features we are using for this model is `'minutes'`, a quantitative column, and `'fat_category'`, a nominal column which takes on the values of `'high-fat'` or `'low-fat'`. Since the range is quite large, but is somewhat skewed, we encoded the `'minutes'` with `StandardScalar` for more meaningful coefficients. For `'fat_category'`, we performed a one hot encoding and dropped one column, resulting in a `'high-fat'` column which takes value 1 for a recipe being `'high-fat'` and 0 otherwise. These steps allowed us to train the model on the data from our split

Our **MAE** metric for this model evaluated to be **0.336**. This means that our predictions were roughly 0.336 points off from the true value, which, considering that ratings range from 1 to 5, is "good". This error means that our predictions were about 8-9% off from the total scale, which is quite small and means that predictions were consistently close to the true ratings.


---

## Final Model

---

## Fairness Analysis