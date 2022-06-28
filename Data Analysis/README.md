
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

I'll start by visualizing what the ideal output will look like and then I'll tell why each part of the output is important:

<img width="348" alt="image" src="https://user-images.githubusercontent.com/77873198/175645562-4ea5e87b-a2f2-4f03-b02b-a1785b34c151.png">

The fields in the table will bue used to generate the following script for the marketing campaign:

'You've watched `rental_count` films feature `actor_name`! Here are some other films `first_name` stars in that might interest you!\

From that expected script we can ssee that we'll need to populate with the rental count, the actors name, and their first name. 

### Making Film recommendations
Finally, the last parts of the SQL output table we'll need to generate is the film recommendations for each category and actor. 

We'll have the all of the information about the actor, the customer, but after the `customer_id` column we will need to have the 3 recommendations for both top categories. Finally, after those we'll need the 3 recommendations based on the top actor_count.

### Key columns needed:
**For analysis:**
- `customer_id`: We are going to need the customer_id column to identify the customer. 
- `title`: We will need the title column from the film table. 
- `name`: We need the category name column from the category table so we can identify the top categories. 
- `first_name` & `last_name` columns will be needed to identify the top most popular actor. 

**For joining tables**:
- `inventory_id`: This table links the rental column to the inventory table which provides a link to the film table. 
- `film_id`: This column links us to the film table and the film_category table which has links to the category column **AND** it links us to the film_actor table which has a link to the actor table. 
- `category_id`: The category id column in the film_category table will link us to the category table where we can extract the name. 
- `actor_id`: The film_actor table has a link to the actor table through the actor_id column and this allows us to extract the actor's first and last_name. 


### Join Analysis
Now that we've identified which kinds of columns we need, how the tables link, and have an idea of our expected output. We should analyze the different types of joins and whether or not they meet or needs. First, we need to show our desired final state:

<img width="737" alt="image" src="https://user-images.githubusercontent.com/77873198/176062772-c45927fa-475c-4eec-a216-a9af91ba193b.png">

One more time I define the need columns need to generate the output above:
- `category_name`: The name of the top 2 ranking categories.
- `rental_counts`: How many totlal films have they watched in this category?
- `average_comparison`: How many films more watched in comparison to the average for the DVD Rental CO customer
- `percentile`: How does the customer rank is in relation to the top X% compared to other customers in the category?
- `category_percentage`: What proportion of total films watched does this category make up?

### Reverse Engineering
- First, we'll want to get the base table without all the metrics created. How do I get there? It's perhaps easier to go to just the customer_id, category_name, and rental_count parts of the table which will be instrument in constructing the metrics. 

<img width="288" alt="image" src="https://user-images.githubusercontent.com/77873198/176064449-b23803ad-1cca-42c9-8aa2-b90beab0b9e2.png">

Having an output like above will be nice, but I notice that includes only two unique values for each customer_id. AKA the top categories. However, in order to get to the top 2 categories we need calculate the rental count for all categories and then filter for the top 2. So going back further we want something like this:

<img width="273" alt="image" src="https://user-images.githubusercontent.com/77873198/176064726-b73f04c6-d431-4f72-bf81-c9e2cad59af2.png">


