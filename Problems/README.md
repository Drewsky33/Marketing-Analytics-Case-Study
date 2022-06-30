
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
We now have all of the information needed to identify the top two categories per customer. 



