# **Netflix Project**![Netflix Logo](https://github.com/ShuvankarBhattacharjee/Netflix_data_project_03/blob/main/netflix_logo.png)

**DESCRIPTION**\n
/n This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and
answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives,
business problems, solutions, findings, and conclusions.

**OBJECTIVES**

· Analyze the distribution of content types (movies vs TV shows).
. Identify the most common ratings for movies and TV shows.
. List and analyze content based on release years, countries, and durations.
· Explore and categorize content based on specific criteria and keywords.

**DATASET**
The data for this project is sourced from the Kaggle dataset.


**SCHEMA**
```
CREATE TABLE netflix
(

show_id	varchar(255),
type varchar(255),	
title	varchar(255),
director	varchar(255),
casts	varchar(1000),
country	varchar(255),
date_added	varchar(255),
release_year int,
rating	varchar(255),
duration	varchar(255),
listed_in	varchar(255),
description varchar(255)

)
```
**Show DATA**
```
SELECT * from netflix;

-- total row CHECK
SELECT count(*)  as total_content_rows 
from netflix;
```
**DISTINCT TYPE**
```
SELECT DISTINCT(type) from netflix;
```

**16 Business Problems & Solutions**
**1. Count the number of Movies vs TV Shows**
```
SELECT type,count(*) as Total_Contents FROM netflix
group by type;
```

**2.Find the most common rating for movies and TV shows**
```
-- if we want to see common ratings
SELECT type, rating, count(*) as Ratings FROM
netflix
group by rating, type
order by Ratings desc;
-- if we want to see grouped rating like ranking

SELECT type, rating, count(*) as Total_Ratings,
RANk() OVER(partition by type order by count(*) desc)
from netflix
group by 1,2;

-- if we want to see top rating for each type
SELECT type, rating
FROM(

SELECT type, rating, count(*) as Total_Ratings,
RANk() OVER(partition by type order by count(*) desc) as ranking
from netflix
group by 1,2

) as t1
WHERE ranking = 1;

```

**3. List all movies released in a specific year (e.g., 2020)**
```
-- movies release on 2020
select * from netflix
WHERE type = 'Movie' and release_year = 2020;

--if we want to see all movies till now
select title,release_year from netflix
WHERE type = 'Movie' 
order by 2 desc;
```
**4. Find the top 5 countries with the most content on Netflix**
```
SELECT unnest(string_to_array(country,',')) as Country_name, count(show_id) as total_Contents
from netflix
group by 1
order by 2 desc
limit 5;
```
**5. Identify the longest movie or TV show duration**
```
select * from netflix
WHERE type = 'Movie'
and
duration = (select max(duration) from netflix);
```
**6. Find content added in the last 5 years**
```
SELECT * from netflix
where to_date(date_added,'Month DD, YYYY') >= current_date - interval '5 years' 
```
**7. Find all the movies/TV shows by director 'Rajiv Chilaka'!**
```
select * from netflix
where director like '%Rajiv Chilaka%' --as alomng with rajiv there could be many more directors for the same movie / tv show
```
**8. List all TV shows with more than 5 seasons**
 ```
select *, split_part(duration,' ',1)::numeric as seasons 
where type = 'TV Show' and split_part(duration,' ',1)::numeric > '5'
```
**9. Count the number of content items in each genre**
```
select unnest(string_to_array(listed_in,',')) as genre, count(show_id) as total_shows
from netflix
group by 1
order by 2 desc
```
**10. Find the average release year for content produced in a specific country**
```
select  unnest(string_to_array(country,',')) as country_name, round(avg(release_year), 0) from netflix
group by 1
order by 2 desc
```
**11. Find the average release year for content produced in a specific country, return top 5 year with hghest avg content release**
```
select 
	extract(year from to_date(date_added,'Month DD,YYYY')) as Content_added_on, 
	count(*),
	round(count(*):: numeric/(select count(*) from netflix where country = 'India')::numeric*100,0) as average_content_numbers
from netflix
group by 1
order by 3 desc
limit 5
```
**12. List all movies that are documentaries**
```
select *,unnest(string_to_array(listed_in,',')) as Genre_type from netflix
where type = 'Movie' and listed_in = 'Documentaries'
```
-- we can do it in this way too
```
select *
	from netflix
  	where type = 'Movie' and 
	listed_in Ilike '%documentaries%'
```
**13. Find all content without a director**
```
select *
	from netflix
	where director is Null
```
**14. Find how many movies actor 'Salman Khan' appeared in last 10 years!**
```
select *
	from netflix
	where casts ilike '%salman khan%' and 
	release_year >= extract(year from current_date) - 10 
```
**15. Find the top 10 actors who have appeared in the highest number of movies produced in india**
```
select unnest(string_to_array(casts,',')) as Actor_Actress, count(*) as Total_count
	from netflix
	where country ilike '%India%'
	group by 1
	order by 2 desc
	limit 10
```
**16.Categorize the content based on the presence of the keywords 'kill' and 'violence' in
the description field. Label content containing these keywords as 'Bad' and all other
content as 'Good'. Count how many items fall into each category.**
```
with new_table --cte clause to make new table
as
(
SELECT
*,
case	
when	
	description ilike '%kill%' 
	or description ilike '%violence%'
	then 'Bad_Content'
else
	'Good_Content'
	END category
from netflix)
select category, count(*)
from new_table
group by 1
order by 2 desc
```
