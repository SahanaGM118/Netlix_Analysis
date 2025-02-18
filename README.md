# Netlix Analysis Using Sql

![Netflix logo](Netflix_Logo_RGB.png)

## Objective


select* from Netflix;


 ## 1. Count the Number of Movies vs TV Shows

'''select categories,count(*)
from netflix
group by categories;
'''

 ## 2. Find the Most Common Rating for Movies and TV Shows

select 
categories,
rating
FROM(
       select 
           categories,
           rating,
           count(*) as total_rank,
           Rank() over( partition by categories order by count(*) desc) as ranking
       from netflix
       group by 1,2
	   
) as t1
where ranking =1;

 ## 3. List All Movies Released in a Specific Year (e.g., 2020)

select * 
from netflix
where release_year = 2020;

 ## 4. Find the Top 5 Countries with the Most Content on Netflix

select 
UNNEST(string_to_array(country,',')) as country,
count(show_id)
from netflix
group by 1
order by 2 desc
limit 5;

 ##  5.Identify the Longest Movie

select *
from netflix
where  duration= (select Max(duration) from netflix) and categories = 'Movie';

 ##  6. Find Content Added in the Last 5 Years

SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'month dd, YYYY') >= CURRENT_DATE - INTERVAL '5 years';

 ##  7.Find All Movies/TV Shows by Director 'Rajiv Chilaka'

select *
from (
     select 
         *,
         unnest(string_to_array(director,',')) as director_name
     from netflix
) as t
where Director_name = 'Rajiv Chilaka';

 ##  8.List All TV Shows with More Than 5 Seasons

select *
from netflix
where categories = 'TV Show' 
AND SPLIT_PART(duration, ' ', 1)::INT > 5;

-- ::INT > 5 converted into number

 ##  9. Count the Number of Content Items in Each Genre

select 
unnest(string_to_array(listed_in,',')) as genre,
COUNT(*) AS total_content
from netflix
group by genre;

 ##  10.Find each year and the average numbers of content release in India on netflix.


select 
EXTRACT(year from (to_date(date_added, 'month, DD YYYY'))) as years,
count(*) as year_count,
round
(
count(*)::Numeric/(select count(*) from netflix where country = 'India')::NUMERIC*100,2)
as AVG_CONTENT 
from netflix
where country = 'India' 
group by 1;


 ##  11.List All Movies that are Documentaries

select * 
from netflix
where   categories='Movie' and listed_in like '%Documentaries';

--12. Find All Content Without a Director

select*
from netflix 
where Director is null;

 ## 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

select *
from netflix
where casts like '%Salman Khan%'
AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;



 ## 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India


select 
unnest(string_to_array(casts,',')) as actors,
count(*)
from netflix
where categories = 'Movie' and country = 'India'
group by actors
order by count(*) desc
limit 10;

 ## 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

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
