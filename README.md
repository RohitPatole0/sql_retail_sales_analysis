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
### 3. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

**A.Customer Behavior & Segmentation**

**Customer Lifetime Value (CLV)**

**Calculate the total sales made by each customer and identify the top 10 most valuable customers**:
Calculate the total sales made by each customer and identify the top 10 most valuable customers.

```sql
SELECT customer_id, SUM(total_sale) as total_spent
FROM retail_sales
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 10;
```

**Customer Segmentation by Age Group & Gender**:
Create customer segments (e.g., 18–25, 26–35, 46-60).

```sql
SELECT 
    CASE 
        WHEN age BETWEEN 18 AND 25 THEN '18-25'
        WHEN age BETWEEN 26 AND 35 THEN '26-35'
        WHEN age BETWEEN 36 AND 45 THEN '36-45'
        WHEN age BETWEEN 46 AND 60 THEN '46-60'
        ELSE '60+' 
    END AS age_group,
    gender,
    COUNT(DISTINCT customer_id) AS customer_count,
    AVG(total_sale) AS avg_order_value,
    COUNT(*) AS total_transactions
FROM retail_sales
GROUP BY age_group, gender
ORDER BY avg_order_value DESC;
```

**Repeat Customers vs. One-Time Shoppers**:
Identify customers with more than one transaction and compare their average purchase size to one-time buyers.

```sql
WITH customer_order_counts AS (
    SELECT customer_id, COUNT(*) AS order_count
    FROM retail_sales
    GROUP BY customer_id
)
SELECT 
    CASE 
        WHEN order_count = 1 THEN 'One-Time'
        ELSE 'Repeat'
    END AS customer_type,
    COUNT(*) AS num_customers,
    AVG(rs.total_sale) AS avg_order_value
FROM customer_order_counts coc
JOIN retail_sales rs ON coc.customer_id = rs.customer_id
GROUP BY customer_type;
```

**B.Time-Based Sales Trends**

**Monthly Sales Trend**:
Aggregate total sales by month to identify peak months.

```sql
SELECT 
    EXTRACT(MONTH FROM sale_date::date) AS month,
	COUNT(*) as no_of_sales,
    SUM(total_sale) AS monthly_sales
FROM retail_sales
GROUP BY month
ORDER BY monthly_sales DESC;
```

Hourly Sales Analysis:
Determine the busiest hours of the day and compare morning vs. evening sales patterns.

```sql
WITH hourly_sales as (
    SELECT *, 
		EXTRACT(HOUR FROM sale_time::TIME) as sale_hour
	FROM retail_sales
),

day_part_sales as (
	SELECT *,
		CASE 
			WHEN sale_hour < 16 THEN 'Morning'
			ELSE 'Evening'
		END as part_of_day
FROM hourly_sales
)

--Busiest hours
SELECT sale_hour,
	    COUNT(*) AS transactions,
    SUM(total_sale) AS total_sales
FROM hourly_sales
GROUP BY sale_hour
ORDER BY transactions DESC;

--morning vs evening sales
SELECT
	part_of_day,
	COUNT(*) as transactions
FROM day_part_sales
GROUP BY part_of_day;

--comment out one of the queries to execute
```

**Weekday vs. Weekend Sales Performance**:
Classify days as weekday/weekend. Compare total sales, average quantity, and customer count.

```sql
SELECT 
	CASE
		WHEN EXTRACT('dow' FROM sale_date) IN (0,6) THEN 'Weekend'
		ELSE 'Weekday'
	END AS day_of_week,
	SUM(total_sale) as total_sales,
	AVG(quantity) as avg_quantity,
	COUNT(customer_id) as customer_count
FROM retail_sales
GROUP BY day_of_week
```

**C.Product Category Insights**

**Top Performing Product Categories**:
Rank categories by total sales, average price, and quantity sold.

```sql
SELECT category,
	SUM(total_sale) as total_sales,
	AVG(price_per_unit) as avg_price,
	SUM(quantity) as quanty_sold
FROM retail_sales
GROUP BY category
ORDER BY total_sales DESC;
```

**Profitability by Category**:
Calculate gross profit (total_sale - cogs) for each product category.

```sql
SELECT category,
	SUM(total_sale - cogs) AS gross_profit,
	AVG(total_sale - cogs) AS avg_profit_per_sale
FROM retail_sales
GROUP BY category
ORDER BY gross_profit;
```

**Basket Size by Category**:
Find the average quantity purchased per transaction for each category.

```sql
SELECT category,
	AVG(quantity) as avg_basket_size
FROM retail_sales
GROUP BY category
ORDER BY avg_basket_size DESC;
```

**D.Financial & Profitability Metrics**

**Gross Margin per Transaction**:
For each transaction, calculate the gross margin percentage:((total_sale - cogs) / total_sale) * 100

```sql
SELECT transaction_id,
       total_sale,
       cogs,
	   (((total_sale - cogs) / total_sale) * 100) as gross_margin_perc
FROM retail_sales
ORDER BY gross_margin_perc DESC;
```

**High Margin vs. Low Margin Sales**:
Classify transactions based on gross margin thresholds (e.g., high > 40%) and count frequency.

```sql
SELECT 
    CASE 
		WHEN (((total_sale - cogs) / total_sale) * 100) > 40 THEN 'High Margin'
		ELSE 'Low Margin'
	END as margin,
	COUNT(*) AS transaction_count
FROM retail_sales
GROUP BY margin;
```

**E.Advanced Analytics**

**RFM Analysis (Recency, Frequency, Monetary)**:

```sql
WITH rfm_base AS (
    SELECT customer_id,
           MAX(sale_date::DATE) AS last_purchase,
           COUNT(*) AS frequency,
           SUM(total_sale) AS monetary
    FROM retail_sales
    GROUP BY customer_id
),
--Assuming max sale date is recent date
reference AS (
    SELECT MAX(sale_date::DATE) AS max_date FROM retail_sales
)
SELECT r.customer_id,
       (SELECT max_date FROM reference) - r.last_purchase AS recency,
       frequency,
       monetary
FROM rfm_base r;
```

**Rolling Monthly Average Sales**:
Calculate 3-month rolling average sales.

```sql
SELECT 
    EXTRACT(MONTH FROM sale_date::DATE) AS month,
    category,
    SUM(total_sale) AS monthly_sales,
    AVG(SUM(total_sale)) OVER (
        PARTITION BY category 
        ORDER BY EXTRACT(MONTH FROM sale_date::DATE)
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS rolling_avg_sales
FROM retail_sales
GROUP BY month, category
ORDER BY month;	
```

**Rank Sales by Customer**:
Rank each transaction per customer by total_sale descending.

```sql
SELECT *,
       RANK() OVER (PARTITION BY customer_id ORDER BY total_sale DESC) AS sale_rank
FROM retail_sales;
```

**F.Data Quality & Anomalies**

**Detect Outliers**:
Identify transactions with unusually high or low quantity, price, or total_sale using percentiles.

```sql
SELECT *
FROM retail_sales
WHERE quantiy > (
    SELECT PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY quantiy) FROM retail_sales
)
OR price_per_unit > (
    SELECT PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY price_per_unit) FROM retail_sales
);
```

**Customer Age Outliers**:
Check for unreasonable ages (e.g., < 18 or > 90).

```sql
SELECT 
    COUNT(*) AS invalid_ages,
    AVG(age) AS avg_age,
    category
FROM retail_sales
WHERE age < 18 OR age > 90
GROUP BY category;
```