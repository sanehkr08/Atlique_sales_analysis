# Atlique_sales_analysis
<p>This repository possesses the information about SQL sales analysis of Atlique harware company. It consist of SQL queries to retrieve  required insights.

Domain:  Consumer Goods | Function: Executive Management
Problem Statement: 
Atliq Hardwares (imaginary company) is one of the leading computer hardware producers in India and well expanded in other countries too.

However, the management noticed that they do not get enough insights to make quick and smart data-informed decisions. They want to expand their data analytics team by adding several junior data analysts. Tony Sharma, their data analytics director wanted to hire someone who is good at both tech and soft skills. Hence, he decided to conduct a SQL challenge which will help him understand both the skills.

Task:  

Imagine yourself as the applicant for this role and perform the following task

1.    Check ‘ad-hoc-requests.pdf’ - there are 10 ad hoc requests for which the business needs insights.
2.    You need to run a SQL query to answer these requests. 
3.    The target audience of this dashboard is top-level management - hence you need to create a presentation to show the insights.
</p>
<p>Other resources Provided:

Q1. . Provide the list of markets in which customer "Atliq Exclusive" operates its
      business in the APAC region.
      
```mysql
select  market from dim_customer where region = 'APAC'
```
Q2.  What is the percentage of unique product increase in 2021 vs. 2020? The
     final output contains these fields,
      unique_products_2020
      unique_products_2021
      percentage_chg
      
```mysql
select count( distinct case when fiscal_year = 2020 then product_code end) as unique_product_2020,
  count( distinct case when fiscal_year = 2021 then product_code end) as unique_product_2021,
  
  (count( distinct case when fiscal_year = 2021 then product_code end)-count( distinct case when fiscal_year = 2020 then product_code end))/count( distinct case when fiscal_year = 2020 then product_code end)*100 as percentage_chng
  
  from fact_sales_monthly
```
Q3. Provide a report with all the unique product counts for each segment and
    sort them in descending order of product counts. The final output contains
    2 fields,
    segment
    product_count
    
```mysql
select count(distinct product_code) as product_count , segment from dim_product
group by segment
order by product_count desc
```
            
Q4. Follow-up: Which segment had the most increase in unique products in
    2021 vs 2020? The final output contains these fields,
    segment
    product_count_2020
    product_count_2021
    difference
    
    
```mysql
select count(distinct case when T1.fiscal_year = 2020 then T1.product_code end) as product_count_2020,
count(distinct case when T1.fiscal_year = 2021 then T1.product_code end) as product_count_2021,
T2.segment,
count(distinct case when T1.fiscal_year = 2021 then T1.product_code end)-count(distinct case when T1.fiscal_year = 2020 then T1.product_code end) as difference
from fact_sales_monthly as T1
 inner join dim_product as T2
on T1.product_code = T2.product_code
group by segment
order by difference desc
```    

Q5. Get the products that have the highest and lowest manufacturing costs.
    The final output should contain these fields,
    product_code
    product
    manufacturing_cost
    
```mysql
select t1.product_code , t1.product , t2.manufacturing_cost
from dim_product as t1
join fact_manufacturing_cost as t2
on t1.product_code = t2.product_code
where manufacturing_cost = (select max(manufacturing_cost) from fact_manufacturing_cost)
union
select t1.product_code , t1.product , t2.manufacturing_cost
from dim_product as t1
join fact_manufacturing_cost as t2
on t1.product_code = t2.product_code
where manufacturing_cost = (select min(manufacturing_cost) from fact_manufacturing_cost)
```
            
  Q6. Generate a report which contains the top 5 customers who received an
      average high pre_invoice_discount_pct for the fiscal year 2021 and in the
      Indian market. The final output contains these fields,
      customer_code
      customer
      average_discount_percentage
      
```mysql
select t1.customer_code, t2.customer , t1.pre_invoice_discount_pct from fact_pre_invoice_deductions as t1
join dim_customer as t2
on t1.customer_code = t2.customer_code
where pre_invoice_discount_pct > (select avg(pre_invoice_discount_pct) from fact_pre_invoice_deductions) and t2.market = "India"
order by pre_invoice_discount_pct desc
limit 5
```

            
Q7.  Get the complete report of the Gross sales amount for the customer “Atliq
      Exclusive” for each month. This analysis helps to get an idea of low and
      high-performing months and take strategic decisions.
      The final report contains these columns:
      Month
      Year
      Gross sales Amount
      
```mysql
select sum(a.gross_price * b.sold_quantity) as gross_sale_amount , b.fiscal_year , b.month_column , c.customer
from fact_gross_price as a
join fact_sales_monthly as b
on a.product_code = b.product_code
join dim_customer as c on b.customer_code = c.customer_code
where c.customer = 'Atliq Exclusive'
group by b.month_column , b.fiscal_year
order by b.fiscal_year ;
```

Q8. In which quarter of 2020, got the maximum total_sold_quantity? The final
    output contains these fields sorted by the total_sold_quantity,
    Quarter
    total_sold_quantity
    
```mysql
SELECT 
CASE
    WHEN date BETWEEN '2019-09-01' AND '2019-11-01' then CONCAT('[',1,'] ',MONTHNAME(date))  
    WHEN date BETWEEN '2019-12-01' AND '2020-02-01' then CONCAT('[',2,'] ',MONTHNAME(date))
    WHEN date BETWEEN '2020-03-01' AND '2020-05-01' then CONCAT('[',3,'] ',MONTHNAME(date))
    WHEN date BETWEEN '2020-06-01' AND '2020-08-01' then CONCAT('[',4,'] ',MONTHNAME(date))
    END AS Quarters,
    SUM(sold_quantity) AS total_sold_quantity
FROM fact_sales_monthly
WHERE fiscal_year = 2020
GROUP BY Quarters
```
Q9. Which channel helped to bring more gross sales in the fiscal year 2021
    and the percentage of contribution? The final output contains these fields,
    channel
    gross_sales_mln
    percentage
    
```mysql
with output as
(
select  a.channel , round(sum(b.gross_price*c.sold_quantity/1000000),2) as gross_sale_mln
from fact_sales_monthly as c join dim_customer as a on c.customer_code = a.customer_code
                                           join fact_gross_price as b on c.product_code = b.product_code
where c.fiscal_year = 2021
group by a.channel
)
select channel , concat(gross_sale_mln , 'M') as gross_sale_mln , concat(round(gross_sale_mln*100/total , 2) , '%') as percentage
from
(
(select sum(output.gross_sale_mln) as total from output) as A , 
(select * from output) as B
)
order by percentage desc
```


Q10. Get the Top 3 products in each division that have a high
      total_sold_quantity in the fiscal_year 2021? The final output contains these
      fields,
      division
      product_code
      product
      total_sold_quantity
      rank_order
      
```mysql
WITH Output1 AS 
(
SELECT P.division, FS.product_code, P.product, SUM(FS.sold_quantity) AS Total_sold_quantity
FROM dim_product P JOIN fact_sales_monthly FS
ON P.product_code = FS.product_code
WHERE FS.fiscal_year = 2021 
GROUP BY  FS.product_code, division, P.product
),
Output2 AS 
(
SELECT division, product_code, product, Total_sold_quantity,
        RANK() OVER(PARTITION BY division ORDER BY Total_sold_quantity DESC) AS 'Rank_Order' 
FROM Output1
)
 SELECT Output1.division, Output1.product_code, Output1.product, Output2.Total_sold_quantity, Output2.Rank_Order
 FROM Output1 JOIN Output2
 ON Output1.product_code = Output2.product_code
WHERE Output2.Rank_Order IN (1,2,3)
```
    

