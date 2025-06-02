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

Got it! Here’s your README section with your exact SQL code, cleanly formatted and organized—no changes to logic or queries, just structured neatly for GitHub:

⸻

9. SQL Analysis: Complex Queries and Business Problem Solving

This section includes complex SQL queries used to solve real-world business problems using the Walmart dataset.

⸻

1) Find the Different Payment Methods, Number of Transactions, and Quantity Sold

```sql
select 
payment_method,count(*) as no_payments,
sum(quantity) as no_of_sold
from walmart
group by payment_method
```

⸻

2) Identify the Highest-Rated Category in Each Branch, Displaying the Branch, Category, and Average Rating
```sql
select *
from
( select 
branch,
category,
avg(rating) as avg_rating,
rank() over(partition by branch order by avg(rating) desc) as rank
from walmart
group by 1,2
)
where rank =1

```
⸻

3) Identify the Branch Where It Has the Highest Number of Transactions
```sql
select *
from
(select
branch,
to_char(to_date(date,'DD/MM/YY'),'day') as format_date,
count(*) as no_trans,
rank() over(partition by branch order by count(*) desc) as rank
from walmart
group by 1,2
)
where rank =1

```
⸻

4) Calculate Total Quantity of Items Sold per Payment Method. List Payment Method and Total Quantity
```sql
select 
payment_method,count(*) as no_payments,
sum(quantity) as no_of_sold
from walmart
group by payment_method

```
⸻

5) Determine the Avg, Min, and Max Rating of Category for Each City
```sql
select 
city,category,
min(rating) as min_rating,
max(rating) as max_raing,
avg(rating) as avg_rating
from walmart
group by 1,2
order by 1,2 desc;

```
⸻

6) Calculate the Total Profit for Each Category by Considering Total Profit as (unit_price * quantity * profit_margin)
```sql
select 
category,
sum(total) as total_revenue,
sum(total * profit_margin) as profit
from walmart
group by 1

```
⸻

7) Determine the Most Common Payment Method for Each Branch
```sql
with cte
as
(select
branch,
payment_method,
count(*) as total_tran,
rank() over(partition by branch order by count(*) desc) as rank
from walmart
group by 1,2
)
select * 
from cte
where rank = 1

```
⸻

8) Categorize Sales into Morning, Afternoon, and Evening; Find Orders per Shift
```sql
SELECT
    branch,
    CASE 
        WHEN EXTRACT(HOUR FROM time::time) < 12 THEN 'morning'
        WHEN EXTRACT(HOUR FROM time::time) BETWEEN 12 AND 17 THEN 'afternoon'
        ELSE 'evening'
    END AS day_time,
    COUNT(*) AS total_orders
FROM walmart
GROUP BY 1,2
ORDER BY 1,3 DESC;

```
⸻

9) Identify 5 Branches with the Highest Decrease Ratio in Revenue Compared to Last Year (2023 vs 2022)
```sql
SELECT *,
EXTRACT(YEAR FROM TO_DATE(date, 'DD/MM/YY')) as formated_date
FROM walmart

-- 2022 sales
WITH revenue_2022
AS
(
	SELECT 
		branch,
		SUM(total) as revenue
	FROM walmart
	WHERE EXTRACT(YEAR FROM TO_DATE(date, 'DD/MM/YY')) = 2022 -- psql
	-- WHERE YEAR(TO_DATE(date, 'DD/MM/YY')) = 2022 -- mysql
	GROUP BY 1
),

revenue_2023
AS
(
	SELECT 
		branch,
		SUM(total) as revenue
	FROM walmart
	WHERE EXTRACT(YEAR FROM TO_DATE(date, 'DD/MM/YY')) = 2023
	GROUP BY 1
)

SELECT 
	ls.branch,
	ls.revenue as last_year_revenue,
	cs.revenue as cr_year_revenue,
	ROUND(
		(ls.revenue - cs.revenue)::numeric/
		ls.revenue::numeric * 100, 
		2) as rev_dec_ratio
FROM revenue_2022 as ls
JOIN
revenue_2023 as cs
ON ls.branch = cs.branch
WHERE 
	ls.revenue > cs.revenue
ORDER BY 4 DESC
LIMIT 5

```
⸻

Let me know if you want this exported to Markdown or used in a Jupyter Notebook!
   - **Documentation**: Keep clear notes of each query's objective, approach, and results.

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
