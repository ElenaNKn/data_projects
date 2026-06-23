# Multiclass classification with imbalanced classes

## Project Description

The problem of multiclass classification with class imbalance in the initial data is considered in this project.

The project was carried out using the `load_wine` dataset from the `sklearn.datasets` library. A number of models (logistic regression, support vector machine, k-nearest neighbors, decision tree) were built to predict the wine class based on its characteristics.

## Project Execution Stages

1. Dataset Analysis (EDA), investigation of pairwise feature correlations, checking for missing values and outliers, feature standardization. Specification the metric to describe the performance of the models.

2. Building a logistic regression model. Cross-validation-based search for optimal parameters. Model performance evaluation on the test dataset.

3. Building a multiclass classification model based on the Support Vector Machine (SVM) method. Development of a custom approach for searching optimal model parameters (regularization coefficient and kernel type). Model performance evaluation on the test dataset.

4. Building a multiclass classification model based on the K-Nearest Neighbors (KNN) algorithm. Determining the optimal number of neighbors and the method for calculating weight coefficients for neighbors. Model performance evaluation on the test dataset.

5. Building a classification model based on a decision tree. Selection of optimal parameters. Visualization of the decision tree and model performance evaluation on the test dataset.

## Conclusions

The task under investigation is multiclass classification with imbalanced classes. Therefore, the f1_weighted metric (`weighted f1-score`) was adopted as the target metric.

For this dataset, the "one-versus-rest" and "one-versus-one" strategies were found to perform equivalently when reducing the problem to binary classification. This is largely because the dataset is relatively small and contains few classes. Meanwhile "one-versus-rest" approach tends to work better with many classes, whereas "one-versus-one" is generally preferred for very large datasets.

A logistic regression model was built. Based on a grid search via cross-validation (using `GridSearchCV`), the optimal regularization parameter `C = 0.06` was obtained. The `f1_weighted` for the constructed logistic regression model with the selected optimal regularization parameter value on the test set was 1.

A multiclass classification model based on the SVM (Support Vector Machine) method was built (a composite model `OneVsRestClassifier(SVC())`). Hyperparameter tuning for the created model via cross-validation was implemented explicitly, and the optimal regularization parameter value and kernel type were found. The target metric `f1_weighted` for the constructed model on the test set was 0.96 (the model misclassified 1 instance from class 1 and 1 instance from class 2; instances of class 0 in the test set were classified without error).

A multiclass classification model based on the KNN algorithm was built. Via cross-validation grid search (`GridSearchCV`), the optimal number of neighbors and the method for calculating weight coefficients for neighbors were determined. The target metric `f1_weighted` for the constructed model on the test set was 0.98. A more detailed analysis showed that the model misclassified only 1 instance of class 1 (incorrectly assigned to class 2). So overall the model performs well.

A classification model based on a decision tree was built, and optimal parameters were tuned. The highest `f1_weighted` metric value achieved on the test set for this model was 0.89, which is substantially worse than for the previously considered models. The model classified class 0 instances without error, but performed relatively poorly at identifying class 1 instances (true positive = 14, false negative = 4). The model can be improved by tuning optimal class weight coefficients, as well as by transitioning to tree ensembles. At the same time, training a tree takes more time than logistic regression.

Thus, logistic regression is chosen as the final model, with regularization parameter `C = 0.06`, automatic class balancing (`class_weight='balanced'`), and the "one-versus-rest" strategy for handling the multiclass setting (`multi_class='ovr'`). This model classified every instance in the test set correctly.

The notebook of the project can be reached via the following link: ["Multiclass classification with imbalanced classes"](https://github.com/ElenaNKn/data_projects/blob/master/project_multiclass_classification/project.ipynb)