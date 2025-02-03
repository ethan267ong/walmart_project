# Walmart Data Analysis: End-to-End SQL + Python Project P-9

## Project Overview

![Project Pipeline](https://github.com/najirh/Walmart_SQL_Python/blob/main/walmart_project-piplelines.png)


This project is an end-to-end data analysis solution designed to extract critical business insights from Walmart sales data. We utilize Python for data processing and analysis, SQL for advanced querying, and structured problem-solving techniques to solve key business questions. The project is ideal for data analysts looking to develop skills in data manipulation, SQL querying, and data pipeline creation.

---

## Project Steps

### 1. Set Up the Environment
   - **Tools Used**: Visual Studio Code (VS Code), Python, SQL (MySQL and PostgreSQL)
   - **Goal**: Create a structured workspace within VS Code and organize project folders for smooth development and data handling.

### 2. Set Up Kaggle API
   - **API Setup**: Obtain your Kaggle API token from [Kaggle](https://www.kaggle.com/) by navigating to your profile settings and downloading the JSON file.
   - **Configure Kaggle**: 
      - Place the downloaded `kaggle.json` file in your local `.kaggle` folder.
      - Use the command `kaggle datasets download -d <dataset-path>` to pull datasets directly into your project.

### 3. Download Walmart Sales Data
   - **Data Source**: Use the Kaggle API to download the Walmart sales datasets from Kaggle.
   - **Dataset Link**: [Walmart Sales Dataset](https://www.kaggle.com/najir0123/walmart-10k-sales-datasets)
   - **Storage**: Save the data in the `data/` folder for easy reference and access.

### 4. Install Required Libraries and Load Data
   - **Libraries**: Install necessary Python libraries using:
     ```bash
     pip install pandas numpy sqlalchemy mysql-connector-python psycopg2
     ```
   - **Loading Data**: Read the data into a Pandas DataFrame for initial analysis and transformations.

### 5. Explore the Data
   - **Goal**: Conduct an initial data exploration to understand data distribution, check column names, types, and identify potential issues.
   - **Analysis**: Use functions like `.info()`, `.describe()`, and `.head()` to get a quick overview of the data structure and statistics.

### 6. Data Cleaning
   - **Remove Duplicates**: Identify and remove duplicate entries to avoid skewed results.
   - **Handle Missing Values**: Drop rows or columns with missing values if they are insignificant; fill values where essential.
   - **Fix Data Types**: Ensure all columns have consistent data types (e.g., dates as `datetime`, prices as `float`).
   - **Currency Formatting**: Use `.replace()` to handle and format currency values for analysis.
   - **Validation**: Check for any remaining inconsistencies and verify the cleaned data.

### 7. Feature Engineering
   - **Create New Columns**: Calculate the `Total Amount` for each transaction by multiplying `unit_price` by `quantity` and adding this as a new column.
   - **Enhance Dataset**: Adding this calculated field will streamline further SQL analysis and aggregation tasks.

### 8. Load Data into MySQL and PostgreSQL
   - **Set Up Connections**: Connect to MySQL and PostgreSQL using `sqlalchemy` and load the cleaned data into each database.
   - **Table Creation**: Set up tables in both MySQL and PostgreSQL using Python SQLAlchemy to automate table creation and data insertion.
   - **Verification**: Run initial SQL queries to confirm that the data has been loaded accurately.

### 9. SQL Analysis: Complex Queries and Business Problem Solving
   - **Business Problem-Solving**: Write and execute complex SQL queries to answer critical business questions, such as:
     - Revenue trends across branches and categories.
     - Identifying best-selling product categories.
     - Sales performance by time, city, and payment method.
     - Analyzing peak sales periods and customer buying patterns.
     - Profit margin analysis by branch and category.
   - **Documentation**: Keep clear notes of each query's objective, approach, and results.
```
-- BUSINESS PROBLEMS
-- Q1. Find the different payment method and number of transactions, number of qty sold
SELECT 
	DISTINCT payment_method,
    COUNT(*) as no_payments,
    SUM(quantity) as no_qty_sold
FROM walmart
GROUP BY payment_method;

-- Q2. Identify the highest-rated category in each branch, disoplaying the branch, category, avg rating
SELECT * FROM 
(SELECT 
	Branch,
    Category,
    AVG(rating) as  avg_rating,
    RANK() OVER(PARTITION BY Branch ORDER BY AVG(rating) DESC) as ranking
FROM WALMART
GROUP BY 1,2) AS ranked_data WHERE ranking = 1;

-- Q3. Identify the busiest day for each branch based on the number of transactions
SELECT * FROM (
    SELECT  
        Branch,  
        DATE_FORMAT(STR_TO_DATE(date, '%d/%m/%y'), '%W') AS day_name,  
        COUNT(*) AS no_transactions,  
        RANK() OVER(PARTITION BY Branch ORDER BY COUNT(*) DESC) AS ranking  
    FROM walmart  
    GROUP BY 1,2
) AS ranked_data  
WHERE ranking = 1;

-- Q4. Calculated the total quantity of items sold per payment method. List payment_method and total_quantity
SELECT 
	payment_method,
    SUM(quantity) as no_qty_sold
FROM walmart
GROUP BY payment_method;

-- Q5. Determine the average, minimum, and maximum rating of products for each city. List the city, average_rating, min_rating, and max_rating
SELECT 
	city,
    category,
    MIN(rating) as min_rating,
    MAX(rating) as max_rating,
    AVG(rating) as avg_rating
FROM walmart
GROUP BY 1,2;

-- Q6. Calculate the Total Revenue for Each Branch
SELECT 
    branch, 
    SUM(total) AS total_revenue
FROM walmart
GROUP BY branch;

-- Q7. Find the Branch with the Highest Average Rating
SELECT 
    branch, 
    AVG(rating) AS avg_rating
FROM walmart
GROUP BY branch
ORDER BY avg_rating DESC
LIMIT 1;

-- Q8. Count the Number of Transactions by Payment Method for Each Branch
SELECT 
    branch, 
    payment_method, 
    COUNT(*) AS transaction_count
FROM walmart
GROUP BY branch, payment_method;

-- Q9. Calculate the Total Profit for Each Category (Based on Profit Margin and Total Sales)
SELECT 
    category, 
    SUM(total * profit_margin) AS total_profit
FROM walmart
GROUP BY category
ORDER BY total_profit DESC;

-- Q10. Get the Day of the Week with the Highest Number of Transactions for Each Branch
SELECT 
    branch,
    DAYNAME(STR_TO_DATE(date, '%d/%m/%y')) AS day_of_week,
    COUNT(*) AS transaction_count
FROM walmart
GROUP BY branch, day_of_week
ORDER BY branch, transaction_count DESC;

-- Q11. Calculate the total proft for each category by considering total_profit as (unit_price * quantity * profit_margin)
-- List category and total_profit, ordered from highest to lowest profit
SELECT
	category,
    SUM(total) as total_revenue,
    SUM(total * profit_margin) as profit
FROM walmart
GROUP BY 1;

-- Q12. Determine the most common payment method for each branch. Display branch and the preferred payment method
WITH cte AS (
    SELECT 
        branch,
        payment_method,
        COUNT(*) AS total_trans,
        RANK() OVER (PARTITION BY branch ORDER BY COUNT(*) DESC) AS ranking
    FROM walmart
    GROUP BY branch, payment_method
)
SELECT * FROM cte WHERE ranking = 1;

-- Q13. Categorise sales into 3 groups morning, afternoon and evening. find out each of the shift and number of invoices
SELECT branch,
    CASE 
        WHEN HOUR(TIME(time)) < 12 THEN 'Morning'
        WHEN HOUR(TIME(time)) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS day_time,
    COUNT(*)
FROM walmart
GROUP BY 1,2
ORDER BY 1,3 DESC;

-- Q14. Identify 5 branches with highest decrease in revenue from 2022 to 2023
WITH revenue_2022 AS (
    SELECT branch, 
           SUM(total) AS revenue 
    FROM walmart 
    WHERE YEAR(STR_TO_DATE(date, '%d/%m/%y')) = 2022 
    GROUP BY branch
),
revenue_2023 AS (
    SELECT branch, 
           SUM(total) AS revenue 
    FROM walmart 
    WHERE YEAR(STR_TO_DATE(date, '%d/%m/%y')) = 2023 
    GROUP BY branch
)
SELECT 
	previous_year.branch,
    previous_year.revenue AS previous_year_revenue,
    current_year.revenue AS current_year_revenue,
    ROUND((previous_year.revenue - current_year.revenue) / previous_year.revenue * 100,2) as difference
FROM revenue_2022 AS previous_year 
JOIN revenue_2023 AS current_year 
ON previous_year.branch = current_year.branch
WHERE previous_year.revenue > current_year.revenue
ORDER BY difference DESC
LIMIT 5;
```

### 10. Project Publishing and Documentation
   - **Documentation**: Maintain well-structured documentation of the entire process in Markdown or a Jupyter Notebook.
   - **Project Publishing**: Publish the completed project on GitHub or any other version control platform, including:
     - The `README.md` file (this document).
     - Jupyter Notebooks (if applicable).
     - SQL query scripts.
     - Data files (if possible) or steps to access them.

---

## Requirements

- **Python 3.8+**
- **SQL Databases**: MySQL, PostgreSQL
- **Python Libraries**:
  - `pandas`, `numpy`, `sqlalchemy`, `mysql-connector-python`, `psycopg2`
- **Kaggle API Key** (for data downloading)

## Getting Started

1. Clone the repository:
   ```bash
   git clone <repo-url>
   ```
2. Install Python libraries:
   ```bash
   pip install -r requirements.txt
   ```
3. Set up your Kaggle API, download the data, and follow the steps to load and analyze.

---

## Project Structure

```plaintext
|-- data/                     # Raw data and transformed data
|-- sql_queries/              # SQL scripts for analysis and queries
|-- notebooks/                # Jupyter notebooks for Python analysis
|-- README.md                 # Project documentation
|-- requirements.txt          # List of required Python libraries
|-- main.py                   # Main script for loading, cleaning, and processing data
```
---

## Results and Insights

This section will include your analysis findings:
- **Sales Insights**: Key categories, branches with highest sales, and preferred payment methods.
- **Profitability**: Insights into the most profitable product categories and locations.
- **Customer Behavior**: Trends in ratings, payment preferences, and peak shopping hours.

## Future Enhancements

Possible extensions to this project:
- Integration with a dashboard tool (e.g., Power BI or Tableau) for interactive visualization.
- Additional data sources to enhance analysis depth.
- Automation of the data pipeline for real-time data ingestion and analysis.

---

## License

This project is licensed under the MIT License. 

---

## Acknowledgments

- **Data Source**: Kaggle’s Walmart Sales Dataset
- **Inspiration**: Walmart’s business case studies on sales and supply chain optimization.

---
