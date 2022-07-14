
# Solving Business Problems

## Planning
So at this point we have a basic idea of what we want to do. 

1. We want to gather the information into a complete dataset with the relevant tables to perform our transformations. 
2. We want to look at the category insights. 
3. We want to look at the insights for each actor. 

1. Customer Insights to-do list:
- Create a base dataset by joining the relevant tables.
- Calculate the customer rentals by category. This category wil be called `category_counts`.
- Aggregate all customer total films watched. This category will be called `total_counts`
- Identify the top categories for each customer. This will be called `top_categories`.
- Calculate each categories aggregated average rental count. `average_category_count`
- Calculate the percentile metric for each customer's top category film_count. `top_category_percentile`
- Generate first top category insights table using all previously generated tables. `top_category_insights`
- Generate 2nd category insights. `second_category_insights`

2. Category Recommendations to-do list:
- Generate a summarised film count table with the category included. We will use this to rank film by popularity. `film_counts`
- Create a previously watched films insight that will exclude the top 2 categories for each customer. `category_film_exclusions`
- Perform an anti join from the relevant category films  in the exclusions table. Then use window functions to keep the top 3 from each category by popularity. Finally, we will split out the recommendations by category ranking. `category_recommendations

3. Actor Insights to-do list:
- Create a new base dataset which is based around the insights needed for actors. `actor_join_table`
- ID the top actor and their the rental count for each actor for each customer based off the ranked rental counts. `top_actor_counts`

4. Actor Recommendations to-do list:
- Generate total actor rental counts to use for film popularity later on. `actor_film_counts`
- Create a film exclusions table which includeds previously watched films, similar to hat we did for the category recommendations. This time we'll add films which were previously recommended. `actor_film_exclusions`
- Apply the anti join technique and use window functions to identify the top 3 film recommendations for each customer. `actor_recommendations`

## Category Insights

- **Create base dataset**

``` sql

DROP TABLE IF EXISTS complete_joint_dataset;
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  -- also included rental_date for sorting purposes
  rental.rental_date
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_category
  ON film.film_id = film_category.film_id
INNER JOIN dvd_rentals.category
  ON film_category.category_id = category.category_id;

-- Sample the dataset
SELECT *
FROM complete_joint_dataset
LIMIT 10;

```

**OUTPUT**

<img width="1158" alt="image" src="https://user-images.githubusercontent.com/77873198/179047395-f73df06c-b2f0-4318-90a6-19c942d2d1a8.png">


- **Category Counts**- Use the `complete_joint_dataset` to aggregate data and generate the `rental_count` and `rental_date` to be used later. 

``` sql

-- Create category counts 

DROP TABLE IF EXISTS category_counts;
CREATE TEMP TABLE category_counts AS 
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  
  -- latest_rental_date is a tiebreaker used downstream
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset
GROUP BY
  customer_id,
  category_name;


-- Look at the category_counts table 
SELECT *
FROM category_counts
WHERE customer_id = 1
ORDER BY
  rental_count DESC,
  latest_rental_date DESC;
  
```

**OUTPUT**

<img width="1159" alt="image" src="https://user-images.githubusercontent.com/77873198/179048728-74222ee0-4ed0-4c68-85cb-7b1cc406667f.png">


- **Total counts**- aggregate all customers total films watched. 

``` sql

-- Aggregate films into new table called total_counts

DROP TABLE IF EXISTS total_counts;
CREATE TEMP TABLE total_counts AS 
SELECT
  customer_id,
  SUM(rental_count) AS total_count
FROM category_counts
GROUP BY
  customer_id;
  
-- Look at the new table 

SELECT *
FROM total_counts
ORDER BY total_count DESC
LIMIT 5;

```

**OUTPUT**

<img width="313" alt="image" src="https://user-images.githubusercontent.com/77873198/179049461-b9f6a573-b455-452c-8981-3ec2f8eb6397.png">


Next, we'll move on to generating the top categories. 

- **Top Categories**: Let's generate a rank of the categories for each customer. We will split ties using the latest_rental_data value we created in the `category_counts` table. 

``` sql

-- Generate top categories table 
DROP TABLE IF EXISTS top_categories;
CREATE TEMP TABLE top_categories AS 
WITH ranked_cte AS (
  SELECT
    customer_id,
    category_name,
    rental_count,
    DENSE_RANK() OVER (
      PARTITION BY customer_id
      ORDER BY 
        rental_count DESC,
        latest_rental_date DESC,
        category_name
    ) AS category_rank
  FROM category_counts
) 

-- Take out the top category ranks 

SELECT *
FROM ranked_cte 
WHERE category_rank <= 2;

-- Observe the query
SELECT *
FROM top_categories
LIMIT 6;

```

**OUTPUT**

<img width="865" alt="image" src="https://user-images.githubusercontent.com/77873198/179066862-f8cea06c-bdfa-4cc8-b44f-c1b68f5e86d4.png">

We now have the top 2 categories for the first 3 customer id's. Next we need to get the average category count so that we can make a comparison for our top_category_percentile column that we will create downstream. 

- **Average per category**

``` sql

-- Generate the average category count for comparison later 

DROP TABLE IF EXISTS average_category_count;
CREATE TEMP TABLE average_category_count AS 
SELECT
  category_name,
  -- Round down to the nearest integer
  FLOOR(AVG(rental_count)) AS category_average
FROM category_counts
GROUP BY category_name;

-- Look at the new table and averages 

SELECT *
FROM average_category_count
ORDER BY 
  category_average DESC,
  category_name;

```

**OUTPUT**

<img width="448" alt="image" src="https://user-images.githubusercontent.com/77873198/179068636-0a67ff05-6260-4186-b6f7-a5e45b8a9121.png">

Next, we're going to make a comparison with the average category counts from above to determine which percentile the customer falls in based on their viewership of the category. 

- **Top Category Percentile**

``` sql

-- Create temp table to compare each customer's rental count for their top categories by comparing it to the average category count 
DROP TABLE IF EXISTS top_category_percentile;
CREATE TEMP TABLE top_category_percentile AS 
WITH calculated_cte AS (
SELECT
  top_categories.customer_id,
  top_categories.category_name AS top_category_name,
  top_categories.rental_count,
  category_counts.category_name,
  top_categories.category_rank,
  -- Create a percentage based on the category rental count 
  PERCENT_RANK() OVER (
    PARTITION BY category_counts.category_name
    ORDER BY category_counts.rental_count DESC 
  ) AS raw_percentile_value
FROM category_counts
LEFT JOIN top_categories
  ON category_counts.customer_id = top_categories.customer_id
)

-- Select necessary information, we want to know the customer id, the category name of their top, the rental count, the category rank by customer, and percentile they fall in 
SELECT
  customer_id,
  category_name,
  rental_count, 
  category_rank,
  CASE
    WHEN ROUND(100* raw_percentile_value) = 0 THEN 1
    ELSE ROUND(100 * raw_percentile_value)
  END AS percentile
FROM calculated_cte
WHERE 
  category_rank = 1
  AND top_category_name = category_name;
  
-- View the selection 

SELECT *
FROM top_category_percentile
LIMIT 10;

```

**OUTPUT**

<img width="1068" alt="image" src="https://user-images.githubusercontent.com/77873198/179071209-56dd5ca8-ecab-4f99-b0d2-6d6da1e96161.png">

We've created a lot of tables that we now need to combine into a single category insights table. The tables we'll be using are the `top_category_percentile` serving as the base table (because most of the information we need is in this table) we'll left join in order to preserve this information with our average table for the final average comparison. 

- **1st Category Insights**

``` sql

DROP TABLE IF EXISTS first_category_insights;
CREATE TEMP TABLE first_category_insights AS 

-- alias the top_cateogry_percentile as base 
SELECT
  base.customer_id,
  base.category_name, 
  base.rental_count,
  average.category_average,
  base.rental_count - average.category_average AS average_comparison
FROM top_category_percentile AS base
LEFT JOIN average_category_count AS average
  ON base.category_name = average.category_name;
  
-- View the table and average comparison 

SELECT *
FROM first_category_insights
LIMIT 10;


```

**OUTPUT**

<img width="1160" alt="image" src="https://user-images.githubusercontent.com/77873198/179073766-176337d3-78d5-4e90-8793-df04514d731f.png">


As you can see we now have a table that has the rental count for the category for each customer, the average number of movies viewed for that category, and the difference between them which places each customer into a percentile based on viewership. Next, we will have to calculate the ratio of movies viewed in the second top category when compared with the total movies viewed for each customer. 

- **Second Insight**: (2nd top category rental count / total movies viewed for customer) * 100

``` sql

-- Generate a second table for 2nd category insights 

DROP TABLE IF EXISTS second_category_insights;
CREATE TEMP TABLE second_category_insights AS 
SELECT
  base.customer_id,
  base.category_name,
  base.rental_count,
  -- Cast as a numeric to avoid INTEGER floor division
  ROUND(
    100 * base.rental_count::NUMERIC / total.total_count
  ) AS percentage_total_viewing
FROM top_categories AS base
LEFT JOIN total_counts AS total
  ON base.customer_id = total.customer_id
WHERE category_rank = 2;

-- Check the second category insights
SELECT * 
FROM second_category_insights
LIMIT 10;

```

**OUTPUT**:

<img width="919" alt="image" src="https://user-images.githubusercontent.com/77873198/179088524-76832c10-d8c0-4247-b9aa-9c84cfcdbf9f.png">


As you can see there is now a percentage row which is the percentage that their 2nd top category makes up of their total viewership. 
