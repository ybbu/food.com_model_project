# Recipes & Ratings Regressor Model

My exploratory data analysis on this dataset can be found [here](https://ybbu.github.io/food.com_analysis_project/)...
## Framing the Problem

In this project, I built a regressor model to **predict the average rating of a recipe** based on data from [Food.com](https://www.food.com/). 

#### Breaking Down

The response variable for my prediction problem is simply what I want to predict - the average rating calculated from all reviews, which is a continuous, quantitative variable. 

The metric I would use to evaluate my model's performance would be RMSE(root mean squared error). I chose this metric because it is commonly used to evaluate how well a regressor model is able to predict accurately. Specifically, RMSE measures the average differences between the predicted values and the target values, so my goal in this project is to build a regressor model that minimizes RMSE.

The goal of the project is to help submitters to estimate the ratings of their recipes before the recipes are posted, so I won't use information from the reviews as they are not yet available to the submitters at that moment.

## Baseline Model

First of all, I created a baseline model using <mark>Decision Tree regressor</mark>. The features I used are the recipe's cooking time (`minutes`), the number of steps in the recipe (`n_steps`), the number of ingredients to prepare (`n_ingredients`), and the number of calories in the recipe (`calories`). All four features are quantitative, discrete variables. For the baseline model, I didn't perform any encodings.

*(Note: I chose the Decision Tree regressor because based on my previous exploratory data analysis, the features don't quite share a linear relationship with the response variable, which is average rating. Also, there are large outliers in some of the features like `minutes` which would strongly influence the best fitting line of a regressor model. Therefore, I used a nonlinear regressor model that's robust to outliers in this problem, which is the Decision Tree regressor)*

#### Model Performance

The performance of the baseline model, in terms of RMSE, on the training set is 0.027 (rounded to 3 decimal places). On the test data, the performance, in terms of RMSE, is around **0.944** (rounded to 3 decimal spaces).

Overall, the baseline model is not too bad because the RMSE values for both the training set and test set are below 1, which is quite small. However, the large difference in the model's performance on the training set and test set shows that the model is overfitting to the training data and generalizes poorly to unseen data.

In future steps, my primary goal would be to improve the model's performance on unseen data.

## Final Model

#### Feature Encoding

**Quantile Transformation:**

The first thing I did was to transform all the quantitative variables (`n_steps`, `minutes`, `n_ingredients`, and `calories`) with quantile information. I did this for two reasons. 
- First, the quantitative variables have very different ranges. For example, the values in `calories` range from 0 to 45609 while the values in `n_ingredients` only range from  1 to 37. In this case, the `calories` will automatically carry more weightage than the `n_steps`. Using quantile transformation will bring all quantitative variables to the same scale so they will contribute equally to the analysis. 
- Secondly, both`calories` and `minutes` have very large outliers which would stay large even after z-score standardization. Quantile transformation can solve this problem because it performs a nonlinear transformation by squeezing all values into 0 to 1.

**New Categorical Feature:**

The second thing I did was encode a new feature about the recipe's course category from the `tag` column in the dataset. I added this feature because users may have different rating standards for different types of recipes. For example, users may value the number of calories most when rating a salad recipe. When it comes to rating a breakfast recipe, users may focus more on the number of steps instead.

Since each recipe may be labeled with multiple course categories, I used a MultiLaber Binarizer to encode the feature.

#### Model Fine-tuning

I decided to use the model I used in the baseline model for my final model due to the reasons stated in the Baseline section. 

To fine-tune the model, I used 5-fold cross-validation and performed a grid search to find the combination of hyperparameters that maximizes average validation accuracy. The hyperparameters I chose to fine-tune were:
- *splitter*: strategy used to choose the split at each node
- *max_features*: the number of features to consider when looking for the best split
- *max_depth*: the maximum depth of the tree
- *min_samples_split*: the minimum number of samples required to split an internal node
All of the hyperparameters manipulate the complexity of the model and therefore affect the model's generalizability to unseen data. 

The best combinations of hyperparameters are: `splitter='random', max_features=None, max_depth=100, min_samples_split=0.5`. Using those hyperparameters, the performance of my final model, in terms of RMSE, on the training set is 0.638 (rounded to 3 decimal places). On the test data, the performance, in terms of RMSE, is around **0.641** (rounded to 3 decimal spaces). 

To conclude, though the final model performs worse on training sets as compared to the baseline model, its performance on the test set increases. And more importantly, the difference in the model's performance on the training set and test set shrinks from 0.917 to 003. This is a large improvement because, unlike the baseline model, my final model performs nearly equally well on the test set and training set, showing its good generalizability to unseen data.

## Fairness Analysis

Here I performed a fairness analysis of my final model's performance, in terms of RMSE, in recipes with different calorie levels. I artificially defined the threshold for being a high-calorie recipe to be 780. Specifically, my hypothesis is as follows:

- **Null Hypothesis**: My model is fair. Its RMSE for high-calorie recipes (#calorie > 800) and normal (#calorie <= 800) recipes are roughly the same, and any differences are due to random chance.
- **Alternative Hypothesis**: My model is unfair. Its RMSE for high-calorie recipes (#calorie > 800) is different from its RMSE for normal recipes (#calorie <= 800).

My choice of test statistic was the absolute RMSE difference between the two groups. And my significance level is 0.05.

#### Conclusion

The resulting p-value is 0.011. A visualization of the permutation test is as follows: 
<iframe src="assets/test.html" width=800 height=600 frameBorder=0></iframe>
To conclude, since the p-value is less than the significance level, I reject the null hypothesis. The result suggests that my final model is not fair. It appears to be performing differently for high-calorie recipes and normal recipes. 

