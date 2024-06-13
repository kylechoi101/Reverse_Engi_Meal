# Reverse Engineering for Healthy Meal

## by Kyle Choi         k3choi@ucsd.edu

## Introduction


As people started to focus more on well-being than money after subprime-mortgage and major recessions. 
The focus has been finding a healthier lifestyle.
As people focus to find a healthier life, recipy website boomed. 
There were many 'so-called' healthy meals, but do they actually have nuturitional effects that they claim to have?

In the data set I used to perform the research on had 234429 rows and 18 columns. 

Out of those, I will focus on: 'n_steps', 'name', 'n_ingredients', 'date', 'rating', and 'nutrition'.
1. **n_steps**: Number of steps in recipies (int)
1. **name**: Recipe name (str)
1. **n_ingredients** : Number of ingredients in recipies
1. **nutrition**: Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” (str)
1. **date**: Date of interaction (str)
1. **rating**: Rating a recipies got (float)

## Data Cleaning and Exploratory Data Analysis

To clean and analyze the data effectively from the recipes and interactions datasets, several key steps were taken, each addressing different aspects of the data's integrity and usability for further analysis. Here's a detailed breakdown of the process:

### 1. **Reading Data**
First, the data from two CSV files (`RAW_recipes.csv` and `RAW_interactions.csv`) was loaded into separate DataFrames called `Recipe` and `Interaction`. This step is straightforward and sets the stage for combining these datasets.

### 2. **Merging Data**
The two DataFrames were merged on the `id` field from the `Recipe` DataFrame and the `recipe_id` field from the `Interaction` DataFrame. The merge was performed as a left join (`how='left'`), ensuring all entries from the `Recipe` DataFrame were retained, adding interaction data where available. This is crucial for analyses that involve both recipe details and user interactions.

### 3. **Cleaning Ratings**
The `rating` field, which likely represents user ratings, had zeroes that were considered invalid or placeholders and thus converted to NaN (not a number). This step helps in accurate calculation of statistics like mean or median by ignoring these placeholder values.

### 4. **Calculating Average Ratings**
Post-cleaning, the average rating for each recipe was calculated using `groupby('id')['rating'].mean()` and merged back into the DataFrame. The merge was done on the `id` index of the groupby result and the `id` field of the main DataFrame. The columns were then renamed for clarity: `rating_x` to `rating` and `rating_y` to `avg_rating`. This enriches the dataset by providing a meaningful metric of overall recipe quality as perceived by users.

### 5. **Extracting Nutritional Information**
The `nutrition` column contained stringified lists with nutritional data. These were transformed into actual lists using a lambda function that strips the surrounding brackets and splits the string by commas. Each element was then cleaned to remove residual commas. A new DataFrame (`nutrition_df`) was created with these lists, specifying column names that represent each nutritional component (`calories`, `total fat`, etc.). This DataFrame was then concatenated with the original DataFrame, effectively replacing the `nutrition` string column with separate, usable numeric columns.

### 6. **Handling Outliers in Cooking Time**
To address outliers in the `minutes` column, which represents cooking time, the 1st and 99th percentiles were calculated to determine reasonable bounds. Values outside these bounds were clipped to the nearest boundary. This prevents extreme values from skewing analyses related to cooking times.

This is a snippnet of my cleaned dataframe.

| name                                 |     id |   minutes |   contributor_id | submitted   | tags                                                                                                                                                                                                                        |   n_steps | steps                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | description                                                                                                                                                                                                                                                                                                                                                                       | ingredients                                                                                                                                                                    |   n_ingredients |          user_id |   recipe_id | date       |   rating | review                                                                                                                                                                                                                                                                                                                                           |   avg_rating |   calories |   total fat |   sugar |   sodium |   protein |   saturated fat |   carbohydrates |   fatigue_factor | fatigue_factor_bins   |
|:-------------------------------------|-------:|----------:|-----------------:|:------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------:|-----------------:|------------:|:-----------|---------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------:|-----------:|------------:|--------:|---------:|----------:|----------------:|----------------:|-----------------:|:----------------------|
| 1 brownies in the world    best ever | 333281 |        40 |           985201 | 2008-10-27  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts', 'lunch', 'snacks', 'cookies-and-brownies', 'chocolate', 'bar-cookies', 'brownies', 'number-of-servings'] |        10 | ['heat the oven to 350f and arrange the rack in the middle', 'line an 8-by-8-inch glass baking dish with aluminum foil', 'combine chocolate and butter in a medium saucepan and cook over medium-low heat , stirring frequently , until evenly melted', 'remove from heat and let cool to room temperature', 'combine eggs , sugar , cocoa powder , vanilla extract , espresso , and salt in a large bowl and briefly stir until just evenly incorporated', 'add cooled chocolate and mix until uniform in color', 'add flour and stir until just incorporated', 'transfer batter to the prepared baking dish', 'bake until a tester inserted in the center of the brownies comes out clean , about 25 to 30 minutes', 'remove from the oven and cool completely before cutting']                                                  | these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven!                                                                                                              | ['bittersweet chocolate', 'unsalted butter', 'eggs', 'granulated sugar', 'unsweetened cocoa powder', 'vanilla extract', 'brewed espresso', 'kosher salt', 'all-purpose flour'] |               9 | 386585           |      333281 | 2008-11-19 |        4 | These were pretty good, but took forever to bake.  I would send it ended up being almost an hour!  Even then, the brownies stuck to the foil, and were on the overly moist side and not easy to cut.  They did taste quite rich, though!  Made for My 3 Chefs.                                                                                   |            4 |      138.4 |          10 |      50 |        3 |         3 |              19 |               6 |          65.9489 | Low                   |
| 1 in canada chocolate chip cookies   | 453467 |        45 |          1848091 | 2011-04-11  | ['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', 'north-american', 'for-large-groups', 'canadian', 'british-columbian', 'number-of-servings']                                                               |        12 | ['pre-heat oven the 350 degrees f', 'in a mixing bowl , sift together the flours and baking powder', 'set aside', 'in another mixing bowl , blend together the sugars , margarine , and salt until light and fluffy', 'add the eggs , water , and vanilla to the margarine / sugar mixture and mix together until well combined', 'add in the flour mixture to the wet ingredients and blend until combined', 'scrape down the sides of the bowl and add the chocolate chips', 'mix until combined', 'scrape down the sides to the bowl again', 'using an ice cream scoop , scoop evenly rounded balls of dough and place of cookie sheet about 1 - 2 inches apart to allow for spreading during baking', 'bake for 10 - 15 minutes or until golden brown on the outside and soft & chewy in the center', 'serve hot and enjoy !'] | this is the recipe that we use at my school cafeteria for chocolate chip cookies. they must be the best chocolate chip cookies i have ever had! if you don't have margarine or don't like it, then just use butter (softened) instead.                                                                                                                                            | ['white sugar', 'brown sugar', 'salt', 'margarine', 'eggs', 'vanilla', 'water', 'all-purpose flour', 'whole wheat flour', 'baking soda', 'chocolate chips']                    |              11 | 424680           |      453467 | 2012-01-26 |        5 | Originally I was gonna cut the recipe in half (just the 2 of us here), but then we had a park-wide yard sale, & I made the whole batch & used them as enticements for potential buyers ~ what the hey, a free cookie as delicious as these are, definitely works its magic! Will be making these again, for sure! Thanks for posting the recipe! |            5 |      595.1 |          46 |     211 |       22 |        13 |              51 |              26 |          81.9953 | Low                   |
| 412 broccoli casserole               | 306168 |        40 |            50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                        |         6 | ['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese , garlic powder , pepper , salt , milk , 1 cup of french onions , and soy sauce', 'pour into baking dish , sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly , about 10 more minutes']                                                                                                                                                                                                                                                                                                                              | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', 'garlic powder', 'ground black pepper', 'salt', 'milk', 'soy sauce', 'french-fried onions']          |               9 |  29782           |      306168 | 2008-12-31 |        5 | This was one of the best broccoli casseroles that I have ever made.  I made my own chicken soup for this recipe. I was a bit worried about the tsp of soy sauce but it gave the casserole the best flavor. YUM!                                                                                                                                  |            5 |      194.8 |          20 |       6 |       32 |        22 |              36 |               3 |          53.9944 | Low                   |
|                                      |        |           |                  |             |                                                                                                                                                                                                                             |           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |                                                                                                                                                                                                                                                                                                                                                                                   |                                                                                                                                                                                |                 |                  |             |            |          | The photos you took (shapeweaver) inspired me to make this recipe and it actually does look just like them when it comes out of the oven.                                                                                                                                                                                                        |              |            |             |         |          |           |                 |                 |                  |                       |
|                                      |        |           |                  |             |                                                                                                                                                                                                                             |           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |                                                                                                                                                                                                                                                                                                                                                                                   |                                                                                                                                                                                |                 |                  |             |            |          | Thanks so much for sharing your recipe shapeweaver. It was wonderful!  Going into my family's favorite Zaar cookbook :)                                                                                                                                                                                                                          |              |            |             |         |          |           |                 |                 |                  |                       |
| 412 broccoli casserole               | 306168 |        40 |            50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                        |         6 | ['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese , garlic powder , pepper , salt , milk , 1 cup of french onions , and soy sauce', 'pour into baking dish , sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly , about 10 more minutes']                                                                                                                                                                                                                                                                                                                              | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', 'garlic powder', 'ground black pepper', 'salt', 'milk', 'soy sauce', 'french-fried onions']          |               9 |      1.19628e+06 |      306168 | 2009-04-13 |        5 | I made this for my son's first birthday party this weekend. Our guests INHALED it! Everyone kept saying how delicious it was. I was I could have gotten to try it.                                                                                                                                                                               |            5 |      194.8 |          20 |       6 |       32 |        22 |              36 |               3 |          53.9944 | Low                   |
| 412 broccoli casserole               | 306168 |        40 |            50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                        |         6 | ['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese , garlic powder , pepper , salt , milk , 1 cup of french onions , and soy sauce', 'pour into baking dish , sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly , about 10 more minutes']                                                                                                                                                                                                                                                                                                                              | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', 'garlic powder', 'ground black pepper', 'salt', 'milk', 'soy sauce', 'french-fried onions']          |               9 | 768828           |      306168 | 2013-08-02 |        5 | Loved this.  Be sure to completely thaw the broccoli.  I didn&#039;t and it didn&#039;t get done in time specified.  Just cooked it a little longer though and it was perfect.  Thanks Chef.                                                                                                                                                     |            5 |      194.8 |          20 |       6 |       32 |        22 |              36 |               3 |          53.9944 | Low                   |





<iframe
  src="univariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The mean cooking time is approximately 64.66 minutes, which is considerably higher than the median of 35 minutes. 



<iframe
  src="bivariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


Most recipes likely require a relatively low number of ingredients(20) and steps(40), which is typical for everyday or simplified recipes. This clustering can make it hard to discern patterns in the rest of the data.



| Fatigue Factor Bins | n_steps (mean) | n_steps (median) | n_steps (std) | minutes (mean) | minutes (median) | minutes (std) |
|---------------------|----------------|------------------|---------------|----------------|------------------|---------------|
| Low                 | 10.0016        | 9                | 6.35015       | 64.5566        | 35               | 100.862       |
| Medium              | 78.425         | 80               | 10.5779       | 475            | 502.5            | 201.447       |
| High                | 91.5385        | 88               | 5.547         | 660            | 660              | 0             |


- **Low Fatigue**: Most recipes are simple and quick, with low variability in steps and cooking time.
- **Medium Fatigue**: A significant increase in both steps and time, with more variability, suggesting more complex recipes that are still within a reasonable completion time for an experienced cook.
- **High Fatigue**: Highest steps and consistent maximum cooking time, indicating the most labor-intensive recipes. Notably, the standard deviation for minutes here is zero, implying no variation among the high fatigue recipes—they all max out at the dataset's limit for cooking time.

### Significance

This table is crucial for understanding how recipe complexity (as indicated by the number of steps) and required time impact the cook's fatigue. It could be particularly useful for:

- **Recipe Developers**: To balance complexity and cooking time to target different user groups, such as quick meals for busy individuals or more engaging recipes for cooking enthusiasts.
- **Nutritional Experts**: To explore if there’s a correlation between nutritional content and the fatigue factor, potentially guiding healthier meal plans that are also easier to prepare.


## Assessment of Missingness
In our dataset, the 'description' column exhibits missingness in 114 entries. I hypothesize that this missingness is NMAR, as it potentially correlates with the nature of the entries themselves—possibly entries made as jokes or as mistakes, thus lacking serious descriptions. This theory suggests that the missing descriptions are not randomly distributed but are instead systematically missing based on the unobserved seriousness of the recipe content. To better analyze this, incorporating an 'age' column could be valuable. If younger users are more likely to post non-serious recipes, the age data might explain the missingness, potentially reclassifying it from NMAR to MAR by linking it to an observed variable


<iframe
  src="distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
<iframe
  src="emp_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

I used difference in mean as my statistic to find MAR column. I chose avg_rating, because it was kind of modified version of rating.

p-value for avg_rating missing dependent on user_id  = 0.0
The p-value close to 0 for avg_rating missingness with respect to user_id suggests a strong relationship between specific users and their likelihood to provide ratings. Users who typically rate recipes appear to consistently do so, while those who do not rate, maintain this behavior consistently, indicating a non-random pattern of missing data. 

p-value for avg_rating missing dependent on reviews per recipies = 1.0
The p-value of 1.0 indicates that the number of reviews a recipe receives does not affect the likelihood of avg_rating being missing. This suggests that missingness in avg_rating is independent of review counts, supporting the hypothesis that it is missing completely at random (MCAR) with respect to this variable.

## Hypothesis Testing

* Null Hypothesis (H0): The amount of sugar is the same between between normal recipes and high sodium recipes.
* Alternative Hypothesis (H1): The amount of sugar differs between normal recipes and high sodium recipes.

at 0.05 siginificance level.

<iframe
  src="sugar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

KS test p-value of 0.0 strongly suggests that we reject the null hypothesis.

This indicates that there are statistically significant differences in the sugar content distributions between the low sodium and higher sodium groups.

Although foods with low sodium are often perceived as healthier, this analysis underscores the importance of also considering sugar content. In light of these findings, it is crucial to approach low-sodium options with caution, particularly when they contain high levels of sugar. This holistic nutritional perspective is essential for making better research.


## Framing a Prediction Problem

Classification:
"Can we predict if food is healthy based on its nutritional content?" 
- **Binary Classification**: If "healthy" is defined as a binary outcome (healthy or not healthy).

#### Response Variable

The response variable is a binary indicator (0 or 1) or a category label, depending on your definition of healthy. For binary classification:
   - `0` could represent "not healthy,"
   - `1` could represent "healthy."
   
The question is about the healthiness of food, which is likely determined by specific thresholds or criteria based on nutritional content (like calories, fat, sugar, etc.).

#### Evaluation Metric

- **Accuracy**: I will mainly focus because I am equally concerned with identifying both.
- **Precision and Recall (F1-score)**: If there's an imbalance in the classes, or if the cost of false positives and false negatives differs significantly. I will also aim to increase F1 


Accuracy is simple and gives a quick snapshot of overall performance. 

The F1-score, which balances precision and recall, is often more useful in such cases because it accounts for both the false positives and false negatives, providing a more holistic view of model performance.

#### Time of Prediction Consideration

In developing our model to predict the healthiness of food products, we have carefully selected features that are readily available at the point of decision-making—specifically, before purchase and consumption. These features include 'total fat', 'sugar', 'sodium', 'protein', 'saturated fat', 'carbohydrates', and 'calories', all of which are standard on food nutrition labels. Our model assumes that these labels provide accurate and up-to-date nutritional values, reflecting current product formulations. This model is designed to be used at the time of purchase, allowing consumers to scan a product's label and receive immediate feedback on its healthiness based on the nutritional content provided.


## Baseline Model

#### Model Description

I implemented a Decision Tree Classifier with a maximum depth of 5. I employed this model to predict whether food is considered healthy based on several nutritional factors.

#### Features in the Model

- **Fatigue Factor**: Quantitative.
- **Total Fat**: Quantitative.
- **Sugar**: Quantitative. 
- **Sodium**: Quantitative. 
- **Protein**: Quantitative. 
- **Saturated Fat**: Quantitative. 
- **Carbohydrates**: Quantitative. 
- **Calories**: Quantitative. 

All features are quantitative and directly related to the nutritional content of food items, which are typically available on food labels, making them practical for real-time predictions.

#### Data Encoding

Since all the features in your model are quantitative, no special encoding strategies (like one-hot encoding for categorical data or ordinal encoding for ordinal data) are necessary. However, it's crucial to ensure that these features are scaled appropriately if you decide to use models sensitive to feature scaling in the future.

#### Model Performance

- **Overall Accuracy**: 86%. This seems like a solid score at first glance.
- **Confusion Matrix**:
  - True Negatives: 40095
  - False Positives: 5
  - False Negatives: 6780
  - True Positives: 6

**Precision, Recall, and F1-Score for Each Class**:
- **Non-Healthy Class (False)**:
  - Precision: 86%
  - Recall: Nearly 100%
  - F1-Score: 92%
- **Healthy Class (True)**:
  - Precision: 55%
  - Recall: Approximately 0%
  - F1-Score: 0%

#### Evaluation of the Model's Effectiveness

While the overall accuracy of the model is high (86%), this metric is misleading.

**Is the Model "Good"?**
- **No**, the model is not good for practical purposes, despite high accuracy, because it fails to serve the primary objective of effectively identifying healthy foods. The model is heavily biased towards predicting most items as non-healthy, which significantly undermines its utility.


## Final Model

**Accuracy**: 0.94

**Confusion Matrix**:
|       | Predicted False | Predicted True |
|-------|-----------------|----------------|
| False | 38033           | 349            |
| True  | 2696            | 5808           |

**Classification Report**:
|          | Precision | Recall | F1-Score | Support |
|----------|-----------|--------|----------|---------|
| **False** | 0.93      | 0.99   | 0.96     | 38382   |
| **True**  | 0.94      | 0.68   | 0.79     | 8504    |

**Derived Feature (`calories_high`)**:
This feature is a binary indicator derived from the `calories` feature, where foods above the median calorie value are flagged as high calorie.

This reflects the practical understanding that not all calories are equal, and calorie density can influence satiety, metabolic rates, and nutritional health outcomes.

**Gradient Boosting Classifier** was chosen for the final model due to its robustness in handling various types of data and its effectiveness in improving prediction accuracy through ensemble learning:

Gradient boosting builds an ensemble of weak prediction models, typically decision trees, in a stage-wise fashion. It optimizes for a loss function, with each new tree making up for the errors of those previously fitted.
- **Key Hyperparameters**:
  - `n_estimators=100`: The number of trees in the forest. A higher number of trees can improve the model's ability to generalize but also increases computational cost.
  - `max_depth=12`: Controls the maximum depth of each tree. Deeper trees can model more complex patterns but can lead to overfitting.

Hyperparameters were likely tuned using either cross-validation techniques such as GridSearchCV or RandomizedSearchCV, which systematically explore combinations of parameters to find the most effective model settings.



**Baseline Model**: Initially, it used a Decision Tree Classifier with a simple setup. This model likely served as a baseline with basic interpretability but limited accuracy and generalization capability due to its simplicity and proneness to overfitting.

**Final Model**: The Gradient Boosting Classifier significantly improved upon the baseline model:
- **Accuracy Increase**: The final model achieved a higher accuracy (94%), showing a marked improvement over the baseline.
- **Recall and Precision for 'Healthy' Class**: These metrics saw significant improvement, which is crucial for your task since identifying healthy foods accurately is more critical than just predicting the most frequent class.
- **Handling Class Imbalance**: The final model's enhanced ability to handle class imbalance through techniques embedded in gradient boosting and possibly through adjusted class weights further helped in improving performance metrics across both classes.

## Fairness Analysis

#### Groups Defined

- **Group X (Pre-2013)**: This group consists of predictions made from data collected before the year 2013.
- **Group Y (Post-2013)**: This group includes predictions from data collected in the year 2013 and afterwards.

#### Evaluation Metric

- **Precision**: You've chosen precision as the evaluation metric to assess the proportion of correct positive predictions (true positives) out of all positive predictions (true and false positives) made by the model.

#### Null and Alternative Hypotheses

- **Null Hypothesis (H0)**: There is no difference in the precision of predictions made by the model for data before 2013 and data from 2013 onwards. Formally, $( \text{Precision}_{\text{pre-2013}} = \text{Precision}_{\text{post-2013}} $).
- **Alternative Hypothesis (H1)**: There is a difference in the precision of predictions between the two time periods. Formally, $( \text{Precision}_{\text{pre-2013}} \neq \text{Precision}_{\text{post-2013}} $).

#### Test Statistic

- **Difference in Precision**: The test statistic used is the difference in precision scores between the two groups, calculated as $( \text{Precision}_{\text{pre-2013}} - \text{Precision}_{\text{post-2013}} $).

#### Significance Level

- Significance level of 0.05 (5%) is used
#### Resulting p-value

- **p-value**: 0.2017

#### Conclusion

Based on the p-value obtained from your permutation test (0.2017), which is  higher than the significance level of 0.05, you fail to reject the null hypothesis. This suggests that there is no statistically significant difference in the precision of the model's predictions before and after 2013. 




P-value: 0.2017
