
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
- 


2. What is the distribution of the foreign keys within each table?


3. How many unique foreign key values exist in each table?

