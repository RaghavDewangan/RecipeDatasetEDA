# Exploring the Relationships Between The Calories in a Recipe vs. the Average Rating


Authors: Raghav Dewangan and Supriyaa Chordia


Exploratory Data Analysis, Hypothesis Testing, Predictive Modelling on recipes dataset


## Introduction

Exploring the relationship between nutrients and ratings in food recipes offers invaluable insights for both health-conscious individuals and culinary enthusiasts. Understanding the impact of nutritional features like calories can help one become more cognizant of their cooking and eating patterns to make informed decisions about their health. Calories are one common nutritional aspect that health enthusiasts track to ensure that one's energy intake needs are being fulfilled and are often correlated with weight changes, as they are a simple indicator of a culmination of nutritional factors, including fats, sugar, and protein. According to an article by MedicalNewsToday, consuming the right amount of calories is essential based on your activity level, age, sex, body shape, weight, height, and metabolism. The calorific values between components of food also differ—on average, 1 g of carbohydrates contains 4 kcal, 1 g of protein contains 4 kcal, and 1 g of fat contains 9 kcal. While the nutritional value of food is dependent on a variety of nutrients, our analysis will specifically consider calorie count, as it is a common nutrition-tracking method. We aimed to explore whether the calorie count of the food makes the recipe more or less appealing to people based on its average rating. To support our exploration, we used a subset of data from [food.com](https://www.food.com/), which was originally scraped from this [research paper](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) that was used to generate personalized recipes based on food preferences. We used two datasets entitled 'recipes' and 'interactions'.

Below is the 'recipes' dataframe. There are 10 columns and 83782 rows. Each row corresponds to a unique recipe (recipe_id or name for reference) with its corresponding nutritional and preparational stats.

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

This dataframe works in tandem with the 'interactions' dataset, which has 5 columns and 731927 rows. Each row corresponds to a review left on a certain recipe. Any given recipe can have multiple corresponding reviews.

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |


Our exploration was centered around the idea of calories and ratings. **How does calorie count affect a recipe's rating, and what is the best way to estimate calories based on a recipe’s rating?** We separated the 'nutrition' column values into nutrition-specific columns named 'calories', 'total_fat_PDV', 'sugar_PDV', 'sodium_PDV', 'protein_PDV', 'saturated_fat_PDV', and 'carbohydrates_PDV'. In nutrition labels, "PDV" stands for Percentage Daily Value, and it quantifies the extent of a nutrition's contribution to a total daily diet. We also added the columns named 'calorie_category', which classifies recipes into 3 categories: 'high', 'medium', and 'low' based on their calorific content, and ‘avg_rating’, which displayed the average rating for each recipe. The most important columns for our analysis are: 'calories', which records the calorie count for each entry; 'calorie_category', which categorizes the calorific value according to thresholds determined by quantiles; 'rating', which represents a user's individual rating for a recipe; and 'avg_rating', which provides the average rating from all users for unique recipes.

Nutritional deficiency is a big issue in many areas of the world, and understanding food trends through datasets such as the one above can give important insight into nutritional factors that are prevalent and lacking in the modern diet. Understanding food patterns and nutritional consumption is instrumental in shaping how recipes are marketed and valued.

## Data Cleaning and Exploratory Data Analysis

1. Left merge the 'recipes' and 'interactions' datasets on 'id' and 'recipe_id' respectively.
    * Each unique recipe is linked to all user interactions, allowing us to gather all ratings and reviews associated with that recipe.
2. Check data types of all columns.
    * This helps understand the current data types of the columns and potential ways they can be used for further analysis.

| Column             | Data Type   |
|:------------------ |:----------- |
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


3. Fill all ratings of 0 with np.nan.
    * The recipes were rated on a scale of 1-5. A rating of 0 signified missingness and conducting analysis with 0 values would induce a bias, so we decided to replace all 0 values with np.nan.
4. Add the 'avg_rating' column.
    * Given that numerous users provided ratings and reviews for the recipes, we chose to use the average of the ratings as a simple metric to gauge the overall likeliness of each recipe.
5. Convert nutrition values to lists and create nutrition-specific columns by splitting the list.
    * The nutrition values were originally stored as lists represented as strings. Since the nutrition values followed the format '[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), and carbohydrates (PDV)]', we first used the eval function to convert the strings into actual Python lists. After that, we applied pd.Series to split the list values into individual columns based on their positions. These columns were then assigned to specific nutrition-related column names, and finally, we joined them with the original dataframe.
6. Convert submitted and date to datetime.
    * This would help us conveniently conduct an analysis of trends and patterns over time if needed.
7. Find the calorie count thresholds based on quantiles and add a 'calorie_category' column.
    * We calculated the thresholds for the 'low', 'medium', and 'high' calorie categories using the 1/3 and 2/3 quantiles of the calorie distribution. Then, we created a function to categorize calorie values based on these thresholds and applied it to the 'calories' column, resulting in the creation of a new 'calorie_category' column.  
8. Dropped outliers beyond the 99th percentile to eliminate extreme values. 
    * Since the 'calories' column was found to have extreme outliers with high positive values, we filtered the data to only retain the values within the 99th percentile.

Here are the datatypes of all the columns of the filtered dataframe.

| Column                | Data Type   |
|:--------------------- |:----------- |
| `'name'`              | object      |
| `'id'`                | int64       |
| `'rating'`            | float64     |
| `'avg_rating'`        | float64     |
| `'calories'`          | float64     |
| `'total_fat_PDV'`     | float64     |
| `'sugar_PDV'`         | float64     |
| `'sodium_PDV'`        | float64     |
| `'protein_PDV'`       | float64     |
| `'saturated_fat_PDV'` | float64     |
| `'carbohydrates_PDV'` | float64     |
| `'calorie_category'`  | object      |
   
Our cleaned filtered dataframe had 232089 rows and 12 columns. The first 5 rows of the dataframe of unique recipes are illustrated below.

| name                                 |     id |   rating |   avg_rating |   calories |   total_fat_PDV |   sugar_PDV |   sodium_PDV |   protein_PDV |   saturated_fat_PDV |   carbohydrates_PDV | calorie_category   |
|:-------------------------------------|-------:|---------:|-------------:|-----------:|----------------:|------------:|-------------:|--------------:|--------------------:|--------------------:|:-------------------|
| 1 brownies in the world    best ever | 333281 |        4 |            4 |      138.4 |              10 |          50 |            3 |             3 |                  19 |                   6 | Low                |
| 1 in canada chocolate chip cookies   | 453467 |        5 |            5 |      595.1 |              46 |         211 |           22 |            13 |                  51 |                  26 | High               |
| 412 broccoli casserole               | 306168 |        5 |            5 |      194.8 |              20 |           6 |           32 |            22 |                  36 |                   3 | Low                |
| millionaire pound cake               | 286009 |        5 |            5 |      878.3 |              63 |         326 |           13 |            20 |                 123 |                  39 | High               |
| 2000 meatloaf                        | 475785 |        5 |            5 |      267   |              30 |          12 |           12 |            29 |                  48 |                   2 | Medium             |

# Univariate Analysis

We conducted univariate analyses on the distribution of calories, average ratings, and calorie categories in recipes. 

To begin, we plotted the distribution of calories and noticed that the data was skewed to the right. Most of the data lied in the range of 50-350 calories and there were a few extreme outliers above 1000 calories, indicating some recipes with very high calorie counts. This supports our decision to drop outliers since a few extreme values would have considerably skewed our summary statistics.

<iframe
  src="assets/univariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

On the other hand, the average ratings were greatly skewed to the left, signifying that people were likely to rate the recipes higher on the scale - most of the ratings lie between 4-5, although there were some recipes with lower ratings.

<iframe
  src="assets/univariate_ratings.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We also decided to explore a higher-level analysis by plotting the mean of the average recipe ratings for each calorie category — low, medium, and high. The error bars in the plot represent the spread of the data, calculated using the standard deviation of the average ratings per recipe in each category. Both the mean and the standard deviation of the average ratings within each category appear to be similar. Given the previously observed distributions of calories and average ratings across the dataset, this similarity can be attributed to the highly skewed nature (with most ratings falling between 4 and 5) and the limited range (a 1-5 rating scale) of the average ratings.

<iframe
  src="assets/calorie_category.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

# Bivariate Analysis

For our bivariate analyses, we delved into the relationship between calories and average ratings. We mapped out the average ratings conditioned on the calorie values for each recipe on a scatter plot to visualize the relationship across the large dataset. The graph below shows more points for higher average ratings and lower calorie counts, indicating that people, in general, tend to rate recipes higher and that recipes tend to have lower calories — which aligns with our univariate analysis. However, visually, there seems to be no clear relationship between average ratings and calories.

<iframe
  src="assets/calories-avg_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

# Interesting Aggregates

To further understand how average recipe ratings differed across the three calorie categories, we grouped the data by the calorie_category and aggregated the average ratings using mean and standard deviation. The table below was then used to visually represent the distribution of average ratings across the 3 calorie categories in a box plot.

| calorie_category   |   avg_rating |   std_dev |
|:-------------------|-------------:|----------:|
| High               |      4.66377 |  0.501469 |
| Low                |      4.68497 |  0.4973   |
| Medium             |      4.68119 |  0.494124 |

To further our analysis, we wanted to explore a comprehensive visual representation of the summary statistics presented in the table above. We used a box plot that displayed the distribution of average ratings within each calorie category, including key statistics such as the median, interquartile range (IQR), range, and the distribution of the data points, which were jittered to avoid overlapping. The distributions of the data points, the medians, the interquartile ranges, the ranges, and the significant presence of outliers all appeared to be similar across the calorie categories. This suggests that the distributions of average ratings are quite consistent; however, despite these visual similarities, we still need to determine whether this relationship is statistically significant. This will be addressed in the hypothesis testing section later.

<iframe
  src="assets/calore_category-avg_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Assessment of Missingness

Since our exploration emphasized the relationship between calories and average ratings, we decided to examine the missingness of the 'calories' column in relation to 'rating' as well as 'protein_PDV'. The latter is relevant since health enthusiasts often emphasize a high-protein diet, and understanding the impact of missing values for this feature is crucial. We added a new column to our dataframe called 'calories_missing' with boolean values of True or False based on whether the calorie values were 0 or non-zero.

# NMAR Analysis

We think that the missingness of the 'review' column is NMAR as the values that are missing correspond to an absence of a review. From a programming perspective, this is a result of merging the recipes column with the interactions column, where some recipes do not have corresponding reviews, leading to missing values in the 'review' column. From a real-world perspective, this missingness is likely to be a result of users typically leaving reviews when they feel incentivized, such as when they highly enjoy the recipe. Therefore, the absence of a review might indicate a lack of strong satisfaction with the recipe, making it NMAR.

# Missingness Dependency

The first missingness relationship that we wanted to analyze was between 'calories' and 'protein_PDV'. 

**Null Hypothesis:** The distribution of protein_PDV does not depend on the missingness of the calorie count for a recipe.

**Alternate Hypothesis:** The distribution of protein_PDV depends on the calorie count of a recipe.

**Test Statistic:** The Kolmogorov-Smirnov (K-S) test statistic which measures the largest difference between the cumulative distributions of protein_PDV when with missing and non-missing calories.

**Significance Level:** 0.05

Since most protein_PDV values were 0 when calorie values were missing (calories == 0), there wasn't enough variation in the data to create a meaningful Kernel Density Estimate (KDE) plot. KDE requires variation in the data to estimate the probability density function, but because we had mostly zero values when calories were missing, the density estimate would be a flat line or have no meaningful peaks. Therefore, instead of a KDE plot, we used a scatter plot to visualize the distribution of individual data points.

<iframe
  src="assets/scatter_missingness_protein_PDV.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Then, we ran a permutation test by shuffling the 'protein_PDV' values randomly and assigning them back to a copy of the dataset with recipes having missing and non-missing calories. Then, the K-S statistic for the 'protein_PDV' column was calculated using the two groups - the missing and non-missing calories. This was repeated 500 times and the K-S values were appended to a list. Lastly, the observed K-S statistic was compared with the K-S statistic distribution generated under the assumption of the null hypothesis.

<iframe
  src="assets/missingness_protein_PDV.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The red line highlights the observed K-S statistic of 0.953. The actual p-value that we found was 0.0. Since 0.0 < 0.05, we reject the null hypothesis - the distribution of protein depends on the missingness of calories. 


**Null Hypothesis:** The distribution of rating does not depend on the missingness of the calorie count for a recipe.

**Alternate Hypothesis:** The distribution of rating depends on the calorie count of a recipe.

**Test Statistic:** The Kolmogorov-Smirnov (K-S) test statistic which measures the largest difference between the cumulative distributions of protein_PDV when with missing and non-missing calories.

**Significance Level:** 0.05

Since there was enough variation in the 'rating' column with both missing and non-missing calories, we used a KDE plot to view the probability distributions of the recipe ratings in the aforementioned groups.

<iframe
  src="assets/kde_missingness_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

A similar method of permutation testing was used by running 500 simulations of K-S statistics by randomly shuffling recipe ratings and calculating their K-S statistic.

<iframe
  src="assets/missingness_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The red line (observed K-S statistic) here lies at 0.0511. The actual p-value that we found was 0.306. Since 0.306 > 0.05, we fail to reject the null hypothesis - there is no significant difference in the distributions of ratings based on the missingness of calories. 

## Hypothesis Testing

We aimed to investigate whether calorie categories influenced mean average ratings. Specifically, we examined if high and low calorie recipes have different mean average ratings. To do this, we used hypothesis testing because it allowed us to approximate the sampling distribution of the absolute mean difference without assuming the distribution. By comparing the observed statistic to the bootstrap distribution, we can assess whether the observed difference in mean average ratings is statistically significant.

**Null Hypothesis:** The mean average rating for low and high calories is the same.

**Alternate Hypothesis:** The mean average rating for low and high calories is different.

**Test Statistic:** The absolute difference in mean average ratings between low-calorie and high-calorie recipes.

**Significance Level:** 0.05

We repeatedly sampled high-calorie and low-calorie recipes with replacement and calculated the absolute difference in their means, repeating this process 1000 times and storing the results in a list. Subsequently, the observed mean difference was compared to the distribution of mean differences generated from samples under the assumption of the null hypothesis.

**p-value:** 0.51

<iframe
  src="assets/hypothesis_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Since 0.51 > 0.05, we fail to reject the null hypothesis. This indicates that high-calorie and low-calorie recipes do not have significantly different mean average ratings. The results suggest that users do not rate recipes differently based on their calorie count. This could imply that factors other than calorie count, such as taste, recipe quality, or another nutritional value, play a more prominent role in influencing ratings. 

## Framing a Prediction Problem

Our exploration henceforth aims to predict the calorific content in a recipe. This is a regression problem since the 'calories' column contains continuous, numerical values. 

#### Why choose calories as the response variable?

We chose calories as our response variable as during our exploration, we discovered that an increase in nutritional factors generally an increase in calories, indicating the presence of a relationship. This was also supported by our theoretical exploration where we found protein, sugar, and fat to be linked to the calorific content. 

#### Features for Prediction

To predict calories, we will use existing recipe features from the dataset: 'minutes', 'n_steps', 'total_fat_PDV', 'protein_PDV', 'carbohydrates_PDV', and 'sugar_PDV'. 

#### Evaluating Model Performance

Since this is a regression task, we will evaluate our model using:**\( RMSE \) Root Mean Squared Error** which measures the accuracy of the model's predictions. A lower RMSE indicates a better performing model. Since RSME is independent of linearity, we thought that this metric of evaluation is better as it is more versatile than using R^2 values. 

#### Why This Prediction Problem Is Meaningful

Earlier, we explored the relationship between calorie counts and recipe ratings, finding no significant relationship. Nonetheless, it is an important measure of nutritional insight and holds the potential to aid in health and food initiatives. Thus, to aim to use this prediction problem because of its simplicity and potential for nutritional impact.

#### What information is known at "time of prediction"
We have information from all the columns in the merged and the filtered dataframes from the data cleaning and exploratory data analysis section. The columns contain nutrition-specific values and other recipe-specific features.

## Baseline Model

### Description of the model and its features

For our baseline model, we used **Linear Regression** to predict the calorific content of a recipe based on the following features:  sodium ('sodium_PDV') and protein ('protein_PDV'). Both these features are continuous and quantitative. We dropped rows with missing values and split the data into training (80% of the data) and testing sets (20% of the data). 

### Necessary encodings

For preprocessing, we applied StandardScaler on the features to normalize them. This ensured that the features had equal influence on the model, even if they had different scales or units.

### Baseline Model Performance

Following this, we fit the model on the training dataset. Here is a summary of how our model performed:
- Mean Absolute Error (MAE): 157.73
- Root Mean Squared Error (RMSE): 248.5911
- R² Score: 0.4128

### Baseline Model Evaluation

The R² Score of 0.4128 indicates that the model is able to predict around 41.28% of the variance in calorie counts accurately. The high RMSE value of 248.5911 also signifies a great difference between the actual and predicted values. Further improvement on this model is needed and more features can be incorporated to help better predict calorific values.

## Final Model

### Choosing features

Our final model uses the features 'sodium_PDV', 'protein_PDV', 'saturated_fat_PDV', and 'carbohydrates_PDV' to predict 'calories'. These features were selected based on their relevance to calorie content and the relationships observed during EDA.

1. 'sodium_PDV' - since high sodium often correlates with processed or savory foods, it can be a signifier of calorific content. It was included in both our baseline and final models.
2. 'protein_PDV' - we mapped out the relationship between a recipe's protein and calorie content in the introduction - protein PDV contributes to the overall calorie count. The missingness of calories was related to the distribution of protein, further validating its importance. This feature was also used in our baseline model before.
3. 'saturated_fat_PDV' - this is another significant contributor to the calorie count. Since 1 g of fat contains 9 kcal (as mentioned in our introduction), making saturated fat an important feature.
4. 'carbohydrates_PDV' - this is another important contributor to the calorific value as 1 g of carbohydrates was found to have 4 kcal. Since carbohydrates are a key source of generating energy in the body, it is essential to include this feature when predicting calories.

Similar to the baseline model, we dropped rows with missing values and conducted an 80-20 train-test split. 

### Modelling algorithm and hyperparameters

After plotting the distributions of these features and the 'calories' column, we noticed that all distributions were skewed to the right. Referring to the [Tukey Mosteller Bulge Diagram](https://sites.stat.washington.edu/pds/stat423/Documents/LectureNotes/notes.423.ch4.pdf), we decided to use log transformations on all of the features to improve the model's fit by reducing skewness.

<iframe
  src="assets/log_all_features.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We also applied StandardScaler on the features to normalize them so that all the features have equal influence on the model, even if they have different scales or units that they were originally measured in.

Hence, we selected RandomForestRegressor to capture non-linear relationships that were visualized in our features. We performed hyperparameter tuning using GridSearchCV to find the optimal values for n_estimators, max_depth, and min_samples_split. The hyperparameter grid we searched over was:
- model__n_estimators: [50, 100, 200] - the number of trees in the forest
- model__max_depth: [10, 20, None] — the maximum tree depth
- model__min_samples_split: [2, 5, 10] — the minimum number of samples required to split a node

After running GridSearchCV with 5-fold cross-validation and totaling 135 fits, we found the best combination of hyperparameters to be model__n_estimators of 200, model__max_depth of None, and min_samples_split of 2.

### Final Model Evaluation

Here is a summary of the final model's performance:
- Mean Absolute Error (MAE): 17.07
- Root Mean Squared Error (RMSE): 54.97
- R² Score: 0.9713

Compared to our baseline model which had a R² Score of 0.7899, this is a significant improvement. This is also supported by the lower RMSE value of 54.97. Due to the inclusion of additional hyperparameters, our final model performs better than the baseline model as it explains the variance in the calorie data better, indicating predictivity. 

The actual and predicted calories for the final model are illustrated in the graph below.

<iframe
  src="assets/final_model.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Fairness Analysis

We split the recipes into **low-calorie** and **high-calorie** recipe groups using the median of the log of calorie counts which was equal to 5.7021. Since calorie distribution was skewed, the mean would be affected by outliers and extreme values so we decided that the median would be a more robust summary statistic. Calorie counts lesser than the median were considered low-calorie counts and higher than the median were considered high-calorie counts. We chose **RMSE** as our evaluation metric as it is a widely-used measure of regression tasks as it measures the error between actual and predicted values. Further, RMSE penalizes larger discrepancies more heavily, helping detect not only the presence of errors but also their impact. We used RMSE previously in our evaluations of the baseline and the final models, ensuring consistency in our approach to evaluating models. 

**Null Hypothesis:** The RMSE of low-calorie and high-calorie recipes are the same.

**Alternate Hypothesis:** The RMSE of low-calorie and high-calorie recipes are different.

**Test Statistic:** Difference in RMSE for low-calorie and high-calorie recipes

**Significance Level:** 0.05

We conducted a permutation test with 100 iterations by randomly shuffling low and high calorie labels, calculating their RMSE, and appending the resulting values to a list. The observed RMSE difference was then compared to the distribution of RMSE differences generated from samples under the assumption of the null hypothesis.
- RMSE (Low-Calorie Recipes): 10.75
- RMSE (High-Calorie Recipes): 55.09
- Observed RMSE Difference: -44.35

<iframe
  src="assets/fairness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**p-value:** 0.0800

Since 0.008 > 0.005, we fail to reject the null hypothesis - there is no significant difference in RMSE between low-calorie and high-calorie recipes. Therefore, we do not have enough evidence to conclude that our model is unfair based on calorie category. The model does not exhibit a statistically significant difference in prediction accuracy between low and high-calorie recipes, implying that the model’s performance is consistent across both categories.