# MYSQL-Module-End-Assignment-E-Commerce-Customer-Churn-Analysis
https://drive.google.com/drive/folders/1NY34vWvBtRXig3auKApI9tzBgyjSoKhm?usp=drive_link
Handling Missing Values and Outliers
-- Impute mean

-- WarehouseToHome
update customer_churn 
set WarehouseToHome = (select round(avg(WarehouseToHome)) from (select * from customer_churn) as t)
where WarehouseToHome is null;
select 
sum(case when WarehouseToHome is null then 1 else 0 end) as WarehouseToHome_nulls from customer_churn;

set sql_safe_updates=0;

select * from customer_churn;

-- HourSpendOnApp
update customer_churn
set HourSpendOnApp=  (select round(avg(HourSpendOnApp)) from (select * from customer_churn) as t)
where HourSpendOnApp is null;
select 
sum(case when HourSpendOnApp is null then 1 else 0 end) as HourSpendOnApp_nulls from customer_churn;

--  OrderAmountHikeFromlastYear
update customer_churn
set OrderAmountHikeFromlastYear= (select round(avg(OrderAmountHikeFromlastYear)) 
from (select * from customer_churn) as t)
where OrderAmountHikeFromlastYear is null;
select 
sum(case when OrderAmountHikeFromlastYear is null then 1 else 0 end) 
as OrderAmountHikeFromlastYear_nulls from customer_churn;

-- DaySinceLastOrder
update customer_churn
set DaySinceLastOrder= (select round(avg(DaySinceLastOrder)) 
from (select * from customer_churn) as t)
where DaySinceLastOrder is null;
select 
sum(case when DaySinceLastOrder is null then 1 else 0 end) 
as DaySinceLastOrder_nulls from customer_churn;

-- Impute mode
-- Tenure
update customer_churn
set Tenure = (
select Tenure from (
select Tenure from customer_churn 
group by Tenure 
order by count(*) desc limit 1) as t)
where Tenure is null;
select 
sum(case when Tenure is null then 1 else 0 end) 
as Tenure_nulls from customer_churn;

-- CouponUsed
update customer_churn
set CouponUsed = (
select CouponUsed from (
select CouponUsed from customer_churn 
group by CouponUsed 
order by count(*) desc limit 1) as t)
where CouponUsed is null;
select 
sum(case when CouponUsed is null then 1 else 0 end) 
as CouponUsed_nulls from customer_churn;

-- OrderCount
update customer_churn
set OrderCount = (
select OrderCount from (
select OrderCount from customer_churn 
group by OrderCount 
order by count(*) desc limit 1) as t)
where OrderCount is null;
select 
sum(case when OrderCount is null then 1 else 0 end) 
as OrderCount_nulls from customer_churn;

-- Handle outliers in the 'WarehouseToHome' column by deleting rows where the values are greater than 100 --

delete from customer_churn
where WarehouseToHome > 100;
set sql_safe_updates = 1;
select max(WarehouseToHome) as max_value from customer_churn;

-- Dealing with Inconsistencies
-- Replace PreferredLoginDevice and PreferredOrderCat to "Mobile Phone"
set SQL_SAFE_UPDATES = 0;
update customer_churn
set PreferredLoginDevice ='Mobile Phone'
where PreferredLoginDevice='Phone';
update customer_churn
set PreferedOrderCat= 'Mobile Phone'
where PreferedOrderCat= 'Mobile';

select distinct PreferredLoginDevice from customer_churn;

select distinct PreferedOrderCat from customer_churn;

-- Standardize PreferredPaymentMode
update customer_churn
set PreferredPaymentMode= 'Cash on Delivery'
where PreferredPaymentMode= 'COD';
update customer_churn
set PreferredPaymentMode= 'Credit Card'
where PreferredPaymentMode= 'CC';

select distinct PreferredPaymentMode from customer_churn;

-- Data Transformation
-- Column Renaming
alter table customer_churn
Rename column PreferedOrderCat to PreferredOrderCat;
alter table customer_churn
Rename column HourSpendOnApp to HoursSpentOnApp;
select * from customer_churn;

-- Creating New Columns
-- Create a new column named ‘ComplaintReceived’ with values "Yes" if the corresponding value in the ‘Complain’ is 1, and "No" otherwise
alter table customer_churn
add column ComplaintReceived varchar(10);
update customer_churn
set ComplaintReceived= case when complain=1 then 'Yes' else 'No' end;

-- Create a new column named 'ChurnStatus'. Set its value to “Churned” if the corresponding value in the 'Churn' column is 1, else assign “Active”
alter table customer_churn
add column ChurnStatus varchar(10);
update customer_churn
set ChurnStatus= case when churn=1 then 'Churned' else 'Active' end;

select Complain, ComplaintReceived, Churn, ChurnStatus 
from customer_churn;

-- Column Dropping
alter table customer_churn 
drop column Churn, 
drop column Complain;

-- Data Exploration and Analysis
-- Retrieve the count of churned and active customers from the dataset

select ChurnStatus, count(*) as Customer_Count
from customer_churn
group by ChurnStatus;

-- Display the average tenure and total cashback amount of customers who churned
select avg(Tenure) as Avg_Tenure_Churned,
sum(CashbackAmount) as Total_Cashback_Churned
from customer_churn
where ChurnStatus = 'Churned';

-- Determine the percentage of churned customers who complained

select (count(case when ComplaintReceived = 'Yes' then 1 end) * 100.0 / count(*)) 
as Percentage_Complained_And_Churned
from customer_churn 
where ChurnStatus = 'Churned';

-- Identify the city tier with the highest number of churned customers whose preferred order category is Laptop & Accessory

select CityTier, count(*) as Churned_Count
from customer_churn
where ChurnStatus = 'Churned'
and PreferredOrderCat = 'Laptop & Accessory'
group by CityTier
order by Churned_Count desc
limit 1;

-- Identify the most preferred payment mode among active customers

select PreferredPaymentMode, count(*) as Active_Customers
from customer_churn
where ChurnStatus = 'Active'
group by PreferredPaymentMode
order by Active_Customers desc
limit 1;

-- Calculate the total order amount hike from last year for customers who are single and prefer mobile phones for ordering

select sum(OrderAmountHikeFromlastYear) as Total_Hike
from customer_churn
where MaritalStatus ='Single'
and PreferredOrderCat = 'Mobile Phone';

-- Find the average number of devices registered among customers who used UPI as their preferred payment mode

select avg(NumberOfDeviceRegistered) as Avg_UPI_Customers
from customer_churn
where PreferredPaymentMode = 'UPI';

-- Determine the city tier with the highest number of customers

select CityTier, count(*) as Total_Customers
from customer_churn
group by CityTier
order by Total_Customers desc
limit 1;

-- Identify the gender that utilized the highest number of coupons

select Gender, sum(CouponUsed) as Total_Coupons
from customer_churn
group by Gender
Order by Total_Coupons desc
limit 1; 

-- List the number of customers and the maximum hours spent on the app in each preferred order category

select PreferredOrderCat, count(*) as Num_Customers, max(HoursSpentOnApp) as Max_HrsSpent
from customer_churn
group by PreferredOrderCat;

-- Calculate the total order count for customers who prefer using credit cards and have the maximum satisfaction score

select sum(OrderCount) as Total_OrderCount
from customer_churn
where PreferredPaymentMode = 'Credit Card'
and SatisfactionScore = (select max(SatisfactionScore) from customer_churn);

-- What is the average satisfaction score of customers who have complained?

select avg(SatisfactionScore) as Avg_satisfaction_Complained
from customer_churn
where ComplaintReceived = 'Yes';

-- List the preferred order category among customers who used more than 5 coupons

select PreferredOrderCat, count(*) as Morethan5_CouponUsed
from customer_churn
where CouponUsed >5
group by PreferredOrderCat
order by Morethan5_CouponUsed desc;

-- List the top 3 preferred order categories with the highest average cashback amount

select PreferredOrderCat, avg(CashbackAmount) as Highest_avgCashback
from customer_churn
group by PreferredOrderCat
order by Highest_avgCashback desc
limit 3;

-- Find the preferred payment modes of customers whose average tenure is 10 months and have placed more than 500 orders

select PreferredPaymentMode, avg(Tenure) as Avg_Tenure
from customer_churn
where OrderCount >500
group by PreferredPaymentMode
having avg(Tenure)= 10;

-- Categorize customers based on their distance from the warehouse to home such as 'Very Close Distance' for distances <=5km, 'Close Distance' for <=10km, 'Moderate Distance' for <=15km, and 'Far Distance' for >15km. Then, display the churn status breakdown for each distance category

select 
case when WarehouseToHome <=5 then 'Very Close Distance'
	when WarehouseToHome <=10 then 'Close Distance'
    when WarehouseToHome <=15 then 'Moderate Distance'
    else 'Far Distance'
    end as Distance_Category,
    ChurnStatus, count(*) as Customer_Count
    from customer_churn
    group by Distance_Category, ChurnStatus
    order by Distance_Category, ChurnStatus;
    
    -- List the customer’s order details who are married, live in City Tier-1, and their order counts are more than the average number of orders placed by all customers
    
select * from customer_churn
where MaritalStatus = 'Married'
and CityTier = 1
and OrderCount > (select avg(OrderCount) from customer_churn);

-- a) Create a ‘customer_returns’ table in the ‘ecomm’ database and insert the following data

Create table customer_returns (
  ReturnID int primary key,
  CustomerID int,
  ReturnDate date,
  RefundAmount int
  );
  select * from customer_returns;
  
insert into customer_returns (ReturnID, CustomerID, ReturnDate, RefundAmount) values
(1001, 50022, '2023-01-01', 2130),
(1002, 50316, '2023-01-23', 2000),
(1003, 51099, '2023-02-14', 2290),
(1004, 52321, '2023-03-08', 2510),
(1005, 52928, '2023-03-20', 3000),
(1006, 53749, '2023-04-17', 1740),
(1007, 54206, '2023-04-21', 3250),
(1008, 54838, '2023-04-30', 1990);

-- Display the return details along with the customer details of those who have churned and have made complaints

select c.*, r.ReturnID, r.ReturnDate, r.RefundAmount
from customer_churn c
join customer_returns r on c.CustomerID = r.CustomerID
where c.ChurnStatus = 'Churned' 
and c.ComplaintReceived = 'Yes';
