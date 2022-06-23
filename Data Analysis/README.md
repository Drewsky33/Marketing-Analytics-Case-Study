
# Data Analysis

## Approaching the problem
To have an idea about how to tackle this business problem, we need to visualize what kind of output we want and then slowly work backwards. Essentially, we need to observe what kind of SQL outputs will allow us to match the requirements from the marketing team. 

### Defining the target state
The end-goal of this project will be a SQL script that will allow us to generate a completed email template for each unique customer. Our main goal is to help them populate their emails with the correct and necessary data. 

<img width="636" alt="image" src="https://user-images.githubusercontent.com/77873198/175358806-7f840ec8-42b6-4991-ba41-ae5af719cead.png">

[Image from Serious SQL](https://www.datawithdanny.com/)

**Category Data Points**
The data points we'll want to populate the above ad are:
1. Top ranking category name: `cat_1`
2. Top ranking category customer insight: `insight_cat_1`
3. Top ranking category film recommendations: `cat_1_reco_1`, `cat_1_reco_2`, `cat_1_reco_3`
4. 2nd ranking category name: `cat_2`
5. 2nd ranking category customer insight: `insight_cat_2`
6. 2nd ranking category film_recommendations: `cat_2_reco_1`, `cat_2_reco_2`, `cat_2_reco_3`. 

**Actor Data Points**
7. Top actor name: `actor`
8. Top actor insight: `insight_actor`
9. Actor film_recommendations: `actor_reco_1`, `actor_reco_2`, `actor_reco_3`. 

The finaly output should contain all of the information above in column format. There should be a row for the unique customer_id and have the top 2 categories, the top actor's full name, along with the recommendations and insights we stated we want. 

### Identifying key categories 
We can talk about the insights we want to generate, but can't generate those without first identifying which column name will be useful for this problem. In order to generate the customer insights from the add we need:
- `category_name`: For each of the top two categories
- `rental_count`: How many total films a customer has watched for a specific category
- `average_comparison`: How many films more, on average, did the customer watched compared to the rest of the DVD Rental Co customers.
- `percentile`: How does the customer rank in terms of top 10 X% compared to all the other customers in this film category. 
- `category_percentage`: what proportion of total films watched does the count make?

The e-mail template from before will have the following insight for the top category:
- You've watched{`rental_count`}{`category_name`} films, that's {`average_comparison`} more than the DVD Rental Co average and puts you in the top{`percentile`} of {`category_name`} gurus. 

The second category insight text will be:
- You've watched{`rental_count`}{`category_name`} films making up {`category_percentage`}% of your entire viewing history!


**Table Output Example**

<img width="634" alt="image" src="https://user-images.githubusercontent.com/77873198/175362969-451b64a2-c37a-4982-9eb8-e24f5812e076.png">

### Identifying key information for actors
