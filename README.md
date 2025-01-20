# Netflix Movies and TV Shows Data Analysis using SQL

![Netflix Logo](https://github.com/jlarenagon/Netflix_SQL_Project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(20),
    type         VARCHAR(20),
    title        VARCHAR(150),
    director     VARCHAR(210),
    casts        VARCHAR(1000),
    country      VARCHAR(150),
    date_added   VARCHAR(50),
    release_year INT,
    rating       VARCHAR(10),
    duration     VARCHAR(20),
    listed_in    VARCHAR(100),
    description  VARCHAR(250)
);
```

## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows

```sql
SELECT
	type,
	COUNT(*) as total_content
FROM netflix
GROUP BY type
```

### 2. Find the most common rating for movies and TV shows

SELECT
	type,
	rating
FROM
(
	SELECT
		type,
		rating,
		COUNT(*),
		RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
	FROM netflix
	GROUP BY 1, 2
) as t1

WHERE 
	ranking = 1


### 3. List all movies released in a specific year (e.g., 2020)

SELECT * FROM netflix
WHERE 
	type = 'Movie'
	AND
	release_year = 2020


### 4. Find the top 5 countries with the most content on Netflix

SELECT 
	UNNEST(STRING_TO_ARRAY(country, ',')) as new_country,
	COUNT(show_id) as total_content
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5

	-- Let's fix the multiple values String list on 'country' column:

	SELECT 
		UNNEST(STRING_TO_ARRAY(country, ',')) as new_country
	FROM netflix


### 5. Identify the longest movie

SELECT * FROM netflix
WHERE
	type = 'Movie'
	AND
	duration = (SELECT MAX(duration) FROM netflix)


### 6. Find content added in the last 5 years

SELECT * FROM netflix
WHERE
	TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'


### 7. Find all the movies/TV shows by director 'Clint Eastwood'!

SELECT * FROM netflix
WHERE
	director = 'Clint Eastwood'


### 8. List all TV shows with more than 5 seasons

SELECT 
	type,
	title,
	duration
FROM netflix
WHERE 
	type = 'TV Show'
	AND
	SPLIT_PART(duration, ' ', 1)::numeric > 5
ORDER BY 3 DESC


### 9. Count the number of content items in each genre

SELECT
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(show_id) as "Number of Content"
FROM netflix
GROUP BY 1
ORDER BY 2 DESC


### 10.Find each year and the average numbers of content release in "United States" on netflix. 
return top 5 year with highest avg content release!*/

SELECT
	EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) as Year,
	COUNT(*) as yearly_content,
	ROUND(
	COUNT(*)::numeric/ (SELECT COUNT(*) FROM netflix WHERE country = 'United States')::numeric * 100 , 2) as "AVG_content_yearly"
FROM netflix
WHERE country = 'United States'
GROUP BY 1
ORDER BY 3 DESC
LIMIT 5


### 11. List all movies that are documentaries

SELECT 
	type,
	title,
	listed_in
FROM netflix
WHERE 
	type = 'Movie'
	AND
	listed_in ILIKE '%documentaries%'
	
--- ONLY Documentaries, no mixed styles:
SELECT 
	type,
	title,
	listed_in
FROM netflix
WHERE 
	type = 'Movie'
	AND
	listed_in LIKE 'Documentaries'


### 12. Find all content without a director

SELECT * 
FROM netflix
WHERE 
	director IS NULL


### 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!

SELECT DISTINCT
	show_id,
	type,
	title,
	casts,
	release_year
FROM netflix
WHERE 
	casts ILIKE '%Salman Khan%'
	AND
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 15
ORDER BY release_year DESC


### 14. Find the top 10 actors who have appeared in the highest number of movies produced in United States.

SELECT
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actors,
	COUNT(*) as total_content
FROM netflix
WHERE country ILIKE '%United States'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10


### 15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
   the description field. Label content containing these keywords as 'Bad' and all other 
   content as 'Good'. Count how many items fall into each category. */

WITH Categorize_Table
AS
(
	SELECT *,
	  CASE
		WHEN
			description ILIKE '%kill%' OR
			description ILIKE '%violence%' THEN 'Bad_Content'
			ELSE 'Good Content'
		END as category
	FROM netflix
)
SELECT 
	category,
	COUNT(*) as total_content
FROM Categorize_Table
GROUP BY 1
