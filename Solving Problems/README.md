
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


As you can see there is now a percentage row which is the percentage that their 2nd top category makes up of their total viewership. We've now generated the insights for each customer's top two categories. What we need to do next is generate the recommendations based on these insights. 

## Category Recommendations

- **Film counts**: Next we want to generate film counts for each movie in the category and then sort them by most viewed to get the most popular films.

``` sql

-- Generate film counts for each unique movie in each category 
DROP TABLE IF EXISTS film_counts;
CREATE TEMP TABLE film_counts AS 
SELECT DISTINCT
  film_id,
  title,
  category_name,
  COUNT(*) OVER (
    PARTITION BY film_id
  ) AS rental_count
FROM complete_joint_dataset;

-- Observe the new table with the film counts 
SELECT *
FROM film_counts
ORDER BY rental_count DESC
LIMIT 10;

```

**OUTPUT**

<img width="879" alt="image" src="https://user-images.githubusercontent.com/77873198/179089690-a631a750-c799-4865-85b2-bd4962c8be6d.png">


We now have the rental count for each film. So basically, how many time each film has been rented. Additionally, we select the distinct features of the title and the category name. Next, we have to generate a table that with customer's previously watchec films so we don't recommend movies they've already seen. 

- **Category Film exclusion**

``` sql

-- Create a table that has the viewing history of each customer 

DROP TABLE IF EXISTS category_film_exclusions;
CREATE TEMP TABLE category_film_exclusions AS 
SELECT DISTINCT
  customer_id,
  film_id
FROM complete_joint_dataset;

-- Observe the table 
SELECT *
FROM category_film_exclusions
LIMIT 10;

```


**OUTPUT**

<img width="320" alt="image" src="https://user-images.githubusercontent.com/77873198/179090800-b8f17e25-fec1-48d1-ab1c-3e344ee4cdb5.png">

- **Final category recommendations**:

Now that we have a table that will serve as the filter, we will use an anti join on the film exclusions table for the top two categories in teh top categories table. Then we'll use a window function to select the top 3 films for each of the top 2 categories per customer. Need to keep the category rank column in the final output to easily identify each customer's top choices of category. 

``` sql

DROP TABLE IF EXISTS category_recommendations;
CREATE TEMP TABLE category_recommendations AS
WITH ranked_films_cte AS (
  SELECT
    top_categories.customer_id,
    top_categories.category_name,
    top_categories.category_rank,
    film_counts.film_id,
    film_counts.title,
    film_counts.rental_count,
    DENSE_RANK() OVER (
      PARTITION BY
        top_categories.customer_id,
        top_categories.category_rank
      ORDER BY
        film_counts.rental_count DESC,
        film_counts.title
    ) AS reco_rank
  FROM top_categories
  INNER JOIN film_counts
    ON top_categories.category_name = film_counts.category_name
  -- This is a tricky anti-join where we need to "join" on 2 different tables!
  WHERE NOT EXISTS (
    SELECT 1
    FROM category_film_exclusions
    WHERE
      category_film_exclusions.customer_id = top_categories.customer_id AND
      category_film_exclusions.film_id = film_counts.film_id
  )
)
SELECT * FROM ranked_films_cte
WHERE reco_rank <= 3;


-- Check the recommendation ranks for each customer by category
SELECT *
FROM category_recommendations
WHERE customer_id = 1
ORDER BY category_rank, reco_rank;

```

**OUTPUT**

<img width="1115" alt="image" src="https://user-images.githubusercontent.com/77873198/179101268-63b50805-e04d-45b4-8053-2c991dd7c9c8.png">


So we now have the top 3 rated movies by category based on the rental count. Our next task is to focus on the actor insights. 

## Actor Insights

- **Actor Joint Table**: For this analysis, we'll need to create a new base table. The reason for this is that we need information from `dvd_rentals.film_actor` and `dvd_rentals.actor` tables to extract the information required for our final output. We ultimately want to choose 3 of the top 3 films for each customer's favorite actor that they haven't watched.

``` sql

-- Create a base table to generate actor insights and recommendations
DROP TABLE IF EXISTS actor_joint_dataset;
CREATE TEMP TABLE actor_joint_dataset AS 
SELECT
  rental.customer_id,
  rental.rental_id,
  rental.rental_date,
  film.film_id,
  film.title,
  actor.actor_id,
  actor.first_name,
  actor.last_name
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_actor 
  ON film.film_id = film_actor.film_id
INNER JOIN dvd_rentals.actor 
  ON film_actor.actor_id = actor.actor_id;
  
-- Observe the new table
SELECT *
FROM actor_joint_dataset
WHERE customer_id = 1
LIMIT 10;

```

**OUTPUT**

<img width="1001" alt="image" src="https://user-images.githubusercontent.com/77873198/179114567-ce411b0b-418f-468e-8746-ec72e2f4cafb.png">


We've now got a table with each customer id, film names, and the names of all the actors that participated in each film. We also have the id information for each film and actor for analysis later. 

- **Top Actor Counts**

``` sql

-- Generate top actor counts table 
DROP TABLE IF EXISTS top_actor_counts;
CREATE TEMP TABLE top_actor_counts AS 
WITH actor_counts AS (
  SELECT
    customer_id,
    actor_id,
    first_name,
    last_name,
    COUNT(*) AS rental_count,
    MAX(rental_date) AS latest_rental_date
  FROM actor_joint_dataset
  GROUP BY 
    customer_id,
    actor_id,
    first_name,
    last_name
),

ranked_actor_counts AS (
  SELECT
    actor_counts.*,
    DENSE_RANK() OVER (
      PARTITION BY customer_id
      ORDER BY 
        rental_count DESC,
        latest_rental_date DESC,
        first_name,
        last_name
    ) AS actor_rank
  FROM actor_counts
) 
SELECT
  customer_id,
  actor_id,
  first_name,
  last_name,
  rental_count
FROM ranked_actor_counts
WHERE actor_rank = 1;

-- Observe the new table 

SELECT *
FROM top_actor_counts
LIMIT 10;

```

**OUTPUT**


<img width="872" alt="image" src="https://user-images.githubusercontent.com/77873198/179116542-e520152f-1eb8-4a61-a887-0cd0ee200673.png">

We know have the top ranked actor for each customer based on movies viewed with that actor. This table will be satisfactory in making our recommendations. 

## Actor Recommendations

- **Actor Film Counts**: The first thing we need to do is build an aggregated total rentals table. We need to retain the `actor_id` and `film_id` in order to join with the `top_actor_counts` tabe we created before. 

``` sql

-- Generate actor film counts table
DROP TABLE IF EXISTS actor_film_counts;
CREATE TEMP TABLE actor_film_counts AS
WITH film_counts AS (
  SELECT
    film_id,
    COUNT(DISTINCT rental_id) AS rental_count
  FROM actor_joint_dataset
  GROUP BY film_id
)
SELECT DISTINCT
  actor_joint_dataset.film_id,
  actor_joint_dataset.actor_id,
  -- why do we keep the title here? can you figure out why?
  actor_joint_dataset.title,
  film_counts.rental_count
FROM actor_joint_dataset
LEFT JOIN film_counts
  ON actor_joint_dataset.film_id = film_counts.film_id;

-- Look at the new table
SELECT *
FROM actor_film_counts
LIMIT 10;

```
**OUTPUT**

<img width="747" alt="image" src="https://user-images.githubusercontent.com/77873198/179118055-ddae4e8d-90f1-427a-9c0f-bc8d21c9f147.png">


We now have a table with the film_id, the id for each actor in the film, the movie title, and the number of times that title has been rented. 

- **Actor Film Exclusions**: We need to exclude films that we've already seen from our final table in order to make relevant recommendations. We also need to perform a UNION that will exclude category recommendations we have given to customers. We don't want customers to receive recommendations for the same film in one e-mail, which is why we make this recommendation. 


``` sql

-- Generate actor film exclusion table 
DROP TABLE IF EXISTS actor_film_exclusions;
CREATE TEMP TABLE actor_film_exclusions AS 

(
  SELECT DISTINCT
    customer_id,
    film_id
  FROM complete_joint_dataset
)
-- Combine with previously wathched and recommended films 

UNION 
(
  SELECT DISTINCT
    customer_id,
    film_id
  FROM category_recommendations
);

-- Look at the new table 
SELECT *
FROM actor_film_exclusions
LIMIT 10;

```

**OUTPUT**

<img width="1158" alt="image" src="https://user-images.githubusercontent.com/77873198/179121472-7010c2c8-ed96-4fe6-95a4-7260739ede4e.png">


Great, now we can move to making the final recommendations for each actor. 

- **Final Actor Recommendations**

``` sql

-- Generate final actor recomendations table 

DROP TABLE IF EXISTS actor_recommendations;
CREATE TEMP TABLE actor_recommendations AS 
WITH ranked_actor_films_cte AS (
  SELECT
    top_actor_counts.customer_id,
    top_actor_counts.first_name,
    top_actor_counts.last_name,
    top_actor_counts.rental_count,
    actor_film_counts.title,
    actor_film_counts.film_id,
    actor_film_counts.actor_id,
    DENSE_RANK() OVER (
      PARTITION BY
        top_actor_counts.customer_id
      ORDER BY 
        actor_film_counts.rental_count DESC,
        actor_film_counts.title
    ) AS reco_rank
  FROM top_actor_counts
  INNER JOIN actor_film_counts
    ON top_actor_counts.actor_id = actor_film_counts.actor_id
    
  -- Anti join to filter 
  
  WHERE NOT EXISTS (
    SELECT 1
    FROM actor_film_exclusions
    WHERE
      actor_film_exclusions.customer_id = top_actor_counts.customer_id AND 
      actor_film_exclusions.film_id = actor_film_counts.film_id
  )
)

-- Select all information for each actor in the top the recommended rank in terms of film_counts
SELECT *
FROM ranked_actor_films_cte
WHERE reco_rank <= 3;

-- Observe new table
SELECT *
FROM actor_recommendations
ORDER BY 
  customer_id,
  reco_rank
LIMIT 15;


```

**OUTPUT**

<img width="1158" alt="image" src="https://user-images.githubusercontent.com/77873198/179123925-0bf44569-d903-4f4d-b39c-b44eeafc05dc.png">


We now have all of the insights needed to solve our problmem and generate a table that can be used by the marketing team. 


