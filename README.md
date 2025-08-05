# sql_retail_sales_analysis
This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries.

## Objectives

1. **Set up a retail sales database**: Create and populate a retail sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project started by creating a database named `retail_sales_db`.
- **Table Creation**: A table named `retail_sales` is created to store the sales data. The table structure includes columns for transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity sold, price per unit, cost of goods sold (COGS), and total sale amount.

```sql
CREATE DATABASE retail_sales_db;

CREATE TABLE retail_sales
(
    transaction_id INT PRIMARY KEY,
    sale_date DATE,	
    sale_time TIME,
    customer_id INT,	
    gender VARCHAR(20),
    age INT,
    category VARCHAR(25),
    quantity INT,
    price_per_unit FLOAT,	
    cogs FLOAT,
    total_sale FLOAT
);
```

### 2. Data Exploration & Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Customer Count**: Find out how many unique customers are in the dataset.
- **Category Count**: Identify all unique product categories in the dataset.
- **Null Value Check**: Check for any null values in the dataset and delete records with missing data.

```sql
--Data exploration and cleaning

--record count 
SELECT COUNT(*) as total_records FROM retail_sales;

--customer count
SELECT COUNT(DISTINCT customer_id) as unique_cust_count FROM retail_sales;

--category count
SELECT count(DISTINCT category) as category_count FROM retail_sales;

--null record check
SELECT * FROM retail_sales
WHERE 
    sale_date IS NULL 
	OR sale_time IS NULL 
	OR customer_id IS NULL 
	OR gender IS NULL 
	OR age IS NULL 
	OR category IS NULL 
	OR quantity IS NULL 
	OR price_per_unit IS NULL 
	OR cogs IS NULL
	or total_sale IS NULL;

----------------------------------------------------------------------------------------

--cleaning null rows

--deleting rows with null quantity,price,cogs,total sales
DELETE FROM retail_sales
WHERE 
    quantity IS NULL 
	OR price_per_unit IS NULL 
	OR cogs IS NULL
	or total_sale IS NULL;

--updating missing age value with avg age of category
WITH category_avg AS (
    SELECT category, ROUND(AVG(age),0) AS avg_age
    FROM retail_sales
    WHERE age IS NOT NULL
    GROUP BY category
)

UPDATE retail_sales
SET age = (
    SELECT avg_age
    FROM category_avg
    WHERE category_avg.category = retail_sales.category
)
WHERE age IS NULL;
```
