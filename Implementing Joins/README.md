
### Joining the tables
We can now now implement the inner join, but I'm also going to do a left join to demonstrate that there isn't a difference between the output. 

```sql 
DROP TABLE IF EXISTS left_rental_join;
CREATE TEMP TABLE left_rental_join AS 
  SELECT
    rental.customer_id,
    rental.inventory_id,
    inventory.film_id
  FROM dvd_rentals.rental
  LEFT JOIN dvd_rentals.inventory
    ON rental.inventory_id = inventory.inventory_id;


DROP TABLE IF EXISTS inner_rental_join;
CREATE TEMP TABLE inner_rental_join AS 
  SELECT
    rental.customer_id,
    rental.inventory_id,
    inventory.film_id
  FROM dvd_rentals.rental
  INNER JOIN dvd_rentals.inventory
    ON rental.inventory_id = inventory.inventory_id;

-- Checks count for each output

(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT inventory_id) AS unique_key_values
  FROM left_rental_join
)

UNION 
(
  SELECT
    'inner join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT inventory_id) AS unique_key_values
  FROM inner_rental_join
);

```

**OUTPUT**

<img width="586" alt="image" src="https://user-images.githubusercontent.com/77873198/176082355-f304fb2a-b862-48d6-9e97-99a778f2359e.png">


### Joins pt. 2

Great it looks like there is now difference between joining the two tables. Next, we have to go through part two of our table joining journey which was joining the inventory table to the film table to get the film_id foreign key. 

Again, we need to go through our questions of why we are completing this join:

1. What's the purpose of this join?

**Hypothesis for this table**

- We want to match the films on the film_id so that we can get the film name or `title`. 
- There is most likely a 1 to many relationship between the film_id table and the rows for the inventory table. This means that we have multiple copies of each film in inventory. 
- However in the films table there should be a 1 to 1 relationship for the film_id column. Why? because we shouldn't have duplicates for film names. 

### Testing our hypothesis

``` sql
WITH base_counts AS (
  SELECT
    film_id,
    COUNT(*) AS record_count
  FROM dvd_rentals.inventory
  GROUP BY film_id
)

SELECT
  record_count,
  COUNT(DISTINCT film_id) AS unique_film_ids
FROM base_counts
GROUP BY record_count
ORDER BY record_count;

```

**OUTPUT**

<img width="382" alt="image" src="https://user-images.githubusercontent.com/77873198/176764780-3fced6dc-8c96-47fb-9342-f5a001bbdc80.png">



- So we based on this we can see that there 133 unique film ids that have two copies in inventory. We confirm our hypothesis that the film_inventory table has a 1 to many relationship for film_id. 

- Next, we'll try and confirm our other hypothesis that the film table has a 1 to 1 relationship for film_id. 

``` sql

SELECT
  film_id, 
  COUNT(*) AS record_count
FROM dvd_rentals.film 
GROUP BY film_id
ORDER BY record_count DESC;

```

**OUTPUT**

<img width="280" alt="image" src="https://user-images.githubusercontent.com/77873198/176765148-558f7795-82b9-4cb1-81c8-3b78351e4576.png">


We can confirm our other hypothesis that indeed we have a 1 to 1 relationship between the film_id and number records it has in the `dvd_rentals.film` table. Let's move on to our next question. 


2. What is the distribution of the foreign keys within each table?
- As we confirmed before there is a 1 to many relationship for film_id in the inventory table and a 1 to 1 relationship in the film table. 


3. How many unique foreign key values exist in each table?
- To do this we can use an ANTI JOIN again to check whether or not any values exist in the inventory table that don't exist in the film_table. There shouldn't be a problem here because the inventory should contain all of the films we have. So it should be 0. 

``` sql

SELECT
  COUNT(DISTINCT inventory.film_id)
FROM dvd_rentals.inventory
WHERE NOT EXISTS (
  SELECT film_id
  FROM dvd_rentals.film
  WHERE inventory.film_id = film.film_id
);

```

**OUTPUT**

<img width="180" alt="image" src="https://user-images.githubusercontent.com/77873198/176767128-141da9f2-9677-4d9f-922c-4226a1c79879.png">


We can confirm the statement earlier that there are no unique film_id's between tables as expected. We need to check with the film_table as the base table now and I suspect there may be some unique values from the film_table that aren't present in the inventory table. 

``` sql

SELECT
  COUNT(DISTINCT film.film_id)
FROM dvd_rentals.film
WHERE NOT EXISTS (
  SELECT film_id
  FROM dvd_rentals.inventory
  WHERE film.film_id = inventory.film_id
);

```

**OUTPUT**

<img width="185" alt="image" src="https://user-images.githubusercontent.com/77873198/176767729-582f547a-2fa0-4888-8ac2-5eb47685bd51.png">


It looks like there are a significant amount of film_ids that are present in the film table, but not in the inventory table. 

There's not much we can do about that as the store has it's inventory set, but this could be used later for adding more variety in film selection. Again, our goal is to make a personalized e-mail template so that falls outside of the scope of this project. 

### Implementing our next join
Before we perform the join, let's take a look at how many unique film_id values we will generate when joining by performing a left semi join and gettting a count for the values in the base table (inventory). 

``` sql
SELECT
  COUNT(DISTINCT inventory.film_id)
FROM dvd_rentals.inventory
WHERE EXISTS (
  SELECT
    film_id
  FROM dvd_rentals.film
  WHERE inventory.film_id = film.film_id
);

```

**OUTPUT**

<img width="177" alt="image" src="https://user-images.githubusercontent.com/77873198/176768536-b58de794-d6b0-4be5-901c-9d501f1917bb.png">


It looks like we will have 958 unique film_id values when we perform our join. 

**Implementing the join**

We know what information will be present after we perform our join and will implement an inner join. Again, as we saw before there isn't a difference between a left and inner join on these particular tables. But I'll check to be sure. 

``` sql

DROP TABLE IF EXISTS left_join_part_2;
CREATE TEMP TABLE left_join_part_2 AS 
SELECT
  inventory.inventory_id,
  inventory.film_id,
  film.title
FROM dvd_rentals.inventory
LEFT JOIN dvd_rentals.film 
  ON inventory.film_id = film.film_id;
  

DROP TABLE IF EXISTS inner_join_part_2;
CREATE TEMP TABLE inner_join_part_2 AS
SELECT
  inventory.inventory_id,
  inventory.film_id,
  film.title
FROM dvd_rentals.inventory
INNER JOIN dvd_rentals.film 
  ON inventory.film_id = film.film_id;
  

--- Check the counts for each of these outputs
--- Compare them by performing a union

(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_counts,
    COUNT(DISTINCT film_id) AS unique_films
  FROM left_join_part_2
)

-- Use Union all because we don't need union for distinct values 
UNION ALL 

(
  SELECT
    'inner join' AS join_type,
    COUNT(*) AS record_counts,
    COUNT(DISTINCT film_id) AS unique_films
  FROM inner_join_part_2
);

```

**OUTPUT**

<img width="580" alt="image" src="https://user-images.githubusercontent.com/77873198/176770891-ecc0d479-2885-41fe-85e6-d89fc1a079e4.png">


As you can see we have the same number of record counts after each join and the same number of unique films. We're not done **YET**, now we need to join the new table with the one we created before when joining the rental table to the inventory table. We need **ALL** of this information. 

```sql
DROP TABLE IF EXISTS join_parts_1_and_2;
CREATE TEMP TABLE join_parts_1_and_2 AS 
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id;

SELECT * FROM join_parts_1_and_2;

```

**OUTPUT**

<img width="539" alt="image" src="https://user-images.githubusercontent.com/77873198/176771994-d721cf45-eba6-4859-8985-4f559680419d.png">


We now have the customer_id, the film_id, and the title of the film. We're making progress! Next, let's have a look at our join journey progress and decided where to go next. 

<img width="656" alt="image" src="https://user-images.githubusercontent.com/77873198/176774411-c619ae0d-a7ef-4193-8b6f-9d51fbe1c617.png">

