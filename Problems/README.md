
## Solving Business Questions

We have our base table that we'll use to start solving the problems so that we can populate our e-mail template with the correct information for each customer. 

``` sql

DROP TABLE IF EXISTS complete_joint_dataset;
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  rental.rental_date,
  category.name AS category_name
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_category
  ON film.film_id = film_category.film_id
INNER JOIN dvd_rentals.category
  ON film_category.category_id = category.category_id;

SELECT * FROM complete_joint_dataset limit 10;

```

**OUTPUT**


<img width="1159" alt="image" src="https://user-images.githubusercontent.com/77873198/176781668-dd28175d-1b27-4fbf-9cd7-d55305df9b65.png">

We're going to use this to work towards the insights that we must provide for each customer. 

### Revisiting the information we want
- `category_name`: We want the top two categories by rank for each customer.
- `rental_count`: We want the number of films rented for each category
- `average_comparison`: How many films has the customer watched in comparison to the average DVD Rental Co customer?
- `percentile`: A ranking for where the customer fall in the topX% compared to others who view this film category. 
- `category_percentage`: What proportion of total films wathced does this category make up?

We need to generate these fields using the base information we have. 

Some things to note. Our `average_comparison`, `percentile`, and `category_percentage` values are all dependent on the the category_counts that we need to generate. Let's calculate the record counts by aggregating by customer_id and the film_category just to see how our output will look.

``` sql
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_counts
FROM complete_joint_dataset
WHERE customer_id IN (1, 2, 3)
GROUP BY
  customer_id,
  category_name
ORDER BY
  customer_id,
  rental_counts DESC;
  
```

**OUTPUT**


<img width="643" alt="image" src="https://user-images.githubusercontent.com/77873198/176788287-b26f5d79-d45d-4fae-b4fb-f54f49bb7983.png">

Hmm it looks like we have ties. We need to find a way to break the tie between these categories so that we can properly populate a the email template. 

### Dealing with ties 
In the case that we have categories with the same record counts we need to break the tie. What we are going to do is include the rental_date column from our base table and we are going to take the max of that column to signify the most recent rental. 

``` sql

SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_counts,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset
WHERE customer_id IN (1, 2, 3)
GROUP BY
  customer_id,
  category_name
ORDER BY
  customer_id,
  rental_counts DESC
  latest_rental_date DESC;

```

**OUTPUT**

<img width="942" alt="image" src="https://user-images.githubusercontent.com/77873198/176789429-fb8aa8ac-5121-4739-ab4d-cacc367a97df.png">


This metric allows us to take into account the customer's most recent purchase history when breaking a tie. 

## Calculating Averages for the top two categories
We now have all of the information needed to identify the top two categories per customer. However, we need to be careful in generating this output as we can lose some data if we try and aggregate before performing the calculation. What we need to do is calculate our percentages before we isolate the top 2 categories for each customer. **Remember**, we need to calculate the percentile in comparison to other customers, the percentage of the category on viewership, and the average comparison. 

### Calculating customer rental count
First, we should generate the customer rental count for each category on the whole dataset. However, let's make sure it works by first aggregating for 1 customer.

``` sql

DROP TABLE IF EXISTS category_rental_counts;
CREATE TEMP TABLE category_rental_counts AS 
SELECT
  customer_id, 
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset
GROUP BY
  customer_id,
  category_name;

-- Preview one customer and sort rental count
SELECT *
FROM category_rental_counts
WHERE customer_id = 1
ORDER BY 
  rental_count DESC;
```

**OUTPUT**

<img width="933" alt="image" src="https://user-images.githubusercontent.com/77873198/176791484-5e2a644a-d755-4c79-9cb9-e0ea568b8441.png">

It looks fine, what we'll do next is to generate the total rentals per customer that will be involved in the calculation for the `category_percentage`.

``` sql

DROP TABLE IF EXISTS customer_total_rentals;
CREATE TEMP TABLE customer_total_rentals AS 
SELECT
  customer_id,
  SUM(rental_count) AS total_rental_count
FROM category_rental_counts
GROUP BY customer_id;

-- Check the output for the first 5 customer_id values
SELECT *
FROM customer_total_rentals
WHERE customer_id <= 5
ORDER BY customer_id;

```

**OUTPUT**

<img width="371" alt="image" src="https://user-images.githubusercontent.com/77873198/176798039-9695dfaf-b27c-4867-9846-0236a04e7c72.png">


We now have the total_rental_count by customer. We want to know the average category rental counts now. 

### Calculating the Average Category Rental Counts

``` sql
DROP TABLE IF EXISTS average_category_rental_counts;
CREATE TEMP TABLE average_category_rental_counts AS 
SELECT
  category_name,
  AVG(rental_count) AS avg_rental_count
FROM category_rental_counts
GROUP BY
  category_name;
  
-- Produce the table in avg_rental_count DESC order

SELECT *
FROM average_category_rental_counts
ORDER BY 
  avg_rental_count DESC;
```

**OUTPUT**

<img width="457" alt="image" src="https://user-images.githubusercontent.com/77873198/176798471-16b95047-eea2-4761-a4e5-3c047e5e6d2c.png">


Now we have the rental_count averages for each category. However, you can't watch a decimal more of a movie so we need to get the floor values for the calculations. We can do this by updating the table. 

``` sql

UPDATE average_category_rental_counts
SET avg_rental_count = FLOOR(avg_rental_count)
RETURNING * ;

```

**OUTPUT**

<img width="412" alt="image" src="https://user-images.githubusercontent.com/77873198/176799459-db354507-3dd5-4a99-a0d1-89664c699a4d.png">


### Calculating the Percentile column
For this column we need to calculate how the customer ranks in terms of the top X% compared to other customers in the category. In order to get this insight we need to calculate the percentile value for each customer by comparing their result for films watched in a particular category with other customer records in that category. We'll also need to reverse the percentage as we say we are in the top 1 or 2% in the ad. Additionally, if we want to produce the insight:

![image](https://user-images.githubusercontent.com/77873198/176803897-63933c8e-c980-4ec3-85ba-a75c8d8b61d1.png)

We will need to avoid getting a 0% so will use the CEILING FUNCTION to take the upper integer for each percentile metric. 


``` sql

SELECT
  customer_id,
  category_name,
  rental_count,
  -- use CEILING to round up to nearest integer
  CEILING(
    100 * PERCENT_RANK() OVER(
      PARTITION BY category_name
      ORDER BY rental_count DESC
    )
  ) AS percentile
FROM category_rental_counts
ORDER BY customer_id, rental_count DESC
LIMIT 2;

```

**OUTPUT**

<img width="813" alt="image" src="https://user-images.githubusercontent.com/77873198/176804217-36a6c0e2-07de-42ea-8b45-79f900a0c214.png">


The output above will be readable for each customer. 

Now, we'll push this transformation into another temp table and we'll remove the rental_count column since it's already present in the category_rental_counts table

``` sql

DROP TABLE IF EXISTS customer_category_percentiles;
CREATE TEMP TABLE customer_category_percentiles AS
SELECT
  customer_id,
  category_name,
  -- Use ceiling to round up
  CEILING (
    100 * PERCENT_RANK() OVER(
      PARTITION BY category_name
      ORDER BY rental_count DESC
    )
  ) AS percentile
FROM category_rental_counts;

-- inspect the table 

SELECT *
FROM customer_category_percentiles
ORDER BY customer_id, percentile
LIMIT 2;

```

**OUTPUT**

<img width="560" alt="image" src="https://user-images.githubusercontent.com/77873198/176804633-f996fb69-b641-4b47-85b3-23993f044fa4.png">


We have the kind of output we want for each customer now. In total we now have 4 temp tables that we've generated. 3 for the metrics we were looking to find and one for the rental counts. Now we will join them. Our tables are:
- `category_rental_counts`
- `customer_total_rental_counts`
- `average_category_rental_counts
- `customer_category_percentiles`



### Implementing the join

``` sql

DROP TABLE IF EXISTS customer_category_joint_table;
CREATE TEMP TABLE customer_category_joint_table AS
SELECT
  t1.customer_id,
  t1.category_name,
  t1.rental_count,
  t2.total_rental_count,
  t3.avg_rental_count,
  t4.percentile
FROM category_rental_counts AS t1
INNER JOIN customer_total_rentals AS t2
  ON t1.customer_id = t2.customer_id
INNER JOIN average_category_rental_counts AS t3
  ON t1.category_name = t3.category_name
INNER JOIN customer_category_percentiles AS t4
  ON t1.customer_id = t4.customer_id
  AND t1.category_name = t4.category_name;

-- Query the joined table 

SELECT *
FROM customer_category_joint_table
WHERE customer_id = 1
ORDER BY percentile;

```

**OUTPUT**

<img width="657" alt="image" src="https://user-images.githubusercontent.com/77873198/176806632-279490fd-2a57-4bfc-a3b6-a34f917f483f.png">


We are almost to the final output of our table. The last thing we need to add in is:

- `average_comparison`: How many more films has the customer watched compared to the average DVD Rental Co customer?
- `category_percentage`: What proportion of each customer’s total films watched does this count make?

``` sql

-- Adding the calculations

DROP TABLE IF EXISTS customer_category_joint_table;
CREATE TEMP TABLE customer_category_joint_table AS 
SELECT
  t1.customer_id, 
  t1.category_name,
  t1.rental_count,
  t1.latest_rental_date,
  t2.total_rental_count,
  t3.avg_rental_count,
  t4.percentile,
  t1.rental_count - t3.avg_rental_count AS average_comparison,
  ROUND(100 * t1.rental_count / t2.total_rental_count) AS category_percentage
FROM category_rental_counts AS t1 
INNER JOIN customer_total_rentals AS t2
  ON t1.customer_id = t2.customer_id
INNER JOIN average_category_rental_counts AS t3 
  ON t1.category_name = t3.category_name
INNER JOIN customer_category_percentiles AS t4 
  ON t1.customer_id = t4.customer_id
  AND t1.category_name = t4.category_name;
  
-- Inspect the new table with calulations

SELECT *
FROM customer_category_joint_table
WHERE customer_id = 1
ORDER BY percentile
limit 5;

```

**OUTPUT**


<img width="1226" alt="image" src="https://user-images.githubusercontent.com/77873198/176808475-128967ce-0903-4542-97da-4535767b26e7.png">


Here's the new table with the calculations completed. 

### Checking the data types

``` sql

SELECT
  table_name,
  column_name,
  data_type
FROM information_schema.columns
WHERE table_name in ('customer_total_rentals', 'category_rental_counts');

```


**OUTPUT**

<img width="920" alt="image" src="https://user-images.githubusercontent.com/77873198/176808695-35662446-c689-40df-9103-d359f14a6614.png">


Now that we have some information about the columns and their types the last thing we need to is extract the top rows per customer. 


