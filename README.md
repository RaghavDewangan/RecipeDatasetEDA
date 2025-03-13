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



## Introduction
| `'review'`    | Review text         |

Our entire exploration is generally centric around the idea of calories and ratings. **How are ratings effected by the amount of calories , and what is the best way to tell how many calories are going to be in a recipe?** Nutrition is a big issue in many areas of the world, and understanding food trends through datasets such as the one above can give important insight into nutritional factors that are more prevalent in the modern diet, and also highlight those features that are lacking. Understanding what and how people eat is an important look into the future of the world, and can become instrumental to shaping the way food is prepared and marketed in the future. 