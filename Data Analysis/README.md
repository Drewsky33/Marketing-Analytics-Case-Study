
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

So as we can see, we need to aggregate based on the category name. However, we need to get all of the data necessary into 1 table. This is where we have to implement some joins. The two columns that will pave way to the analysis for the rest of the project will start with the two columns:
- `customer_id`: rental table
- `category_name`: category table. 

We have to start at the rentals table because it's the only place where the customer_id is present. We now have our starting and end points for our SQL join journey, but here's what it looks visualized. 

<img width="722" alt="image" src="https://user-images.githubusercontent.com/77873198/176065755-87f3f896-1955-429b-85cc-3a6a062549bd.png">

Image from [Serious SQL](https://www.datawithdanny.com/courses/serious-sql)

The route we will take is: 
 - Part 1: rental -> inventory through inventory_id
 - Part 2: inventory -> film through film_id
 - Part 3: film -> film_category through film_id
 - Part 4: film_category -> category through category_id

### Choosing Join Types
I need to answer the following checklist before choosing the type of join to use:

1. What is the purpose of joining these two tables?
  - We need to generate the record count calculation. This involved the customer_id for each customer and the number of films for each category. So first, I want to inspect a customer_id to see what information shows up. 

``` sql
SELECT *
FROM dvd_rentals.rental
WHERE customer_id = 120;

```
**OUTPUT**:


<img width="1149" alt="image" src="https://user-images.githubusercontent.com/77873198/176066786-09014541-7045-4860-bf06-9001fd41164a.png">


It looks like for each customer_id we have multiple rental_ids which means these are transactions or rentals. An important thing to note is the rentals aren't tracked at the film_id level. We know from the ERD visual before that the film_id information is available in the inventory table. There is a need for a join between the two. So which type of join do we use?
- A left join will return all information from the base table our rental table.
- An inner join will return all of the information that is present between both tables. 

Since we are matching on the inventory_id between tables I need to know the distribution for inventory_id for the `rental` table and the `inventory` table. I also need to know if there is overlap between the inventory_id for each table or if there are any missing values. Essentially we want to know:
  * How many records exist per foreign key value in the left and right table?
  * Is there overlap between this key? Are there missing values between these two tables?

### Generating Hypotheses:
We know that the rental table has every rental for each customer. So it makes some sense that we think there should be a valid inventory_id for each record in the rental table when joining. 

It could also make some sense that records could be rented out by multiple customers at different times since customers will return the DVD when done. 

With this information we are going to generate some hypotheses:
- The number of unique inventory_id records will be equal in both dvd_rentals.rental and dvd_rentals.inventory tables. 
- There will be multiple records per unique inventory_id in the dvd_rentals.rental table.
- There will be multiple inventory_id records per unique film_id value in the dvd_rentals.inventory table. 


**Next** I will try to validate my hypotheses through the use of some queries. 
- Hypothesis 1: The number of unique inventory_id records will be equal in both dvd_rentals.rental and dvd_rentals.inventory tables. 
``` sql
SELECT
  COUNT(DISTINCT inventory_id)
FROM dvd_rentals.rental;

```
**OUTPUT**:

<img width="173" alt="image" src="https://user-images.githubusercontent.com/77873198/176068452-61c50e61-95d6-4999-ae37-5fc4efcc9d73.png">


``` sql
SELECT
  COUNT(DISTINCT inventory_id)
FROM dvd_rentals.inventory;

```

**OUTPUT**

<img width="179" alt="image" src="https://user-images.githubusercontent.com/77873198/176068627-ff8263e9-24f3-42cc-9f52-a2cc83f23392.png">

Hmmm it looks like the inventory table has 1 additional value. This would invalidate the first hypotheses that the two tables are similar. An explanation off the top of my head is that the extra value in the inventory table could be a DVD that hasn't been rented yet. 

- Hypothesis 2: There will be multiple records per unique inventory_id in the dvd_rentals.rental table.

``` sql
-- generate group by counts oon the target column 
WITH counts_base AS (
  SELECT
    inventory_id AS target_column_values,
    COUNT(*) AS row_counts
  FROM dvd_rentals.rental
  GROUP BY target_column_values
)

-- Summarize the group by counts by grouping again on the row_counts from the CTE above 

SELECT
  row_counts,
  COUNT(target_column_values) AS count_of_target_values
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;

```

**OUTPUT**:

<img width="381" alt="image" src="https://user-images.githubusercontent.com/77873198/176069569-bf495fe7-e0c9-4340-ad83-a6703437f404.png">


Based on the output, we can confirm that there are multiple rows per inventory_id in the dvd_rentals.rental table. 

- Hypothesis 3: There will be multiple inventory_id records per unique film_id value in the dvd_rentals.inventory table. 

``` sql
-- Generate group by counts on the film_id column
WITH counts_base AS (
  SELECT
    film_id AS unique_film_id,
    COUNT(DISTINCT inventory_id) AS unique_record_counts
  FROM dvd_rentals.inventory
  GROUP BY unique_film_id
)

-- summarize the group by counts above by grouping again on the row_counts from counts_base cte 
SELECT
  unique_record_counts,
  COUNT(unique_film_id) AS count_film_id
FROM counts_base
GROUP BY unique_record_counts
ORDER BY unique_record_counts;

```

**OUTPUT:**

<img width="463" alt="image" src="https://user-images.githubusercontent.com/77873198/176075752-3b0f7ff2-c021-4ece-881d-f9a3735546f6.png">


Based on the output above, we can confirm that there are multiple unique inventory_id per film_id values in the inventory table. 


### Returning to the other key questions
- How many records exist per inventory_id value in the rental and inventory tables?
**Rental distribution analysis on the `inventory_id` foreign key**

``` sql
-- Generate group by counts on foreign key values 
WITH counts_base AS (
  SELECT
    inventory_id AS foreign_key_values,
    COUNT(*) AS row_counts
  FROM dvd_rentals.rental
  GROUP BY foreign_key_values
)

-- Summarize the group by counts above by grouping again on the row_counts from counts_base cte 
SELECT
  row_counts,
  COUNT(foreign_key_values) AS count_of_foreign_keys
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;

``` 

**OUTPUT**:

<img width="233" alt="image" src="https://user-images.githubusercontent.com/77873198/176077485-3f66ed3b-ad84-4d00-b17c-96d9262dde29.png">




**Inventory distribution analysis on `inventory_id` foreign key

``` sql 
WITH counts_base AS (
  SELECT
    inventory_id AS foreign_key_values,
    COUNT(*) AS row_counts
  FROM dvd_rentals.inventory
  GROUP BY foreign_key_values
)

SELECT
  row_counts,
  COUNT(foreign_key_values) AS count_of_foreign_keys
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;

```
**OUTPUT**:

<img width="236" alt="image" src="https://user-images.githubusercontent.com/77873198/176077526-5af266e9-5c73-4e60-9209-9fde989f0fed.png">

**Analysis**

To summarize, we see that there are different row counts for some foreign key values. We expected this. For example, in the rental table 4 of the foreign key values (inventory_id) only had a single row. There 1,126 inventory_ids who had row counts of at least 2. Which means that we have a 1 to many relationship for inventory_id in the rental table. 

In the case of the inventory table we see that there are 4,581 inventory id's with exactly one row and no other values. Or a one to one relationship. For each inventory_id, there should only be 1 row. Which makes sense when you think about it. The inventory table is supposed to keep track of inventory_ids, duplicates would make that difficult. We can confirm this by running a group by count on inventory_id and ordering by record counts in descendig order. 

``` sql
SELECT
  inventory_id,
  COUNT(*) AS record_counts
FROM dvd_rentals.inventory
GROUP BY inventory_id
ORDER BY record_counts DESC;
```

**OUTPUT**:

<img width="351" alt="image" src="https://user-images.githubusercontent.com/77873198/176078510-db86b45e-4061-42b5-b624-54c3495da510.png">

And indeed every value has a maximum value of 1. It's time to move on to the next part of our analysis.


### How many overlapping and missing unique foreign key values are there between the two tables?
Next, I want to see what's the difference between the foreign keys in each table. To do this, we're going to use an ANTI JOIN to obtain the following information:
- Which foreign keys only exist in the left table?
- Which foreign keys only exist in the right table?

``` sql
SELECT
  COUNT(DISTINCT rental.inventory_id)
FROM dvd_rentals.rental
WHERE NOT EXISTS (
  SELECT inventory_id
  FROM dvd_rentals.inventory
  WHERE rental.inventory_id = inventory.inventory_id
);

```

**OUTPUT**:

<img width="177" alt="image" src="https://user-images.githubusercontent.com/77873198/176079776-67199479-359f-4894-9db8-7976463a7eb9.png">


It looks like no values only exist in the left table rentals. 

``` sql

SELECT
  COUNT(DISTINCT inventory.inventory_id)
FROM dvd_rentals.inventory
WHERE NOT EXISTS (
  SELECT inventory_id
  FROM dvd_rentals.rental
  WHERE rental.inventory_id = inventory.inventory_id
);

```

**OUTPUT**:

<img width="178" alt="image" src="https://user-images.githubusercontent.com/77873198/176080036-ac8b1e6e-c930-466f-ae7e-2cfbc4f09460.png">


It looks like there is only one value that exists in the inventory table that doesn't exist in the rental table. We knew this already. We explained this could be a piece of inventory that hasn't been rented yet. 

Since we've already seen that 0 values in the rentals table do not exist in the inventory table, or simply all of the values that exist in the rentals table also exist in the inventory table we can perform a left-semi join to double check that this is the case. This will give us the count of foreign key values that intersect between the two. 

``` sql

SELECT
  COUNT(DISTINCT rental.inventory_id)
FROM dvd_rentals.rental
WHERE EXISTS (
  SELECT inventory_id
  FROM dvd_rentals.inventory
  WHERE rental.inventory_id = inventory.inventory_id
);

```

**OUTPUT**


<img width="177" alt="image" src="https://user-images.githubusercontent.com/77873198/176080714-b259d753-e1c8-4105-a5b5-3d29aad161e9.png">

We've seen the output before when were counting the distinct number of values in the rental table. This means the tables have a lot of overlap and we can perform an inner join. 



