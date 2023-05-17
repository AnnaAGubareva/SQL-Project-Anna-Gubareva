What are your risk areas? Identify and describe them.



QA Process:
Describe your QA process and include the SQL queries used to execute it.
--1) Risk : duplicated values in the columns--

--Query to check--

SELECT (SKU."productSKU"), COUNT(SKU."productSKU")
FROM PUBLIC."SalesBySKU" AS SKU
GROUP BY SKU."productSKU"
HAVING COUNT (SKU."productSKU") > 1
ORDER BY SKU."productSKU";

--2) Risk: having missing data in the records--

--Query to check --

FROM PUBLIC."SalesBySKU"
WHERE "productSKU"IS NULL;

--3) Risk: not identical format of the data--

--Query to check--

Select sum(total_ordered)
from public."SalesBySKU"
WHERE "productSKU" not like 'GGO%'and length("productSKU")=7;

--4) Risk: high volume of the data--

--To check:(open everything in SQL and check if we have the same amount of records as in original CSV file)--

Select *
From PUBLIC."SalesBySKU";

--5) Risk: data presented in the wrong data type format--

--Query to check--

Select *
From PUBLIC."SalesBySKU";

--and then check if any record type should be changed (an example of changing type below)--

ALTER TABLE PUBLIC."SalesReport"
	ALTER COLUMN "sentimentScore" TYPE Decimal 
	USING "sentimentScore"::numeric;

--6) Risk: extracting data from csv files--

--To check select everything exported to SQL and compare the number of columns to CSV file--
--(also, CSV file cannot be imported to SQQL if we missing a column)--

--7) Risk : data is misspelled in the records--
--(hard to check this one, specifically if you don't know what you looking for. --
--Can be checked by looking for specific symbols (if they are known)--

--Query to check --

SELECT "productSKU"
FROM PUBLIC."SalesReport"
WHERE "productSKU" not like 'GG%'

--8) Risk: non-uniform use of units ( example: price was refleted in mils)--

--Query to correct--

unit_price::integer/1000000 as unit_price

--9) Risk: deleting usefull data--

--Query-- 
--re_import original files--

