# Marketing-Analytics-Case-Study

This project is from Danny Ma's Serious SQL course. 

## Scenario
The marketing team has shared a draft of e-mails they wish to send to customers. Creating personalized customer emails is a staple of a lot of digital companies. We have been asked to support the customer analytics team at DVD Rental Co who want us to gather data points required to popular of their first email marketing campaign. 

I'll be using SQL to solve these problems and capture the data needed to ensure a smooth marketing campaign. 

## Example email template


Requirements:
1. For each customer we need to identify the top 2 categories based on their past rental history. These categories will drive what kind of images will be put into the email. 
2. The marketing team has also asked that we include the 3 most popular fils for each customer's top 2 categories. **However,** we can't recommend a film that the customer has seen already. If there isn't 3 films available, marketing is more than happy to show at least 1 film. If the customer doesn't have any film recommendations for either of their categories, then the customer needs to be flagged so they can be excluded from the email marketing campaign. 3. The marketing team also requires that we provide the following insights for each customer:
  - For category 1:
    * How many total films have they watched in their category?
    * How many more films has the customer watched in comparison to the average customer?
    * How does the customer rank in terms of the top x% compared to all other customers in the film category?
  - For category 2:
    * How many total films has the customer watched in this category?
    * What proportion of each customer's total films watched does this count make?
5. Favorite actor recommendations: Include up to 3 other films featuring their favorite actor. 
  - Additionally, films that have been recommended in top 2 categories must not be included in actor recommendations. 
  - If they don't 1 film recommendation, they need to be flagged with a separator actor exclusion flag. 

## Data Exploration

First, we need to generator an entity-relationship diagram in order to see which table schema are of particular importance for our business challenges. I'm going to look through the tables that we'll be using in our analysis.

### Table 1: Rental Table

``` sql
SELECT
  *
FROM dvd_rentals.rental
WHERE customer_id = 130;

```

** OUTPUT**:

<img width="1290" alt="image" src="https://user-images.githubusercontent.com/77873198/175211890-61dbfcfe-2407-4d8a-8734-acec1747c111.png">


Looking at our table I can see different pieces of information:
- Rental_id: Unique id for the record in the table that corresponds to a specific sutomer. 
- Rental_date and return_date: Dates for the term of the rental
- Inventory_id: Tells which specific item is being rented. 
- Customer_id: Unique id for the customer renting. 
- Staff_id: Unique id for staff member who processed the transaction. 
- Last_update:  The last time there was an update on that spceific rental in the database.

### Table 2: Inventory Table
This table has information about the relationship between specific items at that are avaiable for rent at each store. This table is linked with the rental table through the inventory_id key. Let's have a look at the raw data:

``` sql

SELECT *
FROM dvd_rentals.inventory
WHERE film_id = 1;

```
** OUTPUT**:

<img width="765" alt="image" src="https://user-images.githubusercontent.com/77873198/175214967-01090e1c-561d-422d-8faf-3749cd2f1684.png">


### Table 3: Film table

This table will allow for us to identify the tile of the films rented by the customers. Additionally, information is included for a deeper more granular analysis. A very interesting metric to me is the rental_rate. Additionally, this table allows us to join with Table #4 film_actor so we can identify who appears in what films. Let's have a look at the raw data:

``` sql

SELECT *
FROM dvd_rentals.film
LIMIT 5;


```

### Table 4: Film category
The 4th table we'll be going through is the film_category table. This table shows us that multiple films will appear in each relevant category. 

``` sql
SELECT *
FROM dvd_rentals.film_category
LIMIT 5;

```

**OUTPUT**:

<img width="559" alt="image" src="https://user-images.githubusercontent.com/77873198/175336671-d97609d8-f253-4c8e-99ca-8092d453e1bc.png">

This is nice to have. However, we do need to access the categories names and we can see that the film_category table is linked to the category table through the category_id key. The category table actually has the name within it. 


### Table 5: Category

This is the table that will allows us to access the category name. 

``` sql

SELECT *
FROM dvd_rentals.category
LIMIT 5;

```

<img width="587" alt="image" src="https://user-images.githubusercontent.com/77873198/175337329-146c5438-d10b-4c70-8211-ac0ccfec33ea.png">





