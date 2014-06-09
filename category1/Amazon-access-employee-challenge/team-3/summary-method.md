Summary of our approach.
========================
Our result submission was linear combination of 11 models: 6 GBM models with different sets of features (package gbm in R), 2 GLMNET models (package glmnet in R), 1 RF model (randomForest package in R), 2 logistic regression models (based on Miroslaw code).
1) GBM models were based on different sets of features, here the ideas:
1.1) We used categorical features as numerical in several GBM models.
1.2) We also removed role1 and family from several GBM models.
1.3) Homogeneous ensembling for all GBM models (average of predictions based on parts of the train set).
1.4) One GBM model was with likelihood features, this was the most accurate individual model.
Here is the description from Lucas (Leustagos):
The likelihood dataset consisted on replacing each categorical value (id) for its likelihood. 
Usually this procedure overfits badly, so in order to get around this, we did the following:
Used shrunken likelihood instead of raw likelihood. The shrunken likelihood can be defined as: shr_likelihood = alpha*global_likelihood + (1-alpha)*item_likelihood, where global_likelihood is the global average, item_likelihood is the likelihood of each categorical value, and alpha is a function on the number of occurrences of each item. It could be something like alpha(n) = 1/(n+1).
So if a item occurred many times, its likelihood will be pretty close to item_likelihood. If it occurred a few times (or never occurred before) it will be close to global_likelihood. To calculate this alpha we used the lme4 package of R as a black box. As stated in another forum thread, using the outputs to produce features tends to overfit. To solve this issue, we used nested 
cross-validation. The cross validation usually splits the set into a validation and a training set. The likelihood of the validation set will be calculated only on the training set (so we never use validation outputs). But it still overfits. Se we took the training part and did another 10-fold validation on it. The likelihood of each fold is calculated based on the other 9. Because of this we were forced to have as many likelihood datasets as we had folds, but at least it didnt overfit! We also calculated likelihood for the combination of ids. We used 1,2,3,6,7 degrees of combination here.
1.5) One GBM model was based only on occurrences count for initial variables and their combinations.
1.6) One GBM model was based on distance features: we introduced new categorical variable "person" - different combinations of 7 initial categorical features without resource (there about 12000 persons in the dataset). After we created the graph of persons: two persons are connected if they have common resource. Based on this we created features for each person based on it neighbors.
1.7) In one GBM model we transformed target variable based on boolean XOR function. New target variable = XOR (output of libfm algorithm, initial target variable).
2) GLMNET models were just homogeneous ensembling of 200 glmnet models with different sets of dummy features.
3) RF model was pretty similar to GBM model with categorical features transformed to numbers and some count features, like number of persons with the same resource in the department.
4) Logistic regression model was just modified Miroslaw's code with some improvements: faster search, black list of features and so on.
The final prediction was linear combination with coefficients found based on cv results.