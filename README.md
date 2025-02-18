# Netlix Analysis Using Sql

![Netflix logo](Netflix_Logo_RGB.png)


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
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
SELECT * FROM Netflix;
```



### 1. Count the Number of Movies vs TV Shows

```sql
SELECT 
   categories,
   count(*)
FROM netflix
GROUP BY categories;
```

 ## 2. Find the Most Common Rating for Movies and TV Shows


```sql
SELECT 
categories,
rating
FROM(
       SELECT 
           categories,
           rating,
           COUNT(*) as total_rank,
           Rank() over( partition by categories order by count(*) desc) as ranking
       FROM netflix
       GROUP BY 1,2	   
) as t1
WHERE ranking =1;
```


 ## 3. List All Movies Released in a Specific Year (e.g., 2020)

```sql
SELECT  * 
FROM netflix
WHERE release_year = 2020;
```


 ## 4. Find the Top 5 Countries with the Most Content on Netflix
```sql
SELECT 
UNNEST(string_to_array(country,',')) as country,
COUNT(show_id)
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```


 ##  5.Identify the Longest Movie

```sql
SELECT  *
from netflix
where  duration= (select Max(duration) from netflix) and categories = 'Movie';
```


 ##  6. Find Content Added in the Last 5 Years

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'month dd, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```


 ##  7.Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
SELECT  *
FROM (
     select 
         *,
         unnest(string_to_array(director,',')) as director_name
     FROM netflix
) as t
WHERE Director_name = 'Rajiv Chilaka';
```


 ##  8.List All TV Shows with More Than 5 Seasons

```sql
SELECT  *
FROM netflix
WHERE categories = 'TV Show' 
AND SPLIT_PART(duration, ' ', 1)::INT > 5;

-- ::INT -> converted into number
```


 ##  9. Count the Number of Content Items in Each Genre

```sql
SELECT 
UNNEST(string_to_array(listed_in,',')) as genre,
COUNT(*) AS total_content
FROM netflix
GROUP BY genre;
```


 ##  10.Find each year and the average numbers of content release in India on netflix.

```sql
SELECT
EXTRACT(year from (to_date(date_added, 'month, DD YYYY'))) as years,
COUNT(*) as year_count,
ROUND
(
COUNT(*)::Numeric/(select count(*) from netflix where country = 'India')::NUMERIC*100,2)
as AVG_CONTENT 
FROM netflix
WHERE country = 'India' 
GROUP BY 1;
```



 ##  11.List All Movies that are Documentaries


```sql
SELECT * 
FROM netflix
WHERE   categories='Movie' and listed_in like '%Documentaries';
```


## --12. Find All Content Without a Director


```sql
SELECT*
FROM netflix 
WHERE Director is null;
```


 ## 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years


```sql
SELECT *
FROM netflix
WHERE casts like '%Salman Khan%'
AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```




 ## 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

```sql
SELECT
UNNEST(string_to_array(casts,',')) as actors,
COUNT(*)
FROM netflix
WHERE categories = 'Movie' and country = 'India'
GROUP BY actors
ORDER BY count(*) desc
LIMIT 10;
```


 ## 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords


```sql
--Using subquiery
SELECT 
    status,
    COUNT(*) AS status_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS status
    FROM netflix
) AS status_content
GROUP BY status;

--Using CTE

with NEW_table 
AS 
(
    SELECT *,
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS status
    FROM netflix
) 
SELECT 
    status,
    COUNT(*) AS content_count
FROM NEW_table
GROUP BY status
```
