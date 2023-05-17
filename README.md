# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
To retrieve, clean, and analyse data for future use.

## Process
### (your step 1)
### (your step 2)

Step1: Open CSV files and familiarize myself with data
Step2: Load data to PostgreSQL
Step3: Clean the data using PostgreSQL
Step4: Analyse the data to answer project questions
Step5: Finish the project using data from previous tasks 

## Results
(fill in what you discovered this data could tell you and how you used the data to answer those questions)

The data presented in different table was showing interaction of customers with the product.  I used the most “All Sessions” table to retrieve almost all the information which was asked in the project. Also, I noticed that data provided in “Sale_By_SKU” table is duplicated because the same records can be retrieved from ‘’Sales_Report” file. I have not used “Products” file in my project to answer the questions. However, I performed some data cleaning in this file.

## Challenges 
(discuss challenges you faced in the project)

•	Difficult to export the files to SQL because the majority data was in text format
•	Not every table contained column that can be Primary Key for this table
•	Some files contained empty or duplicated columns 
•	Some columns contain missing data
•	There was significant amount of unique characters ( different languages) in the name of website page
•	Data was provided for different years in two tables: AllSessions and Analysis; and thee other tables contains cumulative information without reference to the date of origin


## Future Goals
(what would you do if you had more time?)

Task which I want to accomplish
•	Remove/ substitute all Null values in the columns
•	Uniform data in the columns ( including changing type of the data)
•	Create/find primary key for every table to be able to relate tables between each other
•	Create a separate table(s) for all products with different name convention ( These products were out_of_stock and was not reordered)
•	Separate data into different tables based on the year ( specifically refer to All_Session table)

