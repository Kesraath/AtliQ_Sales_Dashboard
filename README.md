# AtliQ_Sales_Dashboard

### Project Overview

A sales manager needs help grasping the sales situation from his regional managers and receives nothing but convoluted spreadsheets. He wants one automated central location that allows him to understand the monthly sales data and trends for each region: North, Central, and South. The sales manager can then make data-driven decisions for improving the business in areas that aren’t doing as well using this tool. 

### Data Source

The data is from a "db_dump.sql" file from [codebasics](https://codebasics.io/resources/sales-insights-data-analysis-project)

It includes the sales information from a hypothetical Hardware that supplies hardware to many resellers in India. These stores are split into 3 regional hubs: North, Central and South. 

Although this is hypothetical data, the data will be assumed to be sales data provided directly by the company. As such, the reliability of such data is very high as it is a primary source of data from within the company.

The data consists of sales data from October 2017 to June 2020

Link to all [Project Files](https://github.com/Kesraath/AtliQ_Sales_Dashboard/tree/AtliQProjectFiles)

### Tools

mySQL - Used to clean and prep the data
Tableau Desktop - Used for creating the visualisations

### Data Cleaning & Preparation

To resolve the sales managers' issue, creating a dashboard will be the most straightforward method for them to get the information they need and make data-driven decisions. 
A dashboard is preferable here as it needs to be live. It simplifies the spreadsheet data they received before into digestible visualisations and can be updated at the end of each month as requested by the sales manager.

After importing the data directly into MySQL with server data import and loading the .sql file in directly. We are given a schema with 5 tables inside,
- customers
- date
- markets
- products
- transactions

Using the quick SELECT Rows shortcut on the right of the table names in the schema we get the following results (Limit first 5 rows)

cutomers:
|customer_code|custmer_name            |customer_type |
|-------------|------------------------|--------------|
|Cus001       |Surge Stores            |Brick & Mortar|
|Cus002       |Nomad Stores            |Brick & Mortar|
|Cus003       |Excel Stores            |Brick & Mortar|
|Cus004       |Surface Stores          |Brick & Mortar|
|Cus005       |Premium Stores          |Brick & Mortar|

date:
date  |cy_date                 |year          |month_name|date_yy_mmm|
|------|------------------------|--------------|----------|-----------|
|2017-06-01|2017-06-01              |2017          |June      |17-Jun     |
|2017-06-02|2017-06-01              |2017          |June      |17-Jun     |
|2017-06-03|2017-06-01              |2017          |June      |17-Jun     |
|2017-06-04|2017-06-01              |2017          |June      |17-Jun     |
|2017-06-05|2017-06-01              |2017          |June      |17-Jun     |

markets:
|markets_code|markets_name            |zone          |
|------------|------------------------|--------------|
|Mark001     |Chennai                 |South         |
|Mark002     |Mumbai                  |Central       |
|Mark003     |Ahmedabad               |North         |
|Mark004     |Delhi NCR               |North         |
|Mark005     |Kanpur                  |North         |

products:
|product_code|product_type            |
|------------|------------------------|
|Prod001     |Own Brand               |
|Prod002     |Own Brand               |
|Prod003     |Own Brand               |
|Prod004     |Own Brand               |
|Prod005     |Own Brand               |

transactions:
|product_code|customer_code           |market_code|order_date|sales_qty|sales_amount|currency|
|------------|------------------------|-----------|----------|---------|------------|--------|
|Prod001     |Cus001                  |Mark001    |2017-10-10|100      |41241       |INR     |
|Prod001     |Cus002                  |Mark002    |2018-05-08|3        |-1          |INR     |
|Prod002     |Cus003                  |Mark003    |2018-04-06|1        |875         |INR     |
|Prod002     |Cus003                  |Mark003    |2018-04-11|1        |583         |INR     |
|Prod002     |Cus004                  |Mark003    |2018-06-18|6        |7176        |INR     |


I then looked for inconsistencies in the data by scanning visually scanning the data in the table viewer, and this is what I noticed. 

By filtering the sales_amount in the transactions table, I found several instances of sales_amount = 0 and -1. sales_amount here means the money collected from the sales made.
This table below shows the amount of times sales_amount where 0 and -1, and should be brought up to the sales manager as well as relevant parties as data is missing or incorrectly inputted. 

|number_of_sales|COUNT(sales_amount)     |
|---------------|------------------------|
|-1             |2                       |
|0              |1609                    |

<details>
  <summary> Using this SQL Query </summary>

```sql
SELECT '-1' AS number_of_sales, COUNT(sales_amount)
FROM sales.transactions
WHERE sales_amount = -1

UNION ALL

SELECT '0', COUNT(sales_amount)
FROM sales.transactions
WHERE sales_amount = 0;
```
  
</details>

On top of this, there was USD mixed in with the INR currencies. To fix this, I will multiply the USD data points by the exchange rate of 83. (As of 23/11/2023)

This table also filters out any sales_amount that is greater than 0, making sure we don't have data where there are no sales or where sales are a negative number. This will not affect the data negatively because we are filtering out potential days where there were no sales from the stores. 

We can assure this by checking whether there were any data where the sales quantity (sales_qty) is less than 1 with

```sql
SELECT * FROM sales. transactions
WHERE sales_qty > 1;
```
This would include 0 and anything below that.

This query returns no results confirming that there were no days where the stores reported 0 sales quantity. This means there is a conflict in the data; where there are sales_amounts of 0, there were still sales made, as shown in the sales_qty

There are also markets which are outside of the regions we want. We can confirm whether there are no sales which come from those regions using

```sql
SELECT * FROM sales.transactions
WHERE market_code = "Mark097" OR market_code = "Mark999";
```
This results in no values, ensuring that no new data has been added with this market_code. As such, we don't have to worry about it in terms of joining the data. 

To aggregate all the necessary datasets into one place, the data to join together will be:

**from date**
- date
- year
- month_name
- cy_date

**from transactions**
- product_code
- customer_code
- order_date
- sales_qty
- sales_amount 

**from products**
- product_type

**from customers**
- customer_type
- customer_name

**from markets**
- markets_code
- markets_name
- markets_zone

Joining these data sets allows us not to have redundant columns and all the datasets in one dataset. 

Using the following SQL CREATE TABLE query

```sql
CREATE TABLE sales.merged_cleanmerged_clean

SELECT
  date_info.date,
  date_info.year,
  date_info.month_name,
  date_info.cy_date,
  transactions.product_code,
  products.product_type,
  transactions.customer_code,
  customer.custmer_name AS customer_name, #Customer Name column incorrectly named
  customer.customer_type,
  transactions.order_date,
  markets.markets_code,
  markets.markets_name,
  markets.zone,
  transactions.sales_qty,

  CASE
    WHEN transactions.currency = "USD" 
    THEN sales_amount * 83                  #As of 23/11/2023
    ELSE sales_amount * 1
  END AS sales,

  CASE
    WHEN transactions.currency = "USD"
    THEN 'INR'
    ELSE 'INR'
  END AS currency

FROM sales.transactions AS transactions

LEFT JOIN
  sales.markets AS markets
  ON transactions.market_code = markets.markets_code

LEFT JOIN
  sales.date AS date_info
  ON date_info.date = transactions.order_date

LEFT JOIN
  sales.products AS products
  ON transactions.product_code = products.product_code

LEFT JOIN
  sales.customers AS customer
  ON transactions.customer_code = customer.customer_code

WHERE transactions.sales_amount > 0 
```
Using a LEFT JOIN here makes sense as it returns all records from the transactions dataset and the matching records from any of the joined datasets. 

To confirm the correct amount of data points using the COUNT(*) query.

```sql
SELECT 'merged_clean' AS dataset,
COUNT(*) AS data_points
FROM sales.merged_clean
UNION ALL
SELECT 'transations',
COUNT(*)
FROM sales.transactions
WHERE sales_amount > 0;
```
**Output**
|dataset|data_points|
|-------|-----------|
|merged_clean|148672     |
|transations|148672     |

To check for the markets correctly, not including the unwanted markets outside of India, we will use a SELECT DISTINCT query for the markets_name and code.

```sql
SELECT DISTINCT markets_name, markets_code 
FROM sales.merged_clean;
```

**Output**

|markets_code|markets_name            |zone   |
|------------|------------------------|-------|
|Mark001     |Chennai                 |South  |
|Mark002     |Mumbai                  |Central|
|Mark003     |Ahmedabad               |North  |
|Mark004     |Delhi NCR               |North  |
|Mark005     |Kanpur                  |North  |
|Mark006     |Bengaluru               |South  |
|Mark007     |Bhopal                  |Central|
|Mark008     |Lucknow                 |North  |
|Mark009     |Patna                   |North  |
|Mark010     |Kochi                   |South  |
|Mark011     |Nagpur                  |Central|
|Mark012     |Surat                   |North  |
|Mark013     |Bhopal                  |Central|
|Mark014     |Hyderabad               |South  |
|Mark015     |Bhubaneshwar            |South  |

Compared to the original markets dataset, we see that the data excludes the two markets outside India.

|markets_code|markets_name            |zone   |
|------------|------------------------|-------|
|Mark097     |New York                |       |
|Mark999     |Paris                   |       |

On top of that, we can confirm that all the data that was once USD is not INR by using the same SELECT DISTINCT query but with currency.

``` sql
SELECT DISTINCT currency
FROM sales.merged_clean
```
**Output**
|currency|
|--------|
|INR     |

As the CASE clause works, we can also assume the CASE clause for the currency exchange. However, to be sure, checking the transactions dataset WHERE the currency is USD gives 

|product_code|customer_code|market_code|order_date|sales_qty|sales_amount|currency|
|------------|-------------|-----------|----------|---------|------------|--------|
|Prod003     |Cus005       |Mark004    |2017-11-20|59       |500         |USD     |
|Prod003     |Cus005       |Mark004    |2017-11-22|36       |250         |USD     |

Then multiply the sales amount by 83

```sql
SELECT 
	customer_code,
	order_date,
  sales_qty,
  sales_amount * 83 AS sales_amount_INR
FROM sales.transactions
WHERE currency = "USD";
```

|customer_code|order_date|sales_qty|sales_amount_INR|
|-------------|----------|---------|----------------|
|Cus005       |2017-11-20|59       |41500           |
|Cus005       |2017-11-22|36       |20750           |

To check the same values in the cleaned data using

```sql
SELECT * FROM sales.merged_clean
WHERE customer_code = "Cus005" 
AND markets_code = "Mark004"
AND product_code = "Prod003"
AND order_date = "2017-11-20";
```
We will filter for the exact date, customer, product, and market codes.

**Output**

|date|year  |month_name|cy_date   |product_code|product_type|customer_code|customer_name |customer_type |order_date|markets_code|markets_name|zone |sales_qty|sales|currency|
|----|------|----------|----------|------------|------------|-------------|--------------|--------------|----------|------------|------------|-----|---------|-----|--------|
|2017-11-20|2017  |November  |2017-11-01|Prod003     |Own Brand   |Cus005       |Premium Stores|Brick & Mortar|2017-11-20|Mark004     |Delhi NCR   |North|59       |41500|INR     |

Then, we could do this again for the other data, but from this, we can assume that the other point has also worked.

With this, we now know the data set is correctly cleaned and has all the data in one dataset.

Now, we export the data set as a CSV. In an ideal world, I would make a new Tableau Flow and Tableau Desktop all connected via their cloud platform or a data warehouse. This way, any updates to this output file from SQL will automatically update the Tableau Flow table and output a dataset available to the Tableau Desktop server and can be set to run the updated dataset for the dashboard information. 

### Creating Dashboard

The dashboard should contain only the most pertinent data the Sales Manager requires. Not having been given any specifics, we will assume the following is needed:
- Sales Totals by Month filterable by year
- Sales and sales quantity comparisons between each year
- Tables that show the number of sales and sales quantities by month for each region
- The makeup of the sales by year
- A running sales comparison between all years

These should give the Sales Manager the exact numbers they would be looking for to decide which region to focus more on.

View the Tableau Dashboard in full [AtliQ Sales Dashboard](https://public.tableau.com/app/profile/kes.andrew.raath/viz/AtliQSalesDashboard_16988232682060/AtliQSalesDashboard)

![alt text](https://github.com/Kesraath/AtliQ_Sales_Dashboard/blob/AtliQProjectFiles/AtliQ%20Sales%20Dashboard.png?raw=true)

### Results
- Increase productivity
- Streamline sales manager's access to sales information
- 


