What issues will you address by cleaning the data?
--duplicated values in the columns
--having missing data in the records
--not identical format of the data
--high volume of the data
--data presented in the wrong data type format
--extracting data from csv files
--data is misspelled in the records
--non-uniform use of units ( example: price was refleted in mils)




Queries:
Below, provide the SQL queries you used to clean your data.
--SalesBySKU table
--1) to examen the table

SELECT *
FROM PUBLIC."SalesBySKU"

--2)  to check if 'productSKU' column has uniquie values

SELECT (SKU."productSKU"), COUNT(SKU."productSKU")
FROM PUBLIC."SalesBySKU" AS SKU
GROUP BY SKU."productSKU"
HAVING COUNT (SKU."productSKU") > 1
ORDER BY SKU."productSKU";

--3)check if "productSKU" column has NULL values

SELECT *
FROM PUBLIC."SalesBySKU"
WHERE "productSKU"IS NULL;


--4)set "productSKU" column as Primary Key

 ALTER TABLE IF EXISTS PUBLIC."SalesBySKU" 
 ADD PRIMARY KEY ("productSKU");
		
--5) check if all "productSKU",Primary Key, is in the same format
	--checking lenth of the string

SELECT LENGTH("productSKU") AS LENGTHOFSTRING,
	COUNT(LENGTH("productSKU")) AS COUNTOFCASES
FROM PUBLIC."SalesBySKU"
GROUP BY LENGTH("productSKU");

--6) exame the table to invistigate the difference in lenth for string
Select sum(total_ordered)
from public."SalesBySKU"
WHERE "productSKU" not like 'GGO%'and length("productSKU")=7;

-- All orders for ProductSKU are zero and productSKU id is in different format

--7) "SalesBySKU" table after cleaning data
SELECT *
FROM PUBLIC."SalesBySKU"
WHERE LENGTH("productSKU") > 7;

--SalesReport Table
--1) select everything to exam the table

SELECT*
FROM PUBLIC."SalesReport";

--2)check productSKU column for duplicated values and for NULL values

SELECT "productSKU",
	COUNT("productSKU") AS COUNT
FROM PUBLIC."SalesReport"
WHERE "productSKU" IS NULL
GROUP BY "productSKU"
HAVING COUNT("productSKU") > 1;

--3)set productSKU column as Primary Key
ALTER TABLE IF EXISTS public."SalesReport"
    ALTER COLUMN "productSKU" SET NOT NULL;
ALTER TABLE IF EXISTS public."SalesReport"
    ADD PRIMARY KEY ("productSKU");

--4) perfome checks for "productSKU" which has different format
 --checking if the data has any value assigned

SELECT "productSKU"
FROM PUBLIC."SalesReport"
WHERE "productSKU" not like 'GG%'
GROUP BY "productSKU",
	TOTAL_ORDERED
HAVING TOTAL_ORDERED > 0
OR "stockLevel" > 0;

--productSKU with which do not have 'GG' in their name convention are irrelevant for this table
	
--5) filter out from 'productSKU' column SKUs which are in different format

SELECT *
FROM PUBLIC."SalesReport"
WHERE "productSKU" like 'GG%';

--6) change columns type to numeric

ALTER TABLE PUBLIC."SalesReport"
	ALTER COLUMN "sentimentScore" TYPE Decimal 
	USING "sentimentScore"::numeric;

ALTER TABLE PUBLIC."SalesReport"
	ALTER COLUMN "sentimentMagnitude" TYPE Decimal 
	USING "sentimentMagnitude"::numeric;

ALTER TABLE PUBLIC."SalesReport"
	ALTER COLUMN ratio TYPE Decimal 
	USING ratio::numeric;

--7)check the range of the Ratio column -> expectations that it should be within 100%

SELECT *
FROM public."SalesReport"
WHERE ratio >1;

-- this query returned two results
-- column ratio calculated as "total_ordered"/"stockLevel"
-- two rows from this query should be investigated further 
--		since "ratio" for themis more than 100%	

--8)create CTE to change representation of ratio column from from digits to percentage

WITH CTE_RATIOPCT AS(
SELECT 
	*,
	CONCAT(ROUND((RATIO * 100),4),'%') AS RATIOPCT
FROM PUBLIC."SalesReport" AS RATIO
)

--9) final table after data cleaning
--		filter out ProductSKU without 'GG' in the name convention
--     	        to represent column ratio in %
--		to change the order of the columns for better representation
SELECT *
FROM CTE_RATIOPCT
WHERE "productSKU" like 'GG%';

--Products table
SELECT *
FROM PUBLIC."Products";

--1)check SKU column for unique values and not NUll values

SELECT "SKU",
	COUNT("SKU") AS COUNT
FROM public."Products"
WHERE "SKU" IS NULL
GROUP BY "SKU"
HAVING COUNT("SKU") > 1;

--2) set SKU column as primary key

ALTER TABLE IF EXISTS public."Products"
    ALTER COLUMN "SKU" SET NOT NULL;
ALTER TABLE IF EXISTS public."Products"
    ADD PRIMARY KEY ("SKU");
	
--3)  check data for SKU which does not contain 'GG' in the name

SELECT *
FROM PUBLIC."Products"
WHERE "SKU" not like 'GG%'
GROUP BY "SKU"
HAVING "orderedQuantity" > 0
OR "stockLevel" > 0;

-- products whcih do not have 'GG' in SKU are irrelavant
-- 	and will be filter out

--4) if 'sentimentScore' and 'sentimentMagnitude'columns has Null values

SELECT "sentimentScore",
	"sentimentMagnitude"
FROM PUBLIC."Products"
WHERE "sentimentScore" IS NULL
	OR "sentimentMagnitude" IS NULL;

-- the results give back a row. 
--	This wil requer more attention if we will use this data later in the analysis

--5) final result of the table after cleaning process

SELECT *
FROM PUBLIC."Products"
WHERE "SKU" like 'GG%';

--AllSessions table--
SELECT *
FROM PUBLIC."AllSessions";

--1)change "fullVisitorId" column type to decimal

ALTER TABLE PUBLIC."AllSessions"
ALTER COLUMN "fullVisitorId" 
	TYPE decimal USING "fullVisitorId"::numeric;
	
--2) check if fullVisitorId has unique, not NUll values

select "visitId", count ("visitId")
from public."AllSessions"
where "visitId" is not Null
group by "visitId"
having count ("visitId")>1
order by count ("visitId") desc;

select "fullVisitorId", count ("fullVisitorId")
from public."AllSessions"
where "fullVisitorId" is not Null
group by "fullVisitorId"
having count ("fullVisitorId")>1
order by count ("fullVisitorId") desc;


--ANALYTICS Table--and ALLSession table --combined--

--1)check the date ranage for Analytics table

SELECT date
FROM PUBLIC."Analytics"
GROUP BY date
ORDER BY date ASC

-- result indicated that the earliest date in the file is '20170501'

--2) check if date column has any Null values

SELECT date
FROM PUBLIC."Analytics"
WHERE date IS NULL

-- AllSessions Table--

--3) check the erliest date in the range

SELECT date
FROM PUBLIC."AllSessions"
GROUP BY date
ORDER BY date ASC

-- the result showed '20160801' is the earliest date and '20170801' the last one

--4) deleted all the rows which have the same VisitID,Unit_Price and Date
-- steps a)join table to itself
---------b)checked if two records Dup."visitId" < Orig."visitId" has the same value
---------c)If yes, then the duplicated record will be deleted

DELETE FROM
public."Analytics" as Dup
USING public."Analytics" as Orig
WHERE Dup."visitId" < Orig."visitId"
AND Dup.unit_price = Orig.unit_price
AND Dup.date = Orig.date

-- 5) join tables:AllSessions and Analytics based only on 2017 since there is common data for this year  
-- 	a) filtering out other years but 2017 and making Primary Key by combining VisitID and Date


CREATE VIEW  Analyt_AllSessions as

--1st CTE table--

	WITH CTE_PrimaryKeyAll as
	(
		SELECT CONCAT("visitId",date) AS PrimaryKey,
		time,
		country,
		city,
		"totalTransactionRevenue",
		"timeOnSite",
		pageviews,
		date,
		"visitId",
		"productPrice",
		"productRevenue",
		"productSKU",
		"v2ProductName",
		"currencyCode"
		FROM PUBLIC."AllSessions"
		WHERE date like '2017%' 
	  ),

--2nd CTE table--
--converting price into integer and deviding by 1,000,000--
	
    	CTE_PrimaryKeyAnalyt as
	(
		SELECT distinct(CONCAT("visitId",date)) AS PK,
		"visitNumber",
		units_sold,
		timeonsite,
		revenue,
		unit_price::integer/1000000 as unit_price,
		avg (unit_price::integer/1000000)
		over (partition by CONCAT("visitId",date)  ) as avg		
		FROM public."Analytics"
		
		
	 )
	
--  b) join both results

select *
from CTE_PrimaryKeyAll
join CTE_PrimaryKeyAnalyt 
on CTE_PrimaryKeyAll.PrimaryKey = CTE_PrimaryKeyAnalyt.PK;







