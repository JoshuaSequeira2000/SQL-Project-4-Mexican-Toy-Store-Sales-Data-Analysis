# SQL Project 4 - Exploratory Data Analysis - Mexican Toy Store Sales Analysis

## Complete code attached - Mexico Toy Store Sales Analysis.sql

## Building the Database.
```
-- Creating Mexico Toy Store Sales Database.
create database MexicoToyStoreSalesDatabase
GO

-- Selecting the Database.
use MexicoToyStoreSalesDatabase
GO

-- Creating tables and inserting data.
-- Creating table Stores.
create table Stores(
	Store_ID tinyint not null constraint PK_Stores_Store_ID primary key,
	Store_Name varchar(50) not null,
	Store_City varchar(25) not null,
	Store_Location varchar(12) not null constraint Check_Stores_Store_Location 
	check(Store_Location in ('Airport', 'Commercial', 'Residential', 'Downtown')),
	Store_Open_Date date not null,
	UniqueID uniqueidentifier constraint Default_Stores_UniqueID default NEWSEQUENTIALID(),
	Updated_Timestamp datetimeoffset constraint Default_Stores_Updated_Timestamp default sysdatetimeoffset()
)
GO

-- Inserting data into Stores. 1 row with insert command, remaining with GUI.
insert into Stores([Store_ID], [Store_Name], [Store_City], [Store_Location], [Store_Open_Date])
values(1, 'Maven Toys Guadalajara 1', 'Guadalajara', 'Residential', '1992-09-18')
GO

-- Creating table Inventory.
create table Inventory(
	Store_ID tinyint not null constraint FK_Inventory_Store_ID foreign key references [dbo].[Stores]([Store_ID]),
	Product_ID smallint not null constraint FK_Inventory_Product_ID foreign key references [dbo].[Products]([Product_ID]),
	Stock_On_Hand smallint null constraint Default_Inventory_Stock_On_Hand default 0,
	UniqueID uniqueidentifier constraint Default_inventory_UniqueID default NEWSEQUENTIALID(),
	Updated_Timestamp datetimeoffset constraint Default_Inventory_Updated_Timestamp default sysdatetimeoffset()
)
GO

-- Inserting data into Inventory. 1 row with insert command, remaining with GUI.
insert into Inventory([Store_ID], [Product_ID], [Stock_On_Hand])
values(1, 1, 27)
GO

-- Creating table Products.
create table Products(
	Product_ID smallint not null constraint PK_Products_Product_ID primary key,
	Product_Name varchar(100) not null,
	Product_Category varchar(50) not null,
	Product_Cost decimal(5,2) not null,
	Product_Price decimal(5,2) not null,
	UniqueID uniqueidentifier constraint Default_Products_UniqueID default NEWSEQUENTIALID(),
	Updated_Timestamp datetimeoffset constraint Default_Products_Updated_Timestamp default sysdatetimeoffset()
)
GO

-- Inserting data into Products. 1 row with insert command, remaining with GUI.
insert into Products([Product_ID], [Product_Name], [Product_Category], [Product_Cost], [Product_Price])
values(1, 'Action Figure', 'Toys', 9.99, 15.99)
GO

-- Creating Sales table by importing flat file (CSV File - over 800,000 rows)
alter table Sales
add constraint PK_Sales_Sales_ID primary key (Sale_ID)
GO
```

## Data Insights Using SQL.

### 1) Number of stores currently open.
```
select COUNT(distinct Store_ID) as Number_Of_Stores from Stores
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/2af1d49a-ca2e-4d39-838a-e38c2d9c1058)

### 2) Number of stores in each city.
```
select Store_City, COUNT(Store_City) as Number_Of_Stores from Stores
group by Store_City
order by Store_City
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/81713ac6-cb88-4cf4-975d-c62ea4b37fec)

### 3) Number of stores based on Store_Location.
```
select Store_Location, COUNT(Store_Location) as Number_Of_Stores from Stores
group by Store_Location
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/9a894c0d-b924-4420-a5fc-31d7cdcffeee)

### 4) Number of stores opened each year.
```
with Store_Open_Year as
(
	select Store_ID, Store_Open_Date 
	from Stores
)
select LEFT(Store_Open_Date, 4) as Store_Open_Year, COUNT(Store_ID) as Number_Of_Stores 
from Store_Open_Year
group by LEFT(Store_Open_Date, 4)
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/ad053c17-1603-494f-80ea-577bd77702bd)

### 5) Number of products.
```
select COUNT(Product_ID) as Number_Of_Products from Products
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/19f2faf9-eab5-491e-8a18-8996e9799ebe)

### 6) Number of products per category.
```
select Product_Category, COUNT(Product_ID) as Number_Of_Products 
from Products
group by Product_Category
order by Product_Category
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/1dc65297-2596-4f9c-902d-d5dec6f7ee6f)

### 7) Avg cost for a product in each category.
```
select Product_Category, format(AVG(Product_Cost), 'C', 'es-MX') as Avg_Product_Cost from Products
group by Product_Category
order by Product_Category
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/ed33768d-a117-46da-a1bc-7e46c21a40c5)

### 8) Most expensive and least expensive product under each category.
```
with Most_Expensive_Product as 
(
select Product_Category, Product_Name, Product_Price, 
FIRST_VALUE(Product_Name) over(partition by Product_Category order by Product_Price desc) as Most_Expensive_Product,
DENSE_RANK() over(partition by Product_Category order by Product_Price Desc) as DenseRank
from Products 
),
Least_Expensive_Product as
(
select Product_Category, Product_Name, Product_Price,
LAST_VALUE(Product_Name) over(partition by Product_Category order by Product_Price desc 
							  rows between unbounded preceding and unbounded following) as Least_Expensive_Product,
DENSE_RANK() over(partition by Product_Category order by Product_Price) as DenseRank
from Products
)
select m.Product_Category, m.Product_Name as Most_Expensive_Product, m.Product_Price as Most_Expensive_Product_Price , 	
                           l.Product_Name as Least_Expensive_Product, l.Product_Price as Least_Expensive_Product_Price
from Most_Expensive_Product m
inner join Least_Expensive_Product l on m.Product_Category = l.Product_Category 
where m.DenseRank = 1 and l.DenseRank = 1
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/e3f2a58a-be73-4aca-8278-fbfe5e75d5ba)

### 9) Total products sold to date.
```
select format(SUM(Units), '#,###,###') as Total_Products_Sold 
from Sales
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/128ab630-e1a0-4d26-b21b-9790bf400223)

### 10) Total sales to date.
```
select format(SUM(Units), '#,###,###') as Total_Products_Sold 
from Sales
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/19cdf762-6198-4c34-a912-db1b8251db25)

### 11) Total profit to date.
```
with Total_Sales as
(
	select ROW_NUMBER() over(order by (select null)) as ID_Sales, -- Row number used to assign ID 1. This will be used to join with below table for subtraction purpose.
	SUM(p.Product_Price * s.Units) as Total_Sales from Sales s
	inner join Products p on s.Product_ID = p.Product_ID
),
Total_Cost as
(
	select ROW_NUMBER() over(order by (select null)) as ID_Cost,
	SUM(p.Product_Cost * s.Units) as Total_Cost from Sales s
	inner join Products p on s.Product_ID = p.Product_ID
)
select format((ts.Total_Sales - tc.Total_Cost), 'C', 'es-MX') as Total_Profit
from Total_Cost tc
inner join Total_Sales ts on tc.ID_Cost = ts.ID_Sales
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/198ced19-a5fa-4a19-88c7-4fb388357550)

### 12) Total number of products sold based on Product_ID.
```
with Total_Products_Sold as
(
select p.Product_Category, p.Product_ID, p.Product_Name, (COUNT(s.Product_ID) * s.Units) as Number_Of_Products_Sold
from Products p
inner join sales s on p.Product_ID = s.Product_ID
group by p.Product_Category, p.Product_ID, p.Product_Name, s.Units
)
select Product_Category, Product_ID, Product_Name, format(SUM(Number_Of_Products_Sold), '#,###,###') as Total_Products_Sold
from Total_Products_Sold
group by  Product_Category, Product_ID, Product_Name
order by Product_Category, SUM(Number_Of_Products_Sold) desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/f82a53cd-4d1f-44c2-8888-c73c0bb50dd6)

### 13) Total sales revenue from each product. 
```
with total_revenue as
(
select p.Product_Category, p.Product_ID, p.Product_Name, p.Product_Price, s.Units,
p.Product_Price * s.Units as Total
from Products p
inner join Sales s on p.Product_ID = s.Product_ID
)
select Product_Category, Product_ID, Product_Name, 
format(SUM(Product_Price * Units), 'C', 'es-MX') as Total_Revenue
from total_revenue
group by Product_Category, Product_ID, Product_Name
order by Product_Category, SUM(Product_Price * Units) desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/e064cd9b-4a53-4b8c-b56e-8634e5224e33)

### 14) Total profit from each product.
```
with total_Sales_Per_Product as
(
	select p.Product_Category, p.Product_ID, p.Product_Name,
	sum(p.Product_Price * s.Units) as Total
	from Products p
	inner join Sales s on p.Product_ID = s.Product_ID
	group by p.Product_Category, p.Product_ID, p.Product_Name
),
	total_Cost_Per_Product as
(
	select p.Product_Category, p.Product_ID, p.Product_Name,
	sum(p.Product_Cost * s.Units) as Total
	from products p
	inner join Sales s on p.Product_ID = s.Product_ID
	group by p.Product_Category, p.Product_ID, p.Product_Name
)
select s.Product_Category, s.Product_ID, s.Product_Name, format((s.Total - c.Total), 'C', 'es-MX') as Profit
from total_Sales_Per_Product s
inner join total_Cost_Per_Product c on s.Product_ID = c.Product_ID
group by s.Product_Category, s.Product_ID, s.Product_Name, s.Total, c.Total
order by s.Product_Category, s.Product_ID
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/0f1fadb2-e670-4eee-971a-b912400974a8)

### 15) Total sales from each store.
```
select st.Store_ID, st.Store_Name, st.Store_Location, st.Store_City, format(SUM(p.Product_Price * s.Units), 'C', 'es-MX') as Total_Sales
from Stores st
inner join Sales s on st.Store_ID = s.Store_ID
inner join Products p on s.Product_ID = p.Product_ID
group by st.Store_ID, st.Store_Name, st.Store_Location, st.Store_City
order by st.Store_ID, st.Store_Name, st.Store_Location, st.Store_City
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/19e072e4-5f38-4acc-af09-b103fab3edfe)
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/82ef0f26-e775-445b-ba70-76ff0b7fd24b)

### 16) Total sales in each city and each store location type.
#### 16a) City
```
select st.Store_City, format(SUM(p.Product_Price * s.Units), 'C', 'es-MX') as Total_Sales
from Stores st
inner join Sales s on st.Store_ID = s.Store_ID
inner join Products p on s.Product_ID = p.Product_ID
group by st.Store_City
order by SUM(p.Product_Price * s.Units) desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/1e651254-c6e8-4851-8ae2-266b13a4f3a9)

#### 16b) Store location
```
select st.Store_Location, format(SUM(p.Product_Price * s.Units), 'C', 'es-MX') as Total_Sales
from Stores st
inner join Sales s on st.Store_ID = s.Store_ID
inner join Products p on s.Product_ID = p.Product_ID
group by st.Store_Location
order by SUM(p.Product_Price * s.Units) desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/fd52c67e-e407-4d56-9139-4c6e9cafe05b)

### 17) Total profit generated from each store, each city, and each store location.
#### 17a) Store
```
with Sales_From_Each_Store as
(
	select st.Store_ID, st.Store_Name, st.Store_Location, st.Store_City, SUM(p.Product_Price * s.Units) as Total_Sales
	from Stores st
	inner join Sales s on st.Store_ID = s.Store_ID
	inner join Products p on s.Product_ID = p.Product_ID
	group by st.Store_ID, st.Store_Name, st.Store_Location, st.Store_City
),
	 Cost_From_Each_Store as
(
	select st.Store_ID, st.Store_Name, st.Store_Location, st.Store_City, SUM(p.Product_Cost * s.Units) as Total_Sales
	from Stores st
	inner join Sales s on st.Store_ID = s.Store_ID
	inner join Products p on s.Product_ID = p.Product_ID
	group by st.Store_ID, st.Store_Name, st.Store_Location, st.Store_City
)
select s.Store_ID, s.Store_Name, s.Store_Location, s.Store_City, 
	   format((s.Total_Sales - c.Total_Sales), 'C', 'es-MX') as Profit
from Sales_From_Each_Store s
inner join Cost_From_Each_Store c on s.Store_ID = c.Store_ID
group by s.Store_ID, s.Store_Name, s.Store_Location, s.Store_City, s.Total_Sales, c.Total_Sales
order by s.Total_Sales - c.Total_Sales desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/759172a8-ec8b-411f-a79f-35e6aa4809a0)
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/25871453-9eb4-4eee-bee9-8a66c673f777)

#### 17b) City
```
with Sales_From_Each_City as
(
	select st.Store_City, SUM(p.Product_Price * s.Units) as Total_Sales
	from Stores st
	inner join Sales s on st.Store_ID = s.Store_ID
	inner join Products p on s.Product_ID = p.Product_ID
	group by st.Store_City
),
	 Cost_From_Each_City as
(
	select st.Store_City, SUM(p.Product_Cost * s.Units) as Total_Sales
	from Stores st
	inner join Sales s on st.Store_ID = s.Store_ID
	inner join Products p on s.Product_ID = p.Product_ID
	group by st.Store_City
)
select s.Store_City, 
	   format((s.Total_Sales - c.Total_Sales), 'C', 'es-MX') as Profit
from Sales_From_Each_City s
inner join Cost_From_Each_City c on s.Store_City = c.Store_City
group by s.Store_City, s.Total_Sales, c.Total_Sales
order by s.Total_Sales - c.Total_Sales desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/bdd5d584-59bb-4c7a-b998-2835e3cfc4f3)

#### 17c) Store location
```
with Sales_From_Each_Store_Location as
(
	select st.Store_Location, SUM(p.Product_Price * s.Units) as Total_Sales
	from Stores st
	inner join Sales s on st.Store_ID = s.Store_ID
	inner join Products p on s.Product_ID = p.Product_ID
	group by st.Store_Location
),
	 Cost_From_Each_Store_Location as
(
	select st.Store_Location, SUM(p.Product_Cost * s.Units) as Total_Sales
	from Stores st
	inner join Sales s on st.Store_ID = s.Store_ID
	inner join Products p on s.Product_ID = p.Product_ID
	group by st.Store_Location
)
select s.Store_Location, 
	   format((s.Total_Sales - c.Total_Sales), 'C', 'es-MX') as Profit
from Sales_From_Each_Store_Location s
inner join Cost_From_Each_Store_Location c on s.Store_Location = c.Store_Location
group by s.Store_Location, s.Total_Sales, c.Total_Sales
order by s.Total_Sales - c.Total_Sales desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/b456b5fe-1c63-4fa8-a090-db182e0dbc61)

### 18) Check whether each store, city, and store location sales have increased in 2018.
#### 18a) Store
```
with Store_Sales_Profits_Increase_Decrease as
(
	select st.Store_ID, st.Store_Name, YEAR(s.Sales_Date) as Sales_Year, 
	SUM(p.Product_Price * s.Units) as Sales,
	Lag(SUM(p.Product_Price * s.Units)) over(partition by st.Store_ID order by YEAR(s.Sales_Date)) as Previous_Year_Sales,
	SUM(p.Product_Price * s.Units) - SUM(p.Product_Cost * s.Units) as Profits,
	Lag(SUM(p.Product_Price * s.Units) - SUM(p.Product_Cost * s.Units)) 
	over(Partition by st.Store_ID order by YEAR(s.Sales_Date)) as Previous_Year_Profits

	from Stores st
	inner join Sales s on st.Store_ID = s.Store_ID
	inner join Products p on s.Product_ID = p.Product_ID
	group by st.Store_ID, st.Store_Name, YEAR(s.Sales_Date)
)
select Store_ID, Store_Name, Sales_Year, Sales, Previous_Year_Sales,
	   case when Previous_Year_Sales is null then null
			when Previous_Year_Sales > Sales then 'Lower Sales Than Previous Year'
			when Previous_Year_Sales = Sales then 'Sales Equal To Previous Year'
			when Previous_Year_Sales < Sales then 'Higher Sales Than Previous Year'
			end as Result_Sales,

		Profits, Previous_Year_Profits,
		case when Previous_Year_Profits is null then null
			 when Previous_Year_Profits > Profits then 'Lower Profits Than Previous Year'
			 when Previous_Year_Profits = Profits then 'Profits Equal To Previous Year'
			 when Previous_Year_Profits < Profits then 'Higher Profits Than Previous Year'
			 end as Result_Profits
from Store_Sales_Profits_Increase_Decrease
order by Store_ID, Store_Name, Sales_Year
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/2e20bb1c-bff5-42bf-a511-502df26314e4)
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/9955498f-d30d-45e1-82c2-904f9ba7babb)

#### 18b) City
```
with City_Sales_Profits_Increase_Decrease as
(
	select st.Store_City, YEAR(s.Sales_Date) as Sales_Year, 
	SUM(p.Product_Price * s.Units) as Sales,
	Lag(SUM(p.Product_Price * s.Units)) over(partition by st.Store_City order by YEAR(s.Sales_Date)) as Previous_Year_Sales,
	SUM(p.Product_Price * s.Units) - SUM(p.Product_Cost * s.Units) as Profits,
	Lag(SUM(p.Product_Price * s.Units) - SUM(p.Product_Cost * s.Units)) 
	over(Partition by st.Store_City order by YEAR(s.Sales_Date)) as Previous_Year_Profits

	from Stores st
	inner join Sales s on st.Store_ID = s.Store_ID
	inner join Products p on s.Product_ID = p.Product_ID
	group by st.Store_City, YEAR(s.Sales_Date)
)
select Store_City, Sales_Year, Sales, Previous_Year_Sales,
	   case when Previous_Year_Sales is null then null
			when Previous_Year_Sales > Sales then 'Lower Sales Than Previous Year'
			when Previous_Year_Sales = Sales then 'Sales Equal To Previous Year'
			when Previous_Year_Sales < Sales then 'Higher Sales Than Previous Year'
			end as Result_Sales,

		Profits, Previous_Year_Profits,
		case when Previous_Year_Profits is null then null
			 when Previous_Year_Profits > Profits then 'Lower Profits Than Previous Year'
			 when Previous_Year_Profits = Profits then 'Profits Equal To Previous Year'
			 when Previous_Year_Profits < Profits then 'Higher Profits Than Previous Year'
			 end as Result_Profits
from City_Sales_Profits_Increase_Decrease
order by Store_City, Sales_Year
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/bce9ab23-4d52-4e12-aed4-02ef27653f36)
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/27f8ceb5-f0df-4bf6-92b1-1a9d9bc3cc7b)

#### 18c) Store location
```
with Store_Location_Sales_Profits_Increase_Decrease as
(
	select st.Store_Location, YEAR(s.Sales_Date) as Sales_Year, 
	SUM(p.Product_Price * s.Units) as Sales,
	Lag(SUM(p.Product_Price * s.Units)) over(partition by st.Store_Location order by YEAR(s.Sales_Date)) as Previous_Year_Sales,
	SUM(p.Product_Price * s.Units) - SUM(p.Product_Cost * s.Units) as Profits,
	Lag(SUM(p.Product_Price * s.Units) - SUM(p.Product_Cost * s.Units)) 
	over(Partition by st.Store_Location order by YEAR(s.Sales_Date)) as Previous_Year_Profits

	from Stores st
	inner join Sales s on st.Store_ID = s.Store_ID
	inner join Products p on s.Product_ID = p.Product_ID
	group by st.Store_Location, YEAR(s.Sales_Date)
)
select Store_Location, Sales_Year, Sales, Previous_Year_Sales,
	   case when Previous_Year_Sales is null then null
			when Previous_Year_Sales > Sales then 'Lower Sales Than Previous Year'
			when Previous_Year_Sales = Sales then 'Sales Equal To Previous Year'
			when Previous_Year_Sales < Sales then 'Higher Sales Than Previous Year'
			end as Result_Sales,

		Profits, Previous_Year_Profits,
		case when Previous_Year_Profits is null then null
			 when Previous_Year_Profits > Profits then 'Lower Profits Than Previous Year'
			 when Previous_Year_Profits = Profits then 'Profits Equal To Previous Year'
			 when Previous_Year_Profits < Profits then 'Higher Profits Than Previous Year'
			 end as Result_Profits
from Store_Location_Sales_Profits_Increase_Decrease
order by Store_Location, Sales_Year
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/2dffd148-453f-4b8d-a78d-b8cdce89a87d)

### 19) Pivot table with total sales each month from 2017 and 2018.
```
with Total_Sales_Per_Month_Pivot as
(
	select YEAR(s.Sales_Date) as SalesYear, DATENAME(MONTH, s.Sales_Date) as SalesMonth, 
	p.Product_Price * s.Units as Total_Sales
	from Sales s
	inner join Products p on s.Product_ID = p.Product_ID
)
select * from Total_Sales_Per_Month_Pivot
pivot(SUM(Total_Sales) for SalesMonth in ([January], [February], [March], [April],
[May], [June], [July], [August], [September], [October], [November], [December])) as pvt
order by SalesYear
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/476a5104-79b2-49cd-a6a6-b3f5310b7c40)

### 20) Pivot table with total profit each month from 2017 to 2018.
```
with Total_Profit_Each_month as
(
	select YEAR(s.Sales_Date) as ProfitYear, DATENAME(MONTH, s.Sales_Date) as Profit_Month,
	(p.Product_Price * s.Units) - (p.Product_Cost * s.Units) as Total_Profit
	from Sales s
	inner join Products p on s.Product_ID = p.Product_ID
)
select * from Total_Profit_Each_month
pivot(sum(Total_Profit) for Profit_Month in ([January], [February], [March], [April],
[May], [June], [July], [August], [September], [October], [November], [December])) as pvt
order by ProfitYear
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project4-Exploratory-Data-Analysis/assets/92262753/6500bb66-7433-4d9d-8ddd-8418d49aff1f)
