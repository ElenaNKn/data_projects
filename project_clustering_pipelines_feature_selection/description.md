# Feature Selection. Data clusterization

## Project Description

This project explored several techniques for improving regression model performance (using a decision tree as an example):
- Log transformation of the target variable;
- Introducing polynomial features;
- Feature standardization and selection;
- Encoding categorical features.

In addition, the project demonstrated the potential of clustering for data analysis and for deriving statistical characteristics of data points within clusters.

The work was carried out using a dataset containing information on New York City taxi trips (https://www.dropbox.com/s/glmbcyopi24m2ni/final_project_data.csv?dl=1).

## Project Execution Stages

1. **Data preprocessing**: handling missing values and extracting individual features from composite feature sets.
2. **Exploratory data analysis**: checking for missing values and outliers, and constructing and analyzing feature distribution characteristics.
3. **Building a baseline decision tree model**: evaluating model performance on the test dataset and identifying the most important features according to the baseline model.
4. **Applying various approaches to improve baseline model performance**: evaluating quality on the test dataset.
5. **Clustering** the data and analyzing cluster characteristics.

## Results and Conclusions

Exploratory data analysis revealed that the highest number of taxi orders occurs in the spring, with peak demand in the late afternoon and evening hours (after work and into the evening). Outliers were also detected in the data (remote pickup/dropoff locations).

A baseline decision tree model was built, achieving an R2-score score of 0.5 on the test set. Feature importance was visualized based on this model, showing that pickup and dropoff longitudes have the greatest weight in predicting trip fare.

Several approaches were applied to improve the baseline model's performance:
- Log transformation of the target variable
- Introducing polynomial features
- Feature standardization and selection
- Encoding categorical features

All of these approaches helped improve the baseline model. The most effective was introducing polynomial features, which boosted the R2-score to 0.71.

On the dataset cleaned of outliers (excluding remote dropoff locations), clustering was performed on passenger dropoff points. The cluster with the highest average trip fare was identified. This cluster was then further clustered into 3 subclusters, and the feasibility of computing their geographic centroids was demonstrated. This illustrates the potential application of clustering — for example, in identifying potentially profitable customers based on the destination area of their trips.

The project notebook is available at the next below: ["Feature Selection. Data clusterization"](https://github.com/ElenaNKn/data_projects/blob/master/project_clustering_pipelines_feature_selection/project.ipynb)