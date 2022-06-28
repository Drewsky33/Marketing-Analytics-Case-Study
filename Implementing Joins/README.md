
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
