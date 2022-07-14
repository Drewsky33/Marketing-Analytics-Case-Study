
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

