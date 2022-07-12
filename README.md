# Marketing-Analytics-Case-Study

This project is from the marketing analytics case study within Danny Ma's Serious SQL course. 

## Scenario
The marketing team has shared a draft of e-mails they wish to send to customers. Creating personalized customer emails is a staple of a lot of digital companies. We have been asked to support the customer analytics team at DVD Rental Co who want us to gather data points required to popular of their first email marketing campaign. 

I'll be using SQL to solve these problems and capture the data needed to ensure a smooth marketing campaign. 

## Problem

<img width="411" alt="image" src="https://user-images.githubusercontent.com/77873198/178606207-a8e67712-76e6-4b45-a5c1-92194c5d1f1e.png">
[Source: Serious SQL by Danny Ma](https://www.datawithdanny.com/)

Requirements:
1. For each customer we need to identify the top 2 categories based on their past rental history. These categories will drive what kind of images will be put into the email. 
2. The marketing team has also asked that we include the 3 most popular fils for each customer's top 2 categories. **However,** we can't recommend a film that the customer has seen already. If there isn't 3 films available, marketing is more than happy to show at least 1 film. If the customer doesn't have any film recommendations for either of their categories, then the customer needs to be flagged so they can be excluded from the email marketing campaign.

### Category Insights
3. The marketing team also requires that we provide the following insights for each customer:
  - For category 1:
    * How many total films have they watched in their category?
    * How many more films has the customer watched in comparison to the average customer?
    * How does the customer rank in terms of the top x% compared to all other customers in the film category?
  - For category 2:
    * How many total films has the customer watched in this category?
    * What proportion of each customer's total films watched does this count make?


### Customer Insights

5. Favorite actor recommendations: Include up to 3 other films featuring their favorite actor. 
  - Additionally, films that have been recommended in top 2 categories must not be included in actor recommendations. 
  - If they don't 1 film recommendation, they need to be flagged with a separator actor exclusion flag. 

## [Data Exploration](https://github.com/Drewsky33/Marketing-Analytics-Case-Study/tree/main/Data%20Exploration)

<img width="775" alt="image" src="https://user-images.githubusercontent.com/77873198/175354413-84552686-57c5-4cc0-95cc-47f69ed85f48.png">

[SOURCE: Serious SQL](https://www.datawithdanny.com/)

The data exploration part of the analysis can be found [here](https://github.com/Drewsky33/Marketing-Analytics-Case-Study/tree/main/Data%20Exploration). In this part of the project I looked at the tables indiviudally from the chart above, observed their links, and paths I need to take to secure vital pieces of data that will be used in the e-mail marketing campaign. 


## [Data Analysis- What tables do we need?](https://github.com/Drewsky33/Marketing-Analytics-Case-Study/blob/main/Data%20Analysis/README.md)

## [Join Implementation](https://github.com/Drewsky33/Marketing-Analytics-Case-Study/tree/main/Implementing%20Joins)

## [Solving Our Problem]()

## [Report]()
