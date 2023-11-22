# AtliQ_Sales_Dashboard

### Project Overview

A sales manager needs help grasping the sales situation from his regional managers and receives nothing but convoluted spreadsheets. He wants one automated central location that allows him to understand the monthly sales data and trends for each region: North, Central, and South. The sales manager can then make data-driven decisions for improving the business in areas that aren’t doing as well using this tool. 

### Data Source

The data is from a "db_dump.sql" file from [codebasics](https://codebasics.io/resources/sales-insights-data-analysis-project)

It includes the sales information from a hypothetical Hardware that supplies hardware to many resellers in India. These stores are split into 3 regional hubs: North, Central and South. 

Although this is hypothetical data, the data will be assumed to be sales data provided directly by the company. As such, the reliability of such data is very high as it is a primary source of data from within the company.

The data consists of sales data from October 2017 to June 2020

### Tools

mySQL - Used to clean and prep the data
Tableau - Used for creating the visualisations

### Data Cleaning & Preparation

To resolve the sales managers' issue, creating a dashboard will be the most straightforward method for them to get the information they need and make data-driven decisions. 
A dashboard is preferable here as it needs to be live. It simplifies the spreadsheet data they received before into digestible visualisations and can be updated at the end of each month as requested by the sales manager.

After importing the data directly into MySQL with server data import and loading the .sql file in directly. We are given a schema with 5 tables inside,
- customers
- date
- markets
- products
- transactions

After receiving the data I looked for inconsistencies in the data. By filtering the sales numbers I found several instances of sales_amounts = 0 and -1. On top of this, there were USD mixed in with the INR currencies. To fix this, I created a new table with all the old other information in the transactions dataset while creating a filter to find if the currency is USD to multiply it by the exchange rate of 83 and if it isn’t, multiply by 1 to not affect the number.


### Choosing Relevant Data



### Creating Dashboard

View the Tableau Dashboard in full [AtliQ Sales Dashboard](https://public.tableau.com/app/profile/kes.andrew.raath/viz/AtliQSalesDashboard_16988232682060/AtliQSalesDashboard)



