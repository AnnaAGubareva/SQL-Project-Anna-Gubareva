Question 1: 
--1) Calcualte total time on site spent over years by customers from USA--

SQL Queries:

select country, to_char(totaltime,'999G999G999') as totalT
from
(
select country,
sum("timeOnSite" ::numeric) over () as TotalTime
from public."AllSessions"
where country like 'United States' and "timeOnSite" is not null
limit 1
) as Totaltime;

Answer: 
-- Answer--- Country United States TotalTime 1,597,764--


Question 2: 
--2) Avg number of pageViews per session for 2017--

SQL Queries:

select pageviews, round(avg(pageviews::integer) over (),0)
from public."AllSessions"
where left(date,4) like ('%2017%')
group by pageviews;

Answer:
--Answer: Around 12 pages were viewed per session in 2017--


Question 3: 
--3)Show the product which has the most interaction ( most viewed) --

SQL Queries:

Select "v2ProductName","productSKU", count("productSKU") as PrCount
from
(
	select "v2ProductName","productSKU","fullVisitorId",
	count("fullVisitorId") as countId
	from public."AllSessions"
	group by "fullVisitorId","productSKU","v2ProductName"
	having count("fullVisitorId")=1
	order by "productSKU"
) as UniqueVisitor
group by "v2ProductName","productSKU"
order by PrCount desc
limit 1;

Answer:

--Answer: Google Men 100% cotton short was viewed the most - 282 times--



Question 4: 

SQL Queries:

Answer:



Question 5: 

SQL Queries:

Answer:
