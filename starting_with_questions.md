Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

SELECT CITY,
	COUNTRY,
	TO_CHAR("totalTransactionRevenue"::integer,'999G999G999G999G999')as revenue
FROM PUBLIC.ANALYT_ALLSESSIONS
WHERE "totalTransactionRevenue" IS NOT NULL
ORDER BY "totalTransactionRevenue" DESC
LIMIT 1



Answer:

------City -> Sunnyvale--
------Country -> UnitedStates--
------Revenue -> 649,240,000--


**Question 2: What is the average number of products ordered from visitors in each city and country?**

--steps-- 
---a)change column type to interger--
---b)found amount of prodocts orderd per country and city using subquery--
-----(in subquery) filter out "not set" values for country and city since they are not relavant to question--
---c) then take arg from this amount--
----- filter out countries and cities where count of products =0--



SQL Queries:

ALTER TABLE PUBLIC."AllSessions"
ALTER COLUMN "productQuantity" TYPE numeric 
USING "productQuantity"::integer

SELECT COUNTRY,
	CITY,
	AVG(COUNT)::numeric (4,2)
FROM
	(
		SELECT COUNTRY,
			CITY,
			COUNT("productQuantity")AS COUNT
		FROM PUBLIC."AllSessions"
		WHERE COUNTRY not like '(%not set%)'
			AND CITY not like '(%not set%)'
		GROUP BY COUNTRY,CITY
		ORDER BY COUNTRY
	) AS SUBQ
WHERE COUNT != 0
GROUP BY COUNTRY,CITY
ORDER BY COUNTRY ASC,CITY ASC



Answer:
--Answer: 26 rows --
----------United States has the majority of cities where purchases where made.--
---------- Howevre, the highest avg of purchases is from city "not available in demo dataset"--



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

---steps--
----a)in AllSessions table filter out informations which was not provided--
------- "v2ProductCategory" column filter out "not set" and "undetermined" categories--
-----b)separate product category from the column (CTE_Filtering)--
--------1) replace 'Home/' with ''--
--------2) position '/' which not removed in step 1--
--------3) take everything to the left of step #2--
------c)select country, city, categories, and count categories after per country ( window function)--


SQL Queries:

WITH CTE_FILTERING AS
	(SELECT *,
			LEFT(REPLACE("v2ProductCategory",'Home/',''),
				POSITION ('/' in REPLACE("v2ProductCategory",'Home/','')) -1) AS CATEGORY
	 FROM
---filter out categories with "bad" name
	 		(
				SELECT *
				FROM PUBLIC."AllSessions"
				WHERE "v2ProductCategory" not like '%(not set)%'
					AND "v2ProductCategory" not like '%{%' 
			)AS SUBQ
	)
			
SELECT *,
	COUNT(CATEGORY) OVER (PARTITION BY COUNTRY ORDER BY COUNTRY) AS CATEGORYCOUNT
FROM
	(
		SELECT COUNTRY,
			CITY,
			CATEGORY
		FROM CTE_FILTERING
			WHERE COUNTRY not like '%(not set)%'
			AND COUNTRY not like '%(not set)%'
		GROUP BY COUNTRY,CITY,CATEGORY
		ORDER BY COUNTRY ASC, CITY ASC, CATEGORY ASC
	) AS COUNTRYCITYCATEGORY
	
ORDER BY  CATEGORYCOUNT DESC,CATEGORY DESC,COUNTRY DESC


Answer:

--Answer--there is a table with 1571 entries. The majority purchases from Uninited States.--
----- PS: Im not satisfied by the table presentation. If i have time I would spend it more in analyzing the result of it--


**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

select country,"v2ProductName",countProduct
from
(
	select country,city, "v2ProductName","productSKU",
		count ("productSKU") over (partition by "productSKU" order by country) as countProduct
	from public."AllSessions"
	where country not like '%not set%' and city not like '%not set%'
	order by country asc, city asc, "v2ProductName" asc
) as CountryProduct
where countproduct >200
group by country,"v2ProductName",countProduct
order by country desc, countproduct desc


Answer:
--Assumption: i filtered out the products which is below 200 count to reduce the list--
----top countries for selling over 200 items of product are United States, Uruguay, Vietnam--
---- Top products to sell in these countries  YouTub Twill Cap, Google Men's cotton short,and YouTube custom Decals--


**Question 5: Can we summarize the impact of revenue generated from each city/country?**

---steps--
  --when i selected every record where Product revenue is not Null , i got only 4 rows.--
  -- so to answer this uestion i will need more information on the revenue in each country in each year.--
  --below is query for current data--

SQL Queries:

select country,year,
	sum("productRevenue") over (partition by year) as Revenue
from
(
select country,
  date_part('year',date::date)as year,
  	"productRevenue"::integer
  from public."AllSessions"
  where "productRevenue" is not null
) as USAProdRev


Answer:
-- the result shows United States with total revenue in 2016 and 2017 for avalable data--







