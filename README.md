# Netflix Movies and TV Shows Data Analysis using SQL

![Netflix Logo](https://github.com/jlarenagon/Netflix_SQL_Project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (Movies vs TV shows).
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
**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
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

WHERE ranking = 1
```
**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2020)

```sql
SELECT * FROM netflix
WHERE 
   type = 'Movie'
   AND
   release_year = 2020
```
**Objective:** Retrieve all movies released in a specific year.

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT 
   UNNEST(STRING_TO_ARRAY(country, ',')) as new_country,
   COUNT(show_id) as total_content
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```
**Let's FIX the Multiple Values Received on a String List at the 'Country' Column:**

```sql
   SELECT
      UNNEST(STRING_TO_ARRAY(country, ',')) as new_country
   FROM netflix
```
**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Identify the Longest Movie

```sql
SELECT * FROM netflix
WHERE
   type = 'Movie'
   AND
   duration = (SELECT MAX(duration) FROM netflix)
```
**Objective:** Find the movie with the longest duration.

### 6. Find Content Added in the Last 5 Years

```sql
SELECT * FROM netflix
WHERE
   TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'
```
**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find All Movies/TV Shows by Director 'Clint Eastwood'

```sql
SELECT * FROM netflix
WHERE
   director = 'Clint Eastwood'
```
**Objective:** List all content directed by 'Clint Eastwood'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
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
```
**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql
SELECT
   UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
   COUNT(show_id) as "Number of Content"
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
```
**Objective:** Count the number of content items in each genre.

### 10.Find each year and the average numbers of content release in "United States" on netflix. 
return top 5 year with highest avg content release!

```sql
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
```
**Objective:** Calculate and rank years by the average number of content releases by United States.

### 11. List All Movies that are Documentaries

```sql
SELECT 
   type,
   title,
   listed_in
FROM netflix
WHERE 
   type = 'Movie'
   AND
   listed_in ILIKE '%documentaries%'
```
	
**ONLY Documentaries results, no mixed styles:**

```sql
SELECT 
   type,
   title,
   listed_in
FROM netflix
WHERE 
   type = 'Movie'
   AND
   listed_in LIKE 'Documentaries'
```
**Objective:** Retrieve all movies classified as Documentaries.

### 12. Find All Content Without a Director

```sql
SELECT * 
FROM netflix
WHERE 
   director IS NULL
```
**Objective:** List content that does not have a director.

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

```sql
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
```
**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in United States.

```sql
SELECT
   UNNEST(STRING_TO_ARRAY(casts, ',')) as actors,
   COUNT(*) as total_content
FROM netflix
WHERE country ILIKE '%United States'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
```

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords
   
```sql
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
```

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.

**Thank you if you made it to the end!**
