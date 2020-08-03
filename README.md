# Udacity_Nanodegree_SQL_Project

### Description
This project contains SQL queries in order to provide answers for following interesting questions in a data set of different sorts of movies.

The following list provides some of the crucial information about the considered data set.
1. Information about movies
  * Movie title
  * Movie category
    * Animation
    * Children
    * Classics
    * Music
    * Comedy
    * Family
2. Rental Duration
3. Customer Based Information
  * Customer ID
  * Customer name
4. Store Based Information
  * Store ID
  * Store locations
  * Inventory relevant information
5. Payment Details
  * Total payment per movie
  * Payment date
  * Payment frequencies
  * Payment amount per month

The following questions were studied within the framework of this project.

###### QUESTION 1
We want to understand more about the movies that families are watching. The following categories are considered family movies: Animation, Children, Classics, Comedy, Family and Music.
Create a query that lists each movie, the film category it is classified in, and the number of times it has been rented out.

```
WITH category_selection AS(
  SELECT name, category_id
  FROM category
  WHERE name = 'Animation' OR name ='Children' OR name ='Classics' OR name ='Comedy' OR name ='Family' OR name ='Music')

SELECT f.title film_title, category_selection.name category_name,
COUNT(r.rental_id) rental_count
FROM category_selection
JOIN film_category fc
ON category_selection.category_id=fc.category_id
JOIN film f
ON f.film_id=fc.film_id
JOIN inventory i
ON i.film_id = f.film_id
JOIN rental r
ON r.inventory_id=i.inventory_id
GROUP BY 1,2
ORDER BY 2;
  ```


###### QUESTION 2
Now we need to know how the length of rental duration of these family-friendly movies compares to the duration that all movies are rented for.
Can you provide a table with the movie titles and divide them into 4 levels (first_quarter, second_quarter, third_quarter, and final_quarter) based on the quartiles (25%, 50%, 75%) of the rental duration for movies across all categories? Make sure to also indicate the category that these family-friendly movies fall into.

```
SELECT f.title title, c.name category, f.rental_duration rental_duration,
NTILE(4) OVER (PARTITION BY rental_duration) AS standard_quartile
FROM film f
JOIN film_category fc
ON f.film_id=fc.film_id
JOIN category c
ON c.category_id=fc.category_id
WHERE c.name = 'Animation' OR c.name ='Children' OR c.name ='Classics' OR c.name ='Comedy' OR c.name ='Family' OR c.name ='Music';
```

###### QUESTION 3
We want to find out how the two stores compare in their count of rental orders during every month for all the years we have data for. Write a query that returns the store ID for the store, the year and month and the number of rental orders each store has fulfilled for that month. Your table should include a column for each of the following: year, month, store ID and count of rental orders fulfilled during that month.

```
SELECT DATE_PART ('month', r.rental_date) AS rental_month,
DATE_PART ('year', r.rental_date) AS rental_year,
s.store_id, COUNT(*) AS count_rentals
FROM store s
JOIN staff sf
ON sf.store_id=s.store_id
JOIN rental r
ON sf.staff_id=r.staff_id
GROUP BY 1,2,3
ORDER BY 4 DESC;
```

###### QUESTION 4
We would like to know who were our top 10 paying customers, how many payments they made on a monthly basis during 2007, and what was the amount of the monthly payments. Can you write a query to capture the customer name, month and year of payment, and total payment amount for each month by these top 10 paying customers?

```
WITH top10 AS(SELECT c.first_name || ' ' || last_name AS full_name,
c.customer_id,
SUM(p.amount) pay_amount
FROM customer c
JOIN payment p
ON p.customer_id=c.customer_id
WHERE DATE_PART('year', payment_date) = '2007'
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 10)

SELECT DATE_TRUNC('month', p.payment_date) AS pay_mon,
top10.full_name,
COUNT(*) AS pay_countpermon,
SUM(p.amount) pay_amount
FROM payment p
JOIN top10
ON top10.customer_id=p.customer_id
GROUP BY 1,2
ORDER BY 2,1;
```
