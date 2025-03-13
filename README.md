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


Our exploration was centered around the idea of calories and ratings. **How does calorie count affect a recipe's rating, and what is the best way to estimate calories based on a recipe’s rating?** We separated the 'nutrition' column values into nutrition-specific columns named 'calories', 'total_fat_PDV', 'sugar_PDV', 'sodium_PDV', 'protein_PDV','saturated_fat_PDV', and 'carbohydrates_PDV'. In nutrition labels, "PDV" stands for Percentage Daily Value, and it quantifies the extent of a nutrition's contribution in a total daily diet. We also added the columns named 'calorie_category', which classifies recipes into 3 categories: 'high', 'medium', and 'low' based on their calorific content, and ‘avg_rating’, which displayed the average rating for each recipe. The most important columns for our analysis are: 'calories', which records the calorie count for each entry; 'calorie_category', which categorizes the calorific value according to thresholds determined by quantiles; 'rating', which represents a user's individual rating for a recipe; and 'avg_rating', which provides the average rating from all users for unique recipes.

Nutritional deficiency is a big issue in many areas of the world, and understanding food trends through datasets such as the one above can give important insight into nutritional factors that are prevalent and lacking in the modern diet. Understanding food patterns and nutritional consumption is instrumental in shaping how recipes are marketed and valued.

## Data Cleaning and Exploratory Data Analysis

1. Left merge the 'recipes' and 'interactions' datasets on 'id' and 'recipe_id' respectively.
    * Each unique recipe is linked to all user interactions, allowing us to gather all ratings and reviews associated with that recipe.
2. Check datatypes of all columns.
    * This helps understand current data types of the columns and potential ways they can be used for further analysis.

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
    * The recipies were rated on a scale of 1-5. A rating of 0 signified missingness and conducting analysis with 0 values would induce a bias, so we decided to replace all 0 values with np.nan.
4. Add the 'avg_rating' column.
    * Given that numerous users provided ratings and reviews for the recipes, we chose to use the average of the ratings as a simple metric to gauge the overall likeliness of each recipe.
5. Convert nutrition values to lists and create nutrition-specific columns by splitting the list.
    * The nutrition values were originally stored as lists represented as strings. Since the nutrition values followed the format '[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), and carbohydrates (PDV)]', we first used the eval function to convert the strings into actual Python lists. After that, we applied pd.Series to split the list values into individual columns based on their positions. These columns were then assigned to specific nutrition-related column names, and finally, we joined them with the original dataframe.
6. Convert submitted and date to datetime.
    * This would help us conveniently conduct an analysis of trends and patterns over time if needed.
7. Find the calories thresholds based on quantiles and add a 'calorie_category' column to categorize calorie content.
    * We calculated the thresholds for 'low', 'medium', and 'high' calorie categories using the 1/3 and 2/3 quantiles of the calorie distribution. Then, we created a function to categorize calorie values based on these thresholds and applied it to the 'calories' column, resulting in the creation of a new 'calorie_category' column.  
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
   
Our cleaned filtered dataframed had 232089 rows and 12 columns. The first 5 rows of the dataframe of unique recipies are illustrated below.

| name                                 |     id |   rating |   avg_rating |   calories |   total_fat_PDV |   sugar_PDV |   sodium_PDV |   protein_PDV |   saturated_fat_PDV |   carbohydrates_PDV | calorie_category   |
|:-------------------------------------|-------:|---------:|-------------:|-----------:|----------------:|------------:|-------------:|--------------:|--------------------:|--------------------:|:-------------------|
| 1 brownies in the world    best ever | 333281 |        4 |            4 |      138.4 |              10 |          50 |            3 |             3 |                  19 |                   6 | Low                |
| 1 in canada chocolate chip cookies   | 453467 |        5 |            5 |      595.1 |              46 |         211 |           22 |            13 |                  51 |                  26 | High               |
| 412 broccoli casserole               | 306168 |        5 |            5 |      194.8 |              20 |           6 |           32 |            22 |                  36 |                   3 | Low                |
| millionaire pound cake               | 286009 |        5 |            5 |      878.3 |              63 |         326 |           13 |            20 |                 123 |                  39 | High               |
| 2000 meatloaf                        | 475785 |        5 |            5 |      267   |              30 |          12 |           12 |            29 |                  48 |                   2 | Medium             |

# Univariate Analysis

We conducted univariate analyses on the distribution of calories, avergae ratings, and calorie categories in recipes. 

To begin, we plotted the distribution of calories and noticed that the data is skewed to the right. Most of the data lied between the range of 50-350 calories and there were a few extreme outliers above 1000 calories, indicating some recipes with very high calorie counts. This supports our decision of dropping outliers since a few extreme values would have considerably skewed our summary statistics.

<iframe
  src="assets/univariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

On the other hand, the avgerage ratings were greatly skwewed to the left, signifying that people were likely to rate the recipes higher on the scale - most of the ratings lies between 4-5, although there were some recipes with lower ratings.

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

To gain further understand how average recipe rating differed acorss the three calorie categories, we grouped the data by the calorie_category and aggregated the average ratings using mean and standard deviation. The table below was then used to visually represent the distribution of average ratings across the 3 calorie categories in a box plot.

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

Since our exploration empahsized the relationship between calories and average ratings, we decided to examine the missingness of the 'calories' column in relation to 'rating' as well as 'protein_PDV'. The latter is relevant since health enthusiasts often emphasize a high-protein diet, and understanding the impact of missing values for this feature is crucial. We added a new column to our dataframe called 'calories_missing' with boolean values of True or False based on whether the calorie values were 0 or non-zero.

# NMAR Analysis

We think that the misisngness of the 'review' column is NMAR as the values that are missing coresspond to an absence of a review. From a programming perspective, this is a result of merging the recipies column with the interactions column, where some recipes do not have corresponding reviews, leading to missing values in the 'review' column. From a real-world perspective, this missingness is likely to be a result of users typically leaving reviews when they feel incentivized, such as when they highly enjoy the recipe. Therefore, the absence of a review might indicate a lack of strong satisfaction with the recipe, making it NMAR.

# Missingness Dependency

The first missingness relatonship that we wanted to analyze was between 'calories' and 'protein_PDV'. 

**Null Hypothesis:** The distribution of protein_PDV does not depend on the missigness of the calorie count for a recipe.

**Alternate Hypothesis:** The distribution of protein_PDV depends on the calorie count of a recipe.

**Test Statistic:** The Kolmogorov-Smirnov (K-S) test statistic, which measures the largest difference between the cumulative distributions of protein_PDV when with missing and non-missing calories.

**Significance Level:** 0.05

Since most protein_PDV values were 0 when calorie values were missing (calories == 0), there wasn't enough variation in the data to create a meaningful Kernel Density Estimate (KDE) plot. KDE requires variation in the data to estimate the probability density function, but because we had mostly zero values when calories were missing, the density estimate would be a flat line or have no meaningful peaks. Therefore, instead of a KDE plot, we used a scatter plot to visualize the distribution of individual data points.

<iframe
  src="assets/scatter_missingness_protein_PDV.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Then, we ran a permutation test by shuffling the 'protein_PDV' values randomly and assigning it back to a copy of the dataset with recipes having missing and non-missing calories. Then, the K-S statistic for the 'protein_PDV' column was calculated using the two groups - the missing and non-missing calories. This was repeated 500 times and the K-S values were appended to a list. Lastly, the observed K-S statistic was compared with the K-S statsitic distribution generated under the assumption of the null hypothesis.

<iframe
  src="assets/missingness_protein_PDV.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The red line highlights the observed K-S statistic of 0.953. The actual p-value that we found was 0.0. Since 0.0 < 0.05, we reject the null hypothesis - the distribution of protein depends on the missigness of calories. 


**Null Hypothesis:** The distribution of rating does not depend on the missigness of the calorie count for a recipe.

**Alternate Hypothesis:** The distribution of rating depends on the calorie count of a recipe.

**Test Statistic:** The Kolmogorov-Smirnov (K-S) test statistic, which measures the largest difference between the cumulative distributions of protein_PDV when with missing and non-missing calories.

**Significance Level:** 0.05

Since there was enough variation in the 'rating' column with both missing and non-missing calories, we used a KDE plot to view the probability distributions of the recipe ratings in the aforementioned groups.

<iframe
  src="assets/kde_missingness_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

A similar method of permutation testing was used by running 500 simulations of K-S statistics by randomly shuffling recipe rating and calculating their K-S statistic.

<iframe
  src="assets/missingness_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The red line (observed K-S statistic) here lies at 0.0511. The actual p-value that we found was 0.306. Since 0.306 > 0.05, we fail to reject the null hypothesis - there is no significant difference in the distributions of rating based on the missigness of calories. 

## Hypothesis Testing

We aimed to investigate whether calorie categories influenced mean average ratings. Specifically, we examined if high and low calorie recipes have different mean average ratings. To do this, we used chose hypothesis testing because it allowed us to approximate the sampling distribution of the absolute mean difference without assuming the distribution. By comparing the observed statistic to the bootstrap distribution, we can assess whether the observed difference in mean average ratings is statistically significant.

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




## Baseline Model





## Final Model




## Fairness Analysis









