# Warehouse-Sales-Explorations

## Project Overview
This project aims to provide a comprehensive analysis of sales performance by integrating data from multiple tables, including customer information, product details, order history, customers’ review, products performance, and inventory information.  The analysis will identify key sales trends, top-performing products and regions, customer purchasing patterns, and progress against established goals. The insights derived from this project will enable data-driven decision-making to optimize sales strategies, improve resource allocation, and ultimately drive revenue growth.
	Below are the respective Tables:
(1)	Country Price Comparison Table
(2)	Laptop Customers Review Table
(3)	Phone Description Table
(4)	Shipment Table
(5)	Walk-In Sale Table

## Data Source
The primary dataset used for this analysis is in CSV format, containing detailed information abot each sales made, the mode and modes of pay, as well as that of the product made by the warehouse department of the company

## Tools and Steps
•	EXCEL; Used for:
1. Data loading and inspection
2. Data manipulations
3. Handling missing values,
4. Cleaning and
5. Formatting

•	SQL; Steps took involved:
1. Loading the refined datasets into the Database
2. Analysis

## Challenges
•	Challenge  1: Several improper casing of alphabets, a lot of misspellings, duplicates columns and rows, values errors, nulls and missing values were observed in the raw dataset.
      o	Solution: I use MS EXCEL to perform data manipulations, transformations, cleaning and formatting

•	Challenge 2: Another hurdle was loading the dataset to SQL server base, this was as a result of incorrect formatting of the CSV file to correspond with the SQL data type format
      o	Solution: This was also handled using Excel and SQL

## Exploratory Data Analysis
This involved exploring the dataset to answer key questions, as this will be grouped according to each table; such as
(A) 


## Data  Analysis
This include some interesting codes/features

##### Overview of the Total Sales in Shipment Table and Walikin Table
- Combination of shipment and walkinsales

  
```sql
select
	  w.TXN_ID,
	  w.Customer_Name,
    w.Amount,
    w.Order_Date,
    w.Quantity,
    w.City,
    w.Sub_Category
from walkinsales w

union

select 
	  s.TXN_ID,
    s.Customer_Name,
	  s.Price,
    s.Dispatch_Date,
    s.Quantity_Sold,
    s.Customer_Location,
    s.Product
from shipment s;
```


- Creating our temporary table

  ```sql
  with allsales AS (

select
	  w.TXN_ID,
	  w.Customer_Name,
    w.Amount,
    w.Order_Date,
    w.Quantity,
    w.City,
    w.Sub_Category
from walkinsales w

union

select 
	  s.TXN_ID,
    s.Customer_Name,
	  s.Price,
    s.Dispatch_Date,
    s.Quantity_Sold,
    s.Customer_Location,
    s.Product
from shipment s

)

```

- overview of allsales table

``` sql

select *
from allsales;

```


- Total Revenue

```sql

select 
	sum(Amount * Quantity) as Total_Revenue
from allsales;

```


#### The Shipment Table

- Shipment Table overview

  ```sql
  
select *
from shipment;

```sql

- overall sales performance
select
	sum(price * Quantity_Sold) as Total_Revenue,
    sum(Quantity_Sold) as Total_Sold_Goods,
    min(Inward_Date) as StartInwardDate,
    max(Inward_Date) as EndInwardDate,
    min(Dispatch_Date) as StartDispatchDate,
    max(Dispatch_Date) as EndDispatchDate
from shipment;

``

- Revenue by Product

``sql

select
	Product,
    count(Quantity_Sold) as Total_Qnty_Sold,
	sum(price * Quantity_Sold) as Total_Revenue
from shipment
group by Product;

```

- Revenue by Brand

```sql

select
	Brand,
    sum(Quantity_Sold) as Total_Sold_Goods,
	sum(price * Quantity_Sold) as Total_Revenue,
    avg(Price) as Average_Price_Phone
from shipment
group by Brand
order by Total_Revenue desc
limit 10;

```

- Customer Distribution by Region

```sql

select
	Region,
	sum(price * Quantity_Sold) as Total_Revenue,
    sum(Quantity_Sold) as Total_Sold_Goods
from shipment
group by Region
order by 2 desc;

```

- Track inventory turnover and sales velocity
     - Top brand with the best sales velocity

```sql

select
	Brand,
    avg(datediff(Dispatch_Date, Inward_Date)) as Sales_Velocity_Day
from shipment
group by Brand
order by Sales_Velocity_Day asc
limit 5;

```
    - Top brand with the worst sales velocity

```sql

select
	Brand,
    avg(datediff(Dispatch_Date, Inward_Date)) as Sales_Velocity_Day
from shipment
group by Brand
order by Sales_Velocity_Day desc
limit 5;

```

-  impact of product specifications on sales

    - Revenue by Core Specification

```sql

select
	Core_Specification,
    count(Quantity_Sold) as Total_Qnty_Sold,
	sum(price * Quantity_Sold) as Total_Revenue
from shipment
group by Core_Specification
order by Total_Revenue desc;

```

    - Revenue by Processor Specification

```sql

select
	Processor_Specification,
	sum(price * Quantity_Sold) as Total_Revenue
from shipment
group by Processor_Specification
order by Total_Revenue desc;

```


    - Revenue by RAM

```sql

select
	RAM,
	sum(price * Quantity_Sold) as Total_Revenue
from shipment
group by RAM
order by Total_Revenue desc;

```

     - Revenue by ROM

```sql

select
	ROM,
	sum(price * Quantity_Sold) as Total_Revenue
from shipment
group by ROM
order by Total_Revenue desc;

```

     - Revenue by SSD

```sql

select
	SSD,
	sum(price * Quantity_Sold) as Total_Revenue
from shipment
group by SSD
order by Total_Revenue desc;

```




#### The walkin Table

- walk-in sales

```sql

select *
from walkinsales;

```

- Overall sales performance, profit analysis and Sales Volume

```sql

select
	sum(Amount * Quantity) as Total_Sales,
	sum(Amount) as Total_Amount,
    sum(Profit) as Gross_Profit,
    sum(Quantity) as Total_Quantity
from walkinsales;

```

- Category by Performance

```sql

select
	Category,
    sum(Amount * Quantity) as Total_Sales,
    sum(Profit) as Total_Profit,
    sum(Quantity) as Total_Quantity
    
from walkinsales
group by Category;

```

 - Category by Performance
 
 ```sql

 select
	Sub_Category,
    sum(Amount * Quantity) as Total_Sales,
    sum(Profit) as Total_Profit,
    sum(Quantity) as Total_Quantity
    
from walkinsales
group by Sub_Category;

```


- Sales Trends Over Time with payment mode
- Yearly sales

```sql

with yearlysales as
(
select
		year(Order_Date) as Fiscal_Year,
        Payment_Mode,
		sum(Amount * Quantity) as Total_Sales
from walkinsales       
group by Payment_Mode, Fiscal_Year

)

select Fiscal_Year,
		Payment_Mode,
	    Total_Sales,
        lag(Total_Sales) over(partition by Payment_Mode order by Fiscal_Year) as PreviousMonthSales,
        Total_Sales - lag(Total_Sales) over(partition by Fiscal_Year order by Fiscal_Year) as SalesYOY_Difference,
        (
        (Total_Sales - lag(Total_Sales) over(partition by Payment_Mode order by Fiscal_Year)
        )/lag(Total_Sales) over(partition by Payment_Mode order by Fiscal_Year)) * 100 as Sales_Difference_Percent
from yearlysales;
-- order by Sales_Difference_Percent desc;

```

- Sales by City

```sql
select
		City,
		sum(Amount * Quantity) as Total_Sales
from walkinsales
group by City
order by Total_Sales desc
limit 5; 

```


- Sales by Customer

```sql

select
		Customer_Name,
		sum(Amount * Quantity) as Total_Sales
from walkinsales
group by Customer_Name
order by Total_Sales desc
limit 10; 

``

- Average Order Value by customer

```sql

select
		Customer_Name,
		sum(Amount)/sum(Quantity) as Total_Sales 
from walkinsales
group by Customer_Name
order by Total_Sales desc
limit 10;

```


#### -- Phones Description Table

- Overview of Phone description Table

  ```sql
  
select *
from phonesdescription;

```

- Number of phone brand, Number of phone and Grand total price

```sql

SELECT 
	count( distinct Brand_Name) as Num_Phone_brand,
    count(*) as Num_Phone,
    sum(price) as Grand_Total_Price
FROM phonesdescription;

```

- Brand Performance Analysis

```sql

select
	Brand_Name,
    count(OS) as Distribution_OS,
    Count(Brand_Name) as Num_Phones,
    avg(price) as Average_Price,
    avg(RAM) as Average_RAM,
    avg(Storage) as Average_Storage,
    SUM(CASE WHEN Has_Fast_Charging = 'Yes' THEN 1 ELSE 0 END) * 100 / COUNT(*) as "Fast_Charging_Adoption_Rate",
    SUM(CASE WHEN Has_Fingerprints = 'Yes' THEN 1 ELSE 0 END) * 100 / COUNT(*) as "Fingerprints_Adoption_Rate",
    SUM(CASE WHEN Has_NFC = 'Yes' THEN 1 ELSE 0 END) * 100 / COUNT(*) as "NFC_Adoption_Rate",
    SUM(CASE WHEN Has_5g = 'Yes' THEN 1 ELSE 0 END) * 100 / COUNT(*) as  "5g_Adoption_Rate"
	   
from phonesdescription
group by Brand_Name
order by Average_price desc;

```


- 5g adoption rate per brand

```sql

select
	Brand_Name,
    SUM(CASE WHEN Has_5g = 'Yes' THEN 1 ELSE 0 END) * 100 / COUNT(*) as  "5g_Adoption_Rate"
from phonesdescription
group by Brand_Name
order by 5g_Adoption_Rate desc;

```

- Fast charging adoption rate

```sql

select
	Brand_Name,
    SUM(CASE WHEN Has_Fast_Charging = 'Yes' THEN 1 ELSE 0 END) * 100 / COUNT(*) as "Fast_Charging_Adoption_Rate"
from phonesdescription
group by Brand_Name
order by Fast_Charging_Adoption_Rate desc;

```

- Operating System Market Share

```sql
  
select
	OS,
    avg(RAM) as AVG_RAM,
	count(OS) as Num_Phone    
from phonesdescription
Group by OS;

```

- Top Performing Configurations(based on RAM and Storage) within the best price

```sql
select
	Brand_Name,
    count(Phone_Name) as Num_Phones,
    avg(RAM) as Average_RAM,
    avg(Storage) as Average_Storage,
    avg(Price) as Average_Price
from phonesdescription
group by Brand_Name
order by 3,4 desc
limit 10;

```

- Camera Specification Analysis:

```sql

select
	Brand_Name,
    avg(Num_Rear_Cameras) as Average_Num_Back_Cam,
    avg(Num_Front_Cameras) as Average_Num_Front_Cam,
    avg(Primary_Rear_Camera) as Average_Back_Cam_Capacity,
    avg(Primary_Front_Camera) as Average_Front_Cam_Capacity
from phonesdescription
group by Brand_Name
order by 4,5 desc;

```

- Price vs display chararacteristic

```sql

select
	Display_Types,
    avg(price),
	avg(Display_Size_inch),
    avg(Refresh_Rate_hz)
from phonesdescription
group by Display_Types;

```

- Battery capacity vs price

```sql

select
	Avg(Price),
    Battery_Capacity
from phonesdescription
group by Battery_Capacity;

```

#### Laptops Customers review Table

- Laptop customers review overview

```sql

SELECT *
FROM laptopcustomersreview;

```

- Average Overall Rating: To get a general sense of customer satisfaction

```sql

SELECT
	AVG(Overall_Rating) As AVG_Overall_Rating
FROM laptopcustomersreview;

```


- Distinguished customer's satifaction

```sql

SELECT
	Title,
	COUNT(Overall_Rating) AS Num_OVerall_Rating,
    (COUNT(Overall_Rating)/24113)*100 AS Num_OVerall_Rating_Percentage
FROM laptopcustomersreview
GROUP BY Title
ORDER BY Num_Overall_Rating DESC
LIMIT 20;

```

- RATIO OF NUM OF REVIEW TO NUM OF RATING

```sql

SELECT
	Rating,
    (SUM(No_Reviews)/SUM(No_Ratings)) *100 As Comment_Ratio
FROM laptopcustomersreview
Group by Rating
order by Comment_Ratio desc;

```

- COUNTRY PRICE COMPARISON TABLE OVERVIEW

```sql

SELECT *
FROM countrypricecomparison;

```

- pricing trends across different country markets

```sql

SELECT
    AVG(launched_Price_Pakistan) AS avg_price_pakistan,
    AVG(launched_Price_India) AS avg_price_india,
    AVG(launched_Price_China) AS avg_price_china,
    AVG(launched_Price_USA) AS avg_price_usa,
    AVG(launched_Price_Dubai) AS avg_price_dubai
FROM countrypricecomparison;

```

- Price Variance By Company name, to understand consistency in phone pricing

```sql

SELECT Company_Name,
    STDDEV(launched_Price_Pakistan) AS STDDEV_price_Pakistan,
    STDDEV(launched_Price_India) AS STDDEV_price_India,
    STDDEV(launched_Price_China) AS STDDEV_price_China,
    STDDEV(launched_Price_USA) AS STDDEV_price_USA,
    STDDEV(launched_Price_Dubai) AS STDDEV_price_Dubai
FROM countrypricecomparison
GROUP BY Company_Name;

```

- Average price for each company name: To find the average price for each company

```sql

SELECT Company_Name,
	AVG(RAM) AS AVERAGE_RAM,
    AVG((Launched_Price_Pakistan + Launched_Price_India + Launched_Price_China + Launched_Price_USA + Launched_Price_Dubai)/5) AS AVERAGE_PRICE
FROM countrypricecomparison
GROUP BY Company_Name;

```

- Number of Models Launched per Year

  ```sql
  
SELECT 	Launched_Year,
		COUNT(DISTINCT Model_Name) AS Num_Model
        
FROM Countrypricecomparison
GROUP BY Launched_Year
ORDER BY Launched_Year desc;

```


- Most Frequent Brand by Launches:

```sql

SELECT
	Company_Name,
    COUNT(Model_Name) AS Num_Model
FROM countrypricecomparison
GROUP BY Company_Name
ORDER BY Num_Model DESC;

```

## Results/Findings
The analysis results are summarized as follows:
(A) The total Overall Sales which consist of two Tables, namely; Shipment and Walkin
1. The Total Overall sales from Shipment and Walkin, is 36,257,959,135 and 66,857,557 respectively. And both amount to 36,324,816,692. Also, The shipment and The Walkin contribute 99.8% and 0.2% respectively.

(B) Shipment
1. This sales span from 21/3/2023 to 20/3/2025 is associated with a sales of  36,257,959,135 which is 99.8 of total overall sales
2. More revenue were generated from laptops than mobile, of approximately 11 million difference
3. Western Region generate the highest revenue, although the differences in sales in other region is not as much, but sales can still pushed in all the regions
4. Apple and microsoft product has the best turn-over rate, while lenovo and HP seems to have the worst turn-over rate. Catalytic sales technical should be concentrated on lenovo and HP as they are popular products, so they should good in the market.
5. The Asian core with the IDE unknown core specification has the lion share of the market. so, more stocking the products should encouraged
6. The Snapdragon specification has the lion share of the market. so, more stocking the products should encouraged
7. The 64g RAM has the biggest sales. Also the 256gb ROM has the largest market share. Likewise is the NONE SSD among it pairs

(c) Walkin
1. The highest revenue was generated from office supplies, while highest  quantity generated is from furniture, in terms of Category
2. Markers generates the highest revenue as it perform best in the market among it pairs, while Binders generate least revenue
3. Revenue generated from debit cards as a means of payment appears to be highest with a total revenue of 14,957,183, as it seem to be the best method liked by most of our customers
4. Orlando appears to be the city with the highest sales of 	503,4310
5. The 10 ten customers in both queries Average Order Value and Sales by Customer respectively , should be provided with incentives as a way of encouraging the customers for more patronage and recommendation


(D) Country Price Comparison
1. Sony has the highest average price and RAM of  86114.55555556 and 8.6667 respectively
2. Pakistan has the highest average Price of 123289.4236 and USA has the lowest of 568.7347
3. Pakistan having the highest Price standard Deviation may lead to low sale in the nearest future, which may be as a result of inconsistent pricing system
4. More phone models were launched in 2024 which accounted for 286, models, than any other year. With just 2 models launched in 2014


   
