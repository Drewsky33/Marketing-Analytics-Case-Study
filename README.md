# Marketing-Analytics-Case-Study

This project is from Danny Ma's Serious SQL course. 

## Scenario
The marketing team has shared a draft of e-mails they wish to send to customers. Creating personalized customer emails is a staple of a lot of digital companies. We have been asked to support the customer analytics team at DVD Rental Co who want us to gather data points required to popular of their first email marketing campaign. 

I'll be using SQL to solve these problems and capture the data needed to ensure a smooth marketing campaign. 

## Example email template


Requirements:
1. For each customer we need to identify the top 2 categories based on their past rental history. These categories will drive what kind of images will be put into the email. 
2. The marketing team has also asked that we include the 3 most popular fils for each customer's top 2 categories. **However,** we can't recommend a film that the customer has seen already. If there isn't 3 films available, marketing is more than happy to show at least 1 film. If the customer doesn't have any film recommendations for either of their categories, then the customer needs to be flagged so they can be excluded from the email marketing campaign. 3. The marketing team also requires that we provide the following insights for each customer:
  - For category 1:
    * How many total films have they watched in their category?
    * How many more films has the customer watched in comparison to the average customer?
    * How does the customer rank in terms of the top x% compared to all other customers in the film category?
  - For category 2:
    * How many total films has the customer watched in this category?
    * What proportion of each customer's total films watched does this count make?
5. Favorite actor recommendations: Include up to 3 other films featuring their favorite actor. 
  - Additionally, films that have been recommended in top 2 categories must not be included in actor recommendations. 
  - If they don't 1 film recommendation, they need to be flagged with a separator actor exclusion flag. 

## Data Exploration





