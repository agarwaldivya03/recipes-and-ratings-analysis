Rate My Plate - Predicting Recipe Ratings
==============
This project, conducted at the University of Michigan, delves into the factors that influence the average ratings of recipes on food.com. The analysis spans various stages, beginning with exploratory data analysis and progressing through the development of predictive models. By examining key recipe attributes, this project aims to uncover patterns and insights that contribute to the overall perception and popularity of recipes on the platform.

Author: Divya Agarwal

## Introduction

The one question this project ultimately aims to answer is 'What Decides the average ratings of recipes?'. Online recipe platforms like food.com play a significant role in how people discover and try new recipes. Thus, understanding what drives average recipe ratings can provide valuable insights for both users and recipe creators. For users, it can help identify what features to look for in highly-rated recipes, such as ingredient variety, preparation time, or user engagement through reviews. For recipe creators, it sheds light on what factors might influence the success of their submissions, offering guidance on how to craft recipes that resonate with the community. Moreover, the dataset itself is a rich resource that reflects real-world user preferences and behavior, making it not only relevant for cooking enthusiasts but also a fascinating case study for those interested in data science, user interaction, and online content evaluation.


To investigate this, we use a dataset of recipe info scraped from food.com by the authors of [this paper](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf). The data itself is divided into two .csv files - `RAW_recipes.csv`, which contains the recipes; and `RAW_interactions.csv` which contains reviews and ratings submitted for the recipes in `RAW_recipes.csv`. The original `RAW_recipes.csv` file contains **83782** rows, while `RAW_interactions.csv` contains **731927** rows. Once the two dataframes are merged together based on the `recipe_id` column, the resulting dataframe has **234429** rows. Although the merged dataset has 18 columns, we will only focus on a select few in the merged dataset for our analysis, which are listed below:


| Column            | Description                                                                                                 | 
|-------------------|-------------------------------------------------------------------------------------------------------------|
| `'recipe_id'`     | Recipe ID - uniquely identifies each recipe in the dataset                                                  |
| `'name'`          | Name of the recipe                                                                                          | 
| `'rating'`        | The rating given to each recipe by a single user - this will be used to compute `average_rating` per recipe |
| `'minutes'`       | Minutes to prepare each recipe                                                                              |
| `'n_steps'`       | Number of steps in recipe                                                                                   |
| `'n_ingredients'` | Number of ingredients in recipe                                                                             |

<br>

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning
Before creating an analysis, we must first ensure the data is cleaned and in a usable format. Thus, the following steps are taken to clean the data:
1. The two dataframes resulting from `RAW_recipes.csv` and  `RAW_interactions.csv` are merged on `recipe_id`. In particular, a left merge is performed so that all the recipes in `RAW_recipes.csv` are kept, even if there are no corresponding ratings for them.  

2. All the 0's in the `ratings` column are replaced with `np.nan`. Filling ratings of 0 with `np.nan` is done because 0 often represents missing or invalid data, and not a genuine evaluation. This ensures such entries don't skew statistics like mean or median by treating 0 as true missing data. 

3. The average rating of each recipe is calculated as a series by grouping on `recipe_id`. In addition, we also calculate the number of ratings per recipe. This series is then merged with our dataframe and added as a `average_rating` and `num_ratings` column respectively.

4. The rows of the dataframe are filtered to only include recipes that take at most 1 day (<= 1440 minutes) to make. This ensures the analysis focuses on practical and relevant recipes for typical users. This removes outliers or anomalies with unrealistic preparation times, improving the accuracy and representativeness of insights derived from the dataset.

5. Finally, we drop the uneccessary rows and columns from the dataset. Specifically, `'contributor_id'` , `'submitted'` , `'tags'` , `'nutrition'` , `'steps'` , `'description'` , `'user_id'` , `'date'` , `'rating'` , `'review'` , `'ingredients'` , and any duplicate columns that resulted from merging are dropped. Since the dataframe initially had one row per rating per recipe, it was condensed to one row per recipe, incorporating the calculated average rating. This condensed format is used for subsequent analysis.

The resulting dataframe, is the final, cleaned version of our data that we will now use with **83194** rows. The first few rows of are shown below:

|   recipe_id | name                                 |   average_rating |   minutes |   n_steps |   n_ingredients |   num_ratings |
|------------:|:-------------------------------------|-----------------:|----------:|----------:|----------------:|--------------:|
|      333281 | 1 brownies in the world    best ever |                4 |        40 |        10 |               9 |             1 |
|      453467 | 1 in canada chocolate chip cookies   |                5 |        45 |        12 |              11 |             1 |
|      306168 | 412 broccoli casserole               |                5 |        40 |         6 |               9 |             4 |
|      286009 | millionaire pound cake               |                5 |       120 |         7 |               7 |             1 |
|      475785 | 2000 meatloaf                        |                5 |        90 |        17 |              13 |             2 |

<br>

### Exploratory Data Analysis - Univariate Analysis

Next, we examine the distribution of various variables to gain insights into the behavior of the data.

We begin by analyzing the cooking time (in minutes).
<br>
<iframe
  src="assets/cooking-times-dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The graph suggests that the data is skewed to the left, with majority of the recipes taking less than 50 minutes to make. This implies that shorter preparation times are the norm on Food.com, which could mean that users tend to favor or contribute quick and easy recipes.

Next, we look at the distrubution of number of steps.
<br>
<iframe
  src="assets/steps-dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Once again, the graph is skewed towards lesser number of steps, with the median being 9 - suggesting that less complex recipes might be more frequently contributed to on the website. 


### Exploratory Data Analysis - Bivariate Analysis

We then perform bivariate analyses to see how variables correlate with each other. Two such analyses: Number of Steps vs Average Rating, and Cooking Time vs Average Rating, are shown below. We expect to see a strong negative correlation between the complexity/length of the recipe and the average rating, following the reasoning that easier recipes seem to be favored on the website. 

<br>
<iframe
  src="assets/steps-rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<br>
<iframe
  src="assets/mins-rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

As expected, higher-rated recipes tend to have fewer steps and shorter cooking times, reflecting user preference for simplicity and convenience. Such recipes are more likely approachable, appealing to busy individuals and novice cooks, which boosts satisfaction and ratings. For recipe creators, simplifying instructions seems to enhance a recipe's popularity and success.

<br>

### Interesting Aggregates

To further understand the data, We compute some interesting aggregates. The data is grouped into bins of 2 hours and the average ratings, average n_ingredients, and average n_steps are computed for each bin. The results are shown below:


| time_bin     |   average_rating |   n_ingredients |   n_steps |
|:-------------|-----------------:|----------------:|----------:|
| [0, 120)     |          4.62907 |         9.07412 |   9.77271 |
| [120, 240)   |          4.63321 |        10.6     |  13.503   |
| [240, 360)   |          4.56853 |        10.3019  |  11.7765  |
| [360, 480)   |          4.4933  |         9.94611 |   9.48303 |
| [480, 600)   |          4.51417 |         9.89062 |   9.32452 |
| [600, 720)   |          4.54231 |        10.35    |  10.85    |
| [720, 840)   |          4.63908 |         9.38554 |  12.6807  |
| [840, 960)   |          4.75805 |        10.4412  |  20.2353  |
| [960, 1080)  |          4.62556 |         8.53333 |  15.7333  |
| [1080, 1200) |          4.52941 |         9.17647 |  19.2941  |
| [1200, 1320) |          4.69039 |         9.375   |  17.9583  |
| [1320, 1440) |          4       |         5       |  31       |


Although the data mostly behaves in accordance with our analysis following from looking at the bivariate distributions, there are some anomalies where higher cooking times seem to have high ratings. To better understand this, we also add a column for how many recipes are in each bin.


<br>

| time_bin     |   average_rating |   n_ingredients |   n_steps |   num_ratings_in_bin |
|:-------------|-----------------:|----------------:|----------:|---------------------:|
| [0, 120)     |          4.62907 |         9.07412 |   9.77271 |                72317 |
| [120, 240)   |          4.63321 |        10.6     |  13.503   |                 4513 |
| [240, 360)   |          4.56853 |        10.3019  |  11.7765  |                 1517 |
| [360, 480)   |          4.4933  |         9.94611 |   9.48303 |                 1002 |
| [480, 600)   |          4.51417 |         9.89062 |   9.32452 |                  832 |
| [600, 720)   |          4.54231 |        10.35    |  10.85    |                  160 |
| [720, 840)   |          4.63908 |         9.38554 |  12.6807  |                  166 |
| [840, 960)   |          4.75805 |        10.4412  |  20.2353  |                   34 |
| [960, 1080)  |          4.62556 |         8.53333 |  15.7333  |                   15 |
| [1080, 1200) |          4.52941 |         9.17647 |  19.2941  |                   17 |
| [1200, 1320) |          4.69039 |         9.375   |  17.9583  |                   24 |
| [1320, 1440) |          4       |         5       |  31       |                    1 |


The data reveals a strong preference for quick recipes, with the [0, 120) time bin dominating in popularity at over 72,000 ratings and maintaining a high average rating of 4.63. As cooking times increase, the number of ratings drops significantly, indicating that longer recipes are not well favored. However, outliers like [1200, 1320) may not be representative due to limited data (only 24 ratings in this case). This may suggest an overly niche appeal of the recipes which would explain why the recipes with such high cooking times still have higher ratings. 

<br>

### Imputation - Missing Values

Investigating data missingness is crucial to determine whether missing values need to be addressed and how they might impact the analysis. Using the `.isna()` function in Python, we identified that the only columns with missing values are `'name'` and `'average_rating'`.

Since recipes with missing `'name'` values contain information about the number of steps, ingredients, and cooking times at best—which are the variables already used in our analysis—we choose to ignore the missing values in the `'name'` column.

Additionally, only **3.08%** of the `'average_rating'` values are missing. A plot of Cooking Time vs. Average Rating, excluding these missing values, showed no indication of significant bias introduced by their absence. Therefore, we deemed it safe to drop rows with missing `'average_rating'` values from the dataset.

<br>
<iframe
  src="assets/side-by-side.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


<br>

## Framing A Prediction Problem

Our analysis revealed that cooking time and the number of steps significantly affect a recipe's average rating. Given the wide variation in recipe complexity and preparation details, are there certain factors that consistently determine a recipe's popularity? Specifically, can we predict a recipe's average rating based solely on its characteristics, such as cooking time, number of ingredients, number of steps, and other relevant features?

The ultimate goal of this project is thus to predict the average ratings of recipes, which makes it a regression problem because the target variable (average ratings) is a continuous variable. The response variable is `'average rating'`, chosen because it reflects how much users like a recipe. To evaluate the model, we use Mean Squared Error (MSE) because it gives more weight to bigger errors, helping identify large mistakes, and R², which shows how much of the variation in ratings the model can explain. These metrics together help assess how accurate and useful the model is at predicting recipe ratings. Both are standard metrics in regression tasks, making it easier to compare our results against other models or benchmarks. They are well-documented and commonly used, which makes these indicators a good fit for our model. 

<br>

## Baseline Model

The baseline model uses linear regression to predict a recipe’s average rating based on several features. The features include `cooking time (minutes)`, `number of steps`, `number of ingredients`, and `number of ratings`, all of which are quantitative. Additionally, two engineered features: `minutes per step` and `minutes times steps` were added to the model to capture more nuanced relationships between cooking time and steps. A custom numerical pipeline handles these transformations, while the data preprocessing and model training are combined in a unified pipeline.

 The MSE is 0.411, and the R² score is 0.005. This suggests that the model's performance can be improved, as it struggles to capture meaningful patterns in the data. The likely causes could be a lack of strong linear relationships between the features and the target variable or the need for a more complex model to capture non-linear relationships. While the current model provides a baseline, further exploration with more advanced techniques or additional features is necessary to improve predictive performance

<br>

## Final Model

The new model incorporates two key engineered features: `log(minutes) * log(n_steps)` and `log-transformed num_ratings`, in addition to log transforming all other numerical columns resulting in - `'log(minutes)'`, `'log(n_steps)'`, and `'log(n_ingredients)'`.

1. `log(minutes) * log(n_steps)`: This interaction term is used to better define recipe complexity, while capturing their non-linear relationships. Recipes with a balanced interaction between time and steps might perform better, and this feature helps model such intricate patterns in the data.

2. `log-transformed numerical columns`: These transformations reduce the skewness in the numerical columns and ensure that extremely high or low values don’t disproportionately affect the model. Log transformations aligns with observations from the bivariate analysis, which revealed a non-linear correlation between  `minutes` and `n_steps` with `average_ratings`. The inclusion of `'num_ratings'` is important here as user ratings represent engagement and trustworthiness.

The chosen modeling algorithm is a Random Forest Regressor, a robust ensemble-based approach capable of capturing complex non-linear relationships - which we suspect might be better suited for our data. A grid search was performed over the hyperparameters `n_estimators (number of trees)` and `max_depth (maximum tree depth)`, and the best parameters were found to be:

**n_estimators**: 200

**max_depth**: 5

The hyperparameters were selected using 5-fold cross-validation with negative mean squared error as the scoring metric, ensuring the model generalizes well to unseen data.

The final model’s performance represents an improvement over the baseline:

**Mean Squared Error (MSE)**:

Baseline: 0.41

Final Model: 0.40

**R-squared (R²)**:

Baseline: 0.005

Final Model: 0.009

While the improvement in MSE and R² is modest, the Random Forest model with engineered features captures more variance in the target variable and handles non-linear interactions better than the linear regression baseline. These results suggest that the final model is more aligned with the complexities of the data.














