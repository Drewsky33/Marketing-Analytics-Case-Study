
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

Great it looks like there is now difference between joining the two tables. Next, we have to go through part two of our table joining journey which was joining the inventory table to the film table to get the film_id foreign key. 

Again, we need to go through our questions of why we are completing this join:

1. What's the purpose of this join?
**Hypothesis for this table**
- We want to match the films on the film_id so that we can get the film name or `title`. 
- There is most likely a ]

2. What is the distribution of the foreign keys within each table?


3. How many unique foreign key values exist in each table?

