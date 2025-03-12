# Exploring the Relationships Between The Calories in a Recipe vs. the Average Rating

Authors: Raghav Dewangan and Supriyaa Chordia

Exploratory Data Analysis, Hypothesis Testing, Predictive Modelling on recipes dataset

## Introduction

Health and food are two deeply intertwined things of our daily lives, where one strongly effects the other. Food and meals are all a result of a compilation of macronutrients, ingredients, preparation steps, and time. These are all the things that our dataframes below classify and identify. 


Below is the 'recipes' dataframe. There are 10 columns and 83782 rows. Each row corresponds to an individual recipe (recipe_id or name for reference) with their own nutritional and preparational stats. 

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


## Data Cleaning and Exploratory Data Analysis



## Assessment of Missingness




## Hypothesis Testing




## Framing a Prediction Problem




## Baseline Model



## Final Model


## Fairness Analysis



