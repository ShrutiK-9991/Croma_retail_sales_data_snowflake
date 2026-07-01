# Croma India - Retail Data Analysis (Snowflake SQL)
A comprehensive SQL-based analysis of Croma India's retail operations using Snowflake. The project covers sales performance, customer behaviour, product trends, inventory health, and employee analytics across 6 staging tables.

# Objective
This project aims to analyze Croma India's retail operations using SQL on Snowflake to derive actionable business insights across key domains:
Sales Performance — Track revenue trends, identify peak sales days, and compare weekend vs. weekday
Customer Analytics — Understand repeat purchase behaviour, loyalty tier effectiveness, and city-wise spending patterns
Product Intelligence — Identify best-selling products, revenue by category, and dead catalog items that have never been sold
Store Operations — Evaluate store-level revenue, and performance by state and city
Inventory Health — Out of stock products

# Schema

# 1. CUSTOMER TABLE
```sql
create table customer(
    customer_id	     int,
    first_name	     varchar(25),
    last_name	     varchar(30),
    email	         varchar(50),
    phone	         varchar(25),
    birth_date	     date,
    gender	         varchar(15),
    address	         varchar(50),
    city	         varchar(30),
    state	         varchar(30),
    postal_code	     int,
    country	         varchar(15),
    signup_date	     date,
    loyalty_tier     varchar(25)
);
```

# 2. EMPLOYEE TABLE
```sql
create table employee(
    employee_id	        int,
    first_name	        varchar(25),
    last_name	        varchar(25),
    email	            varchar(50),
    phone	            varchar(25),
    hire_date	        date,
    termination_date	date,
    store_id	        int,
    job_title           varchar(30)
);
```

# 3. INVENTORY TABLE
```sql
create table inventory(
    inventory_id	    int,
    snapshot_date	    date,
    store_id	        int,
    product_id	        int,
    on_hand_qty	        int,
    on_order_qty	    int,
    reorder_point	    int,
    unit_cost           float
);
```

# 4. PRODUCT TABLE
```sql
create table product(
    product_id	         int,
    product_sku	         varchar(30),
    product_name	     varchar(35),
    brand	             varchar(30),
    category	         varchar(30),
    subcategory	         varchar(30),
    color	             varchar(30),
    size	             varchar(20),
    unit_price	         float,
    start_date	         date,
    end_date	         date,
    is_active            boolean
);
```

# 5. SALES TABLE
```sql
create table sales(
    sales_id	        int,
    sales_date	        date,
    sales_time	        time,
    store_id	        int,
    product_id	        int,
    customer_id	        int,
    employee_id	        int,
    quantity	        int,
    unit_price	        float,
    gross_amount	    float,
    discount_amount	    float,
    net_amount	        float,
    payment_type	    varchar(25),
    transaction_id      varchar(35)
);
```

# 6. STORE TABLE
```sql
create table store(
    store_id	        int,
    store_name	        varchar(75),
    store_number	    int,
    format	            varchar(25),
    address	            varchar(50),
    city	            varchar(30),
    state	            varchar(35),
    postal_code	        int,
    country	            varchar(25),
    region	            varchar(25),
    open_date	        date,
    close_date	        date,
    sq_ft               int
);
```

# Analysis Performed 

### 1.  What is the total revenue?
```sql
select sum(gross_amount) as "Total_Revenue", count(*) as "Total_Transactions"
from sales;
```

### 2.  Montly revenue trend
```sql
select sum(gross_amount) as "Total_Revenue", extract(month from sales_date) as "Month" 
from sales
group by 2
order by 2;
```

### 3. Top 10 sales days
```sql
select sales_date, sum(gross_amount) as "Daily_Revenue" 
from sales
group by 1
order by 2 desc
limit 10;
```

###  4. Count of sales as per sales date
```sql
select count(sales_id) as "Total_Count", sales_date
from sales 
group by 2
order by 1;
```

### 5. Top 10 best selling products
```sql
select p.product_name, p.category, sum(s.gross_amount) as "Total_Revenue", sum(s.quantity) as "Total_Quantity_Sold"
from product p 
join sales s 
on p.product_id = s.product_id
group by 1,2
order by 4 desc
limit 10;
```

### 6. Revenue by category
```sql
select p.category, sum(s.gross_amount) as "Total_Revenue_by_Category"
from product p 
join sales s 
on p.product_id = s.product_id
group by 1
order by 2 desc;
```

###  7. Top 10 customers by revenue
```sql
select c.first_name, c.last_name,c.city, c.loyalty_tier, sum(s.gross_amount) as "Total_Revenue" 
from customer c 
join sales s 
on c.customer_id = s.customer_id
group by 1, 2
order by 3 desc
limit 10;
```

### 8. Customers by city
```sql
select city, count(*) as "Count_Of_Customer"
from customer
group by 1
order by 2 desc;
```

###  9.  sales - customers by city
```sql
select c.city, count(distinct c.customer_id) as "Total_Customers", sum(s.gross_amount) as "Total_Revenue"
from customer c
join sales s 
on c.customer_id = s.customer_id
group by 1
order by 3 desc;
```

###  10. Best Performing store
```sql
select st.store_name, st.city, count(s.sales_id) as "Total_Count", sum(s.gross_amount) as "Total_Revenue"
from store st 
join sales s 
on st.store_id = s.store_id
group by 1, 2
order by 4 desc;
```

###  11. Revenue by State
```sql
select st.state, count(s.sales_id) as "Total_Count", sum(s.gross_amount) as "Total_Revenue"
from store st 
join sales s 
on st.store_id = s.store_id
group by 1
order by 3 desc;
```

###  12. Are there any Low Stock Product
```sql
select * 
from inventory
where on_hand_qty <= reorder_point;
```

###  13. Identify which products have low stocks
```sql
select p.product_name, p.category, i.on_hand_qty, i.reorder_point
from inventory i 
join product p 
on i.product_id = p.product_id 
where i.on_hand_qty <= i.reorder_point;
```

###  14. Stock Value By Category
```sql
select p.category, sum(i.on_hand_qty) as "Total_Stock", sum(i.on_hand_qty*p.unit_price) as "Stock_Value"
from inventory i 
join product p 
on p.product_id = i.product_id
group by 1
order by "Stock_Value" desc;
```

###  15. Employees by Store
```sql
select count(distinct e.employee_id) as "No. of Employees", e.store_id, st.store_name
from employee e
left join store st 
on st.store_id = e.store_id
group by 2,3
order by 1 desc;
```

### 16.  Active vs Terminated Employees
```sql
select count(*) as "no. of Employees",
    ( case 
        when termination_date is null then 'Active'
        else 'Terminated'
    end ) as "Employee_Status"
from employee
group by "Employee_Status"
order by 1 desc;
```

###  17.  Revenue by Payment Method
```sql
select payment_type, sum(gross_amount) as "Total_Revenue_by_Pay_Method"
from sales
group by 1
order by "Total_Revenue_by_Pay_Method" desc;
```

###  18. which payment type is used most often
```sql
select payment_type, count(*) as "Transaction_Count"
from sales
group by 1
order by 2 desc;
```

###  19.  Weekend vs Weekday Sales
```sql
select sum(gross_amount) as "Total_Revenue", count(*) as "No._of_Transactions", count(distinct sales_date) as "No._of_Days", 
sum(gross_amount) / count(distinct sales_date) as "Avg_Revenue_per_Day",
    (case
        when dayofweek(sales_date) in (0,6) then 'Weekend'
        else 'Weekday'
    end) as "Day_Type"
from sales
group by "Day_Type";
```

###  20. Highest Revenue Category
```sql
select p.category, sum(s.gross_amount) as "Total_Revenue"
from product p 
join sales s 
on p.product_id = s.product_id
group by 1
order by 2 desc;
```

###  21.  Products never sold
```sql
select p.product_id, p.product_name, p.brand, p.category, p.is_active
from product p
left join sales s
on p.product_id = s.product_id
where s.product_id is null
order by p.category;
```

###  22.  Repeat Customers and One Time Customers
```sql
with customer_orders as (
    select customer_id, count(*) as "no._of_orders"
    from sales
    group by customer_id
)
    select count(*) as "No._Of_Customers",
        (case
            when "no._of_orders" > 1 then 'Repeat'
            else 'One-Time'
        end) as "Customer_Type"
    from customer_orders
    group by "Customer_Type";
```

###  23. Average order value by city
```sql
select c.city, avg(s.gross_amount) as "Avg_Order_Value", sum(s.gross_amount) as "Total_Revenue"
from customer c 
join sales s 
on c.customer_id = s.customer_id
group by 1
order by "Avg_Order_Value" desc;
```

###  24. average discount % given per loyalty tier
```sql
select c.loyalty_tier, avg(s.discount_amount/nullif(s.gross_amount, 0))*100 as "Avg_Discount"
from sales s 
join customer c 
on c.customer_id = s.customer_id
group by 1
order by 2;
```

###  25. Loyalty tier performance
```sql
select c.loyalty_tier, count(distinct c.customer_id) as "No._of_Customers", count(*) as "No._of_Orders", sum(s.gross_amount) as "Gross_Revenue",
avg(s.gross_amount) as "Avg_Order_Value", avg(s.discount_amount) as "Avg_Discount", count(*) / count(distinct c.customer_id) as "Avg_Orders_Per_Customer",
count(s.gross_amount) / count(distinct c.customer_id) as "Avg_Revenue_Per_Customer"
from sales s 
join customer c 
on s.customer_id = c.customer_id
group by 1
order by 3 desc;
```

###  26. out of stock products
```sql
with latest_inventory as (
    select store_id, product_id, on_hand_qty, on_order_qty
    from inventory
    qualify row_number() over (partition by store_id, product_id order by snapshot_date desc) = 1
)
select s.store_name, s.city, p.product_name, p.category, i.on_hand_qty, i.on_order_qty
from latest_inventory i 
join product p 
on i.product_id = p.product_id
join store s 
on i.store_id = s.store_id
where i.on_hand_qty = 0
order by i.on_order_qty desc;
```

# Key Findings
- ₹98.7 Cr total gross revenue across the dataset period
- Televisions is the top revenue category (₹20.9 Cr), followed by Cameras and Laptops
- 37% of products (4,397 SKUs) have never been sold — wide catalog, low per-SKU turnover
- Top spender: Diya Verma (Shimla, Silver tier) — ₹11.5L across 5 orders
- Ludhiana has the highest AOV (₹1,11,563) while Mumbai has the lowest (₹64,116)
- Weekend vs. weekday revenue is nearly identical — no strong day-of-week pattern
- 6.24% employee attrition (749 of 12,000 employees terminated)

🛠️ Tech Stack
Database: Snowflake
Language: SQL
Data Format: CSV
