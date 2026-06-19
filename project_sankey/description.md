# User Flow Sankey Diagram 

## Project Description

This project is based on the dataset ['The Puffy "Lost Sleep" Challenge'](https://www.kaggle.com/competitions/the-puffy-lost-sleepchallenge/data) hosted by Puffy - an American producer of luxury mattresses and bedding. 

The dataset includes 2 tables:
- `events` with user actions at https://puffy.com site 
- `tags` with user ids and tags indicating whether the user returned a product or not

Participants of the Kaggle competition were asked to answer the question: 'Why do some customers complete their purchase with confidence, while others hesitate or ultimately return their product?' The approach suggested by the host is to build a classification model to predict the probability that a customer will return their order.

This project takes a different angle. It's proposed to conduct a thorough EDA (Exploratory Data Analysis) and build a User Flow Sankey Diagram specifically for customers who returned their product, to identify at which stage the return decision is made.

## Project Execution Stages

1. Load the dataset into a locally deployed PostgreSQL database. Extract the necessary data from the database.

2. Identify events that are significant to the problem under investigation. Transform the data to build a user flow map.

3. Build a user flow map for users who ordered a product and returned it. Identify steps in the map that could influence the user's decision to return the product.

4. Prepare recommendations for reducing the share of returned products and suggestions for further research.

## Results and Conclusions

An HTML file with a user flow map for users who have returned products can be downloaded via the following link ["UserFlow"](https://raw.githubusercontent.com/ElenaNKn/data_projects/refs/heads/master/project_sankey/UserFlow_returned_product.html) (it's necessary to right-click on the link and select "Save link as...")

Analysis of the user flow map revealed the following:

1. 4.31% of users who ordered and later returned a product had visited the return policy page before making their purchase. It's suggested to track these users (those who viewed the return policy before adding the item to the cart) and either show them an extra modal that confirms their intent to buy, or reach them out with personalized post-purchase messaging aimed at reducing the likelihood of a return.
2. 25.86% of users with returned products viewed informational content on the site between placing their order and initiating a return. A closer look at what they viewed shows that most checked the store's contact page (further investigation here would require data from the customer support team).
3. 9.48% of users with returned products viewed information about the warranty, trial-period and reviews. It can be suggested that they weren't fully satisfied with their choice. However, only 3 customers (2.6% of all returns) visited the page that explains that the mattress needs about 30 days to break in and adapt to the user.

Overall, these findings point to a need to rethink how the information on the site is currently presented and how effective communication with users is. It should be more clear and visual shown that the mattress requires an adjustment period. Right now, it seems that many users are returning the product without ever having seen that specific information.

The notebook of the project can be reached via the following link: ["User Flow Sankey Diagram project"](https://github.com/ElenaNKn/data_projects/blob/master/project_sankey/project_sankey_diagram.ipynb)