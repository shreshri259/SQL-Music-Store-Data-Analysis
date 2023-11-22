
# Music Store Data Analysis in SQL (PostgreSQL 16)

This project invloves analysing the data of a fictional music store using SQL.

## Instructions to Set up and Use

- Install the latest version of PostgreSQL and pgAdmin 4 on your computer.
- Download the music store data SQL file (Music_Store_database.sql) from this repository and import it into your SQL server database.

## Database Schema

![Music Database Schema](https://raw.github.com/shreshri259/SQL-Music-Store-Data-Analysis/main/MusicDatabaseSchema.png)

## Data Analysis using SQL

![Questions](https://raw.github.com/shreshri259/SQL-Music-Store-Data-Analysis/main/Questions.png)

1. Senior most employee based on job title

```
SELECT * FROM employee
ORDER BY levels DESC
LIMIT 1
```

2. Countries having the most number of invoices

```
SELECT COUNT(*) AS c, billing_country
from invoice
GROUP BY billing_country
ORDER BY c DESC
```

3. Top 3 values of total number of invoices

```
SELECT total from invoice
ORDER BY total DESC
LIMIT 3
```

4. Name of city with highest sum of invoice totals

```
SELECT SUM(total) AS invoice_total, billing_city
FROM invoice
GROUP BY billing_city
ORDER BY invoice_total DESC
LIMIT 1
```

5. Customer who has spent the most amount of money

```
SELECT c.customer_id, c.first_name, c.last_name, SUM(v.total) as total
FROM customer AS c
JOIN invoice AS v
ON c.customer_id = v.customer_id
GROUP BY c.customer_id
ORDER BY total DESC
LIMIT 1
```

6. Email, first name, last name and genre of all Rock music listeners ordered alphabetically by email starting with the letter "A"

```
SELECT DISTINCT email, first_name, last_name
FROM customer AS C
JOIN invoice AS I ON c.customer_id = I.customer_id
JOIN invoice_line AS L ON I.invoice_id = L.invoice_id
WHERE track_id IN(
	SELECT track_id
	FROM track AS T
	JOIN genre AS G
	ON T.genre_id = G.genre_id
	WHERE G.name LIKE 'Rock'
)
ORDER BY email
```
OR

```
SELECT DISTINCT email, first_name, last_name, G.name AS genre_name
FROM customer AS C
JOIN invoice AS I ON c.customer_id = I.customer_id
JOIN invoice_line AS L ON I.invoice_id = L.invoice_id
JOIN track AS T ON T.track_id = L.track_id
JOIN genre AS G ON T.genre_id = G.genre_id
WHERE G.name LIKE 'Rock'
ORDER BY email
```

7. Name of artists and total track count of top 10 Rock bands

```
SELECT A.artist_id, A.name, COUNT(A.artist_id) AS No_of_songs
FROM track AS T
JOIN album AS M ON M.album_id = T.album_id
JOIN artist AS A ON A.artist_id = M.artist_id
JOIN genre AS G ON G.genre_id = T.genre_id
WHERE G.name LIKE 'Rock'
GROUP BY A.artist_id
Order BY No_of_songs DESC
LIMIT 10
```

8. Names and milliseconds of tracks that have a song length longer than the average song length, ordered by song length with the longest songs listed first

```
SELECT name, milliseconds
FROM track
WHERE milliseconds > (
	SELECT AVG(milliseconds) AS avg_track_length
	FROM track
)
ORDER BY milliseconds DESC
```

9. Amount spent by each customer on artists (Customer name, Artist name, Total amount spent)

```
WITH best_selling_artist AS (
	SELECT A.artist_id, A.name AS artist_name, SUM(L.unit_price*L.quantity) AS total_sales
	FROM invoice_line AS L
	JOIN track AS T ON L.track_id = T.track_id
	JOIN album AS M ON T.album_id = M.album_id
	JOIN artist AS A ON M.artist_id = A.artist_id
	GROUP BY 1
	ORDER BY 3 DESC
	LIMIT 1
)
SELECT C.customer_id, C.first_name, C.last_name, BSA.artist_name, SUM(L.unit_price*L.quantity) AS amount_spent
FROM invoice AS I
JOIN customer AS C ON I.customer_id = C.customer_id
JOIN invoice_line AS L ON I.invoice_id = L.invoice_id
JOIN track AS T ON L.track_id = T.track_id
JOIN album AS M ON T.album_id = M.album_id
JOIN best_selling_artist AS BSA ON M.artist_id = BSA.artist_id
GROUP BY 1, 2, 3, 4
ORDER BY 5 DESC
```

10. Most popular music genre for each country, or the genre with the highest amount of purchases (For countries where the maximum number of purchases is shared, all genres sharing that is displayed)

```
WITH popular_genre AS
(
	SELECT COUNT(L.quantity) AS purchases, C.country, G.name, G.genre_id,
	ROW_NUMBER() OVER(PARTITION BY C.country ORDER BY COUNT(L.quantity) DESC) AS RowNo
	FROM invoice_line AS L
	JOIN invoice AS I ON I.invoice_id = L.invoice_id
	JOIN customer AS C ON C.customer_id = I.customer_id
	JOIN track AS T ON T.track_id = L.track_id
	JOIN genre AS G ON G.genre_id = T.genre_id
	GROUP BY 2, 3, 4
	ORDER BY 2 ASC, 1 DESC
)
SELECT * FROM popular_genre WHERE RowNo <= 1
```

11. Customer that has spent the most on music in each country and how much they spent (For countries where the top amount spent is shared, all customers who spent that amount is displayed)

```
WITH Customer_with_country AS 
(
	SELECT C.customer_id, first_name, last_name, billing_country, SUM(total) AS total_spending,
	ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo
	FROM invoice AS I
	JOIN customer AS C ON C.customer_id = I.customer_id
	GROUP BY 1, 2, 3, 4
	ORDER BY 4 ASC, 5 DESC
)
SELECT * FROM Customer_with_country WHERE RowNo <= 1
```

## Acknowledgements

- [Rishabh Mishra - YouTube](https://www.youtube.com/@RishabhMishraOfficial)
- [SQL Data Analytics Portfolio Project](https://www.youtube.com/watch?v=VFIuIjswMKM)

### Disclaimer

This project is a fictional project created for demonstrations purposes only. The data used are placeholders and do not reflect real data. 
