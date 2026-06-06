# Recipe Rating Analysis

by Tianyu Yang (tiy022@ucsd.edu)

---

## Introduction

This project uses the Recipes and Ratings dataset from food.com, which contains recipe information and user-submitted ratings. The dataset includes information such as preparation time, nutritional content, ingredients, cooking instructions, and user ratings. The data originally comes from two files: `RAW_recipes.csv`, which contains recipe-level information, and `interactions.csv`, which contains user ratings and reviews. These datasets were merged using recipe IDs, and average ratings were computed for each recipe.

The cleaned dataset contains **83,782 recipes**.

This project investigates the following question:

**What types of recipes tend to have higher average ratings?**

In particular, the analysis explores whether factors such as preparation time, nutritional content, ingredient count, and recipe complexity are associated with higher average ratings. Understanding which recipe characteristics are associated with highly rated recipes may help users identify recipes they are more likely to enjoy.

The most relevant columns used in this analysis are summarized below:

| Column | Description |
|----------|----------|
| `minutes` | Preparation time of the recipe in minutes |
| `nutrition` | Nutritional information, including calories, fat, sugar, sodium, protein, saturated fat, and carbohydrates |
| `n_steps` | Number of cooking steps in the recipe |
| `n_ingredients` | Number of ingredients in the recipe |
| `description` | User-provided description of the recipe |
| `avg_rating` | Average user rating computed from all ratings associated with the recipe |

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The original dataset consisted of two files: `RAW_recipes.csv`, which contains recipe-level information, and `interactions.csv`, which contains user ratings and reviews. These datasets were merged using recipe IDs so that each recipe could be linked to its corresponding ratings.

Several cleaning steps were performed before analysis. First, ratings of 0 were replaced with missing values (`NaN`) because a rating of 0 indicates that no rating was provided rather than an actual zero-star rating. Average ratings were then computed for each recipe and added as a new column, `avg_rating`.

Additionally, the `submitted` column was converted to datetime format. The `nutrition`, `tags`, `steps`, and `ingredients` columns were stored as strings representing lists, so they were converted into Python lists. The `nutrition` column was further expanded into seven separate numerical columns: `calories`, `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, and `carbs`.

The first five rows of the cleaned DataFrame are shown below.

|    | name                                 |     id |   minutes |   contributor_id | ...   |   sodium |   protein |   saturated_fat |   carbs |
|---:|:-------------------------------------|-------:|----------:|-----------------:|:------|---------:|----------:|----------------:|--------:|
|  0 | 1 brownies in the world    best ever | 333281 |        40 |           985201 | ...   |        3 |         3 |              19 |       6 |
|  1 | 1 in canada chocolate chip cookies   | 453467 |        45 |          1848091 | ...   |       22 |        13 |              51 |      26 |
|  2 | 412 broccoli casserole               | 306168 |        40 |            50969 | ...   |       32 |        22 |              36 |       3 |
|  3 | millionaire pound cake               | 286009 |       120 |           461724 | ...   |       13 |        20 |             123 |      39 |
|  4 | 2000 meatloaf                        | 475785 |        90 |          2202916 | ...   |       12 |        29 |              48 |       2 |

### Univariate Analysis
#### Distribution of Average Recipe Ratings

<iframe src="assets/avg_rating_distribution.html" width="800" height="200" frameBorder="0"></iframe>

The distribution of average recipe ratings is concentrated near the upper end of the rating scale. Most recipes have average ratings between 4 and 5, while relatively few recipes receive low ratings.

This suggests that users on Food.com tend to rate recipes positively, resulting in a distribution that is heavily skewed toward higher ratings.

### Bivariate Analysis
#### Preparation Time vs Average Rating

<iframe src="assets/preparation_time_vs_rating.html" width="800" height="200" frameBorder="0"></iframe>

This scatter plot compares recipe preparation time and average rating. Most recipes are concentrated at shorter preparation times and higher ratings, especially between 4 and 5 stars. There does not appear to be a strong linear relationship between preparation time and average rating, but the plot suggests that highly-rated recipes exist across a wide range of preparation times.

### Interesting Aggregates

To further examine recipe complexity, recipes were grouped based on the number of cooking steps. Recipes with 5 or fewer steps were labeled simple, recipes with 6 to 10 steps were labeled moderate, recipes with 11 to 20 steps were labeled complex, and recipes with more than 20 steps were labeled very complex.

| complexity   |   count |    mean |   median |
|:-------------|--------:|--------:|---------:|
| simple       |   18547 | 4.63786 |        5 |
| moderate     |   31915 | 4.61625 |        5 |
| complex      |   25767 | 4.62345 |        5 |
| very complex |    4944 | 4.6473  |        5 |

Across the four complexity groups, the mean average ratings are very similar, ranging from about 4.62 to 4.65. This suggests that recipe complexity, as measured by the number of steps, does not appear to have a strong relationship with average rating in this dataset.

---

## Assessment of Missingness

### NMAR Analysis

One column that may be NMAR is **`description`**. A missing description may depend on the unobserved value of the description itself. For example, recipes with very short, unclear, or low-effort descriptions may be more likely to have missing descriptions. Since the reason for missingness may depend on the missing description content itself, this missingness could be NMAR.

Additional data, such as whether the user intentionally skipped the description field or how much time the user spent filling out the recipe form, would help determine whether the missingness is actually NMAR.

### Missingness Dependency

To investigate whether the missingness of the `description` column depends on other variables, two permutation tests were performed. A significance level of 0.05 was used.

The first test examined the relationship between description missingness and `n_steps`. The test statistic was the absolute difference in mean `n_steps` between recipes with missing descriptions and recipes with non-missing descriptions. The resulting p-value was **0.186**.

#### Description Missingness vs Number of Steps

<iframe src="assets/description_missingness_test1.html" width="800" height="400" frameBorder="0"></iframe>

The histogram shows the permutation distribution of the test statistic, while the red line represents the observed statistic. Since the p-value is greater than 0.05, there is insufficient evidence to conclude that the missingness of `description` depends on `n_steps`.

The second test examined the relationship between description missingness and `n_ingredients`. The test statistic was the absolute difference in mean `n_ingredients` between recipes with missing descriptions and recipes with non-missing descriptions. The resulting p-value was **0.004**.

#### Description Missingness vs Number of Ingredients

<iframe src="assets/description_missingness_test2.html" width="800" height="500" frameBorder="0"></iframe>

The histogram shows the permutation distribution of the test statistic, while the red line represents the observed statistic. Since the p-value is less than 0.05, there is evidence that the missingness of `description` is associated with `n_ingredients`.

---

## Hypothesis Testing

This hypothesis test examines whether recipes with more ingredients and recipes with fewer ingredients have different average rating distributions. Recipes were divided into two groups based on the median number of ingredients. Recipes with more than the median number of ingredients were labeled as having many ingredients, while recipes with less than or equal to the median were labeled as having few ingredients.

The null hypothesis is that recipes with more ingredients and recipes with fewer ingredients have the same average rating distribution. The alternative hypothesis is that the two groups have different average rating distributions.

The test statistic is the absolute difference in mean average rating between the two groups. A significance level of 0.05 was used.

This test statistic is appropriate because the research question compares the ratings of recipes with more ingredients and recipes with fewer ingredients. Since the goal is to determine whether the two groups have different average ratings, the absolute difference in mean average rating directly measures the size of the difference between the two groups. A permutation test is appropriate because it tests whether the observed difference could have occurred by random chance if ingredient group and average rating were unrelated.

<iframe src="assets/rating_difference_test.html" width="800" height="600" frameBorder="0"></iframe>

The histogram shows the permutation distribution of the test statistic, and the red line represents the observed statistic. The resulting p-value was **0.392**.

Since the p-value is greater than 0.05, there is insufficient evidence to conclude that recipes with more ingredients and recipes with fewer ingredients have different average rating distributions.

---

## Framing a Prediction Problem

The prediction problem is to predict the average rating of a recipe. This is a **regression** problem because the response variable, `avg_rating`, is numerical.

The response variable is `avg_rating` because this project focuses on what types of recipes receive higher ratings. The model is evaluated using **RMSE**, since RMSE measures prediction error for numerical outcomes and penalizes larger errors more heavily.

At the time of prediction, only recipe-level information such as preparation time, number of ingredients, number of steps, and nutrition information would be available.

---

## Baseline Model

The baseline model predicts the average rating of a recipe using two features: `minutes` and `n_ingredients`. Both features are quantitative numerical features; there are no ordinal or nominal features in this baseline model. Since both features are already numerical, no encoding is needed.

A linear regression model is used as the baseline model. The data is split into training and test sets, and the model is trained on the training set and evaluated on the test set using RMSE.

The baseline model achieved an RMSE of **0.6428** on the test set. The baseline model is limited because it only uses two basic recipe features. Adding more informative features and feature transformations may improve performance in the final model.

---

## Final Model

The final model uses more features than the baseline model. In addition to `minutes` and `n_ingredients`, it includes `n_steps`, `calories`, `protein`, `sugar`, and `carbs`.

Two engineered features were added. The first is `log_minutes`, which is a log-transformed version of preparation time. This feature is useful because preparation time has extreme outliers, and the log transformation reduces the influence of very large values. The second is `steps_per_ingredient`, which is calculated as `n_steps / n_ingredients`. This feature measures recipe complexity relative to the number of ingredients.

The final model uses Ridge regression with polynomial features. Polynomial features allow the model to capture nonlinear relationships between the recipe features and average rating. Ridge regression is used because polynomial features create additional terms, which can make the model more complex. Ridge regularization helps reduce overfitting by penalizing large coefficients.

Hyperparameters were selected using `GridSearchCV` with 5-fold cross-validation. The hyperparameters tuned were `poly__degree`, the degree of the polynomial features, and `ridge__alpha`, the alpha value used in Ridge regression. The best hyperparameters were **`poly__degree = 1`** and **`ridge__alpha = 1000`**.

The final model achieved an RMSE of **0.6422** on the test set. Compared to the baseline RMSE of **0.6428**, the final model performed slightly better. However, the improvement is small, suggesting that these additional features and transformations did not substantially improve prediction performance.

---

## Fairness Analysis

This fairness analysis examines whether the final model performs differently for recipes with longer preparation time and recipes with shorter preparation time. Recipes were divided into two groups based on the median preparation time. Recipes with preparation time greater than the median were labeled as longer-preparation-time recipes, while recipes with preparation time less than or equal to the median were labeled as shorter-preparation-time recipes.

The null hypothesis is that the model is fair: recipes with longer preparation time and recipes with shorter preparation time have the same RMSE. The alternative hypothesis is that the model is unfair: the two groups have different RMSE values.

The test statistic is the absolute difference in RMSE between the two groups. A significance level of 0.05 was used.

Since this is a regression model, RMSE is used as the evaluation metric. The final fitted model from the previous step was used to make predictions on the test set.

A permutation test was used to compare the RMSE values of the two groups. During each permutation, the preparation-time group labels were shuffled, while the actual ratings and predicted ratings stayed the same.

<iframe src="assets/fairness_analysis.html" width="800" height="600" frameBorder="0"></iframe>

The histogram shows the permutation distribution of the test statistic, and the red line represents the observed statistic. The resulting simulated p-value was **less than 0.001**.

Since the p-value is less than 0.05, the null hypothesis is rejected. This suggests that the final model's performance differs between recipes with longer preparation time and recipes with shorter preparation time.
