# Netfilx Movies And Tv Shows Data Analysis using sql

![Netflix logo ](https://github.com/Mahto123i/netfilx_sql_project/blob/main/logo.png)

## Overview 

This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives
* Analyze the distribution of content types (movies vs TV shows).
* Identify the most common ratings for movies and TV shows.
* List and analyze content based on release years, countries, and durations.
* Explore and categorize content based on specific criteria and keywords.

## Dataset
The data for this project is sourced from the Kaggle dataset:
- **dataset link:** [movies dataset](https://www.kaggle.com/datasets/aspillai/netflix-stock-price-with-indicators)

...sql

create table  netflix
(
show_id varchar(6),
type  varchar(10),
title  varchar(150),
director  varchar(208),
casts  varchar(1000),
country  varchar(150),
date_added  varchar(50),
release_year int ,
rating  varchar(10),
duration  varchar(15),
listed_in  varchar(100),
description  varchar(250)
);
...

## Business Promblems soluutions


SELECT
	*
FROM
	NETFLIX;

SELECT
	COUNT(*) AS TOTAL_CONTENT
FROM
	NETFLIX;

SELECT DISTINCT
	TYPE
FROM
	NETFLIX;

## 1. count the number of movies vs tv shows

...sql
SELECT
	TYPE,
	COUNT(*) AS TATAL_CONTENT
FROM
	NETFLIX
GROUP BY
	TYPE
...

## 2. find the most common rating for movies and tv shows

...sql
SELECT
	TYPE,
	RATING
FROM
	(
		SELECT
			TYPE,
			RATING,
			COUNT(*),
			RANK() OVER (
				PARTITION BY
					TYPE
				ORDER BY
					COUNT(*) DESC
			) AS RANKING
		FROM
			NETFLIX
		GROUP BY
			1,
			2
	) AS T1
WHERE
	RANKING = 1
 ...
 
## 3. list all movies released in a specific year (e.g. 2020)

...sql
SELECT
	*
FROM
	NETFLIX
WHERE
	TYPE = 'Movie'
	AND RELEASE_YEAR= 2020
...

## 4. find the top 5 countries with the most content on netflix 

...sql
SELECT
	UNNEST(STRING_TO_ARRAY(COUNTRY, ',')) AS NEW_COUNTRY,
	COUNT(SHOW_ID) AS TOTAL_CONTENT
FROM
	NETFLIX
GROUP BY
	1
ORDER BY
	2 DESC
LIMIT
	5
 ...
-- single country karne ke liye 
	
SELECT
	UNNEST(STRING_TO_ARRAY(COUNTRY, ',')) AS NEW_COUNTRY
FROM
	NETFLIX

-- 5. Identify the longest movi or tv show duration

SELECT
	*
FROM
	NETFLIX
WHERE
	TYPE = 'Movie'
	AND DURATION = (
		SELECT
			MAX(DURATION)
		FROM
			NETFLIX
	)
-- 6. find content added in the last 5 years

SELECT
	*
FROM
	NETFLIX
WHERE
	TO_DATE(DATE_ADDED, 'Month dd , yyyy') >= CURRENT_DATE - INTERVAL '5 years'
SELECT
	CURRENT_DATE - INTERVAL '5 years'

-- 7. find all the movies/ tv show by director 'rajiv chilaka'
SELECT
	*
FROM
	NETFLIX
WHERE
	DIRECTOR LIKE '%Rajiv Chilaka%'


-- 8. list all tv show with more than 5 seasons

SELECT
	*
FROM
	NETFLIX
WHERE
	TYPE = 'TV Show'
	AND SPLIT_PART(DURATION, ' ', 1)::NUMERIC > 5


-- 9. count the number of content items in each genre

SELECT
	UNNEST(STRING_TO_ARRAY(LISTED_IN, ',')) AS GENRE,
	COUNT(SHOW_ID) AS TOTAL_CONTANT
FROM
	NETFLIX GROUP BY
	1
	
-- 10. find the average release year for content produced in a specifice country

SELECT
	EXTRACT(
		YEAR
		FROM
			TO_DATE(DATE_ADDED, 'Month dd, yyyy')
	) AS YEAR,
	COUNT(*) AS YEARLY_CONTENT,
	ROUND(
		COUNT(*)::NUMERIC / (
			SELECT
				COUNT(*)
			FROM
				NETFLIX
			WHERE
				COUNTRY = 'India'
		) * 100,
		2
	) AS AVG_CONTENT_PER_YEARFROM
	NETFLIX
WHERE
	COUNTRY = 'India'
GROUP BY
	1

-- 11. list all movies that are documentaries

SELECT
	*
FROM
	NETFLIX
WHERE
	LISTED_IN LIKE '%Documentaries%'

-- 12. find all content without a director

SELECT
	*
FROM
	NETFLIX
WHERE
	DIRECTOR IS NULL

-- 13. find how many movies actor 'salman khamn' appeared in last 10 years

SELECT
	*
FROM
	NETFLIX
WHERE
	CASTS ILIKE '%Salman khan%'
	AND RELEASE_YEAR > EXTRACT(
		YEAR
		FROM
			CURRENT_DATE
	) - 10

-- 14. find the top 10 actors who have appeared in the higest number of movies produced in india

SELECT
	UNNEST(STRING_TO_ARRAY(CASTS, ',')) AS ACTORS,
	COUNT(*) NETFLIX
FROM
	NETFLIX
WHERE
	COUNTRY ILIKE '%india%'
GROUP BY
	1
ORDER BY
	2 DESC LIMIT
	10

-- 15. categorize the content based  on the presence of the keywords 'kill' and 'violence' in the
--description filed. label content containing these keywords as 'Bad'and all other content as 'Good'count 
--how many item fall into each category.

WITH
	NEW_TABLE AS (
		SELECT
			*,
			CASE
				WHEN DESCRIPTION ILIKE '%kill%'
				OR DESCRIPTION ILIKE '%violence%' THEN 'Bad_contebt'
				ELSE 'Good content'
			END CATEGORY
		FROM
			NETFLIX
	)
SELECT
	CATEGORY,
	COUNT(*) AS TOTAL_CONTENT FROM
	NEW_TABLE
GROUP BY
	1
