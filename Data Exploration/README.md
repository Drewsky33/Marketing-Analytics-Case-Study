
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
