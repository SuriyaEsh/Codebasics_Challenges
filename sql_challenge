#1.list of markets in which customer "Atliq Exclusive" operates its business in the APAC region

select distinct market 
from gdb023.dim_customer
where customer = "Atliq Exclusive" and region = 'APAC'
order by market

#2.Percentage of unique product increase in 2021 vs 2020?
## output should contain these fields:unique_products_2020, unique_products_2021, percentage_chg

SELECT COUNT(DISTINCT CASE WHEN fiscal_year = 2020 THEN product_code END) AS product_count_2020,
       COUNT(DISTINCT CASE WHEN fiscal_year = 2021 THEN product_code END) AS product_count_2021,
       ROUND((COUNT(DISTINCT CASE WHEN fiscal_year = 2021 THEN product_code END) -
            COUNT(DISTINCT CASE WHEN fiscal_year = 2020 THEN product_code END))/ 
        COUNT(DISTINCT CASE WHEN fiscal_year = 2020 THEN product_code END) * 100,2) AS pct_change
FROM 
    gdb023.fact_sales_monthly;

#3.unique product counts for each segment 

SELECT segment, count(distinct product_code) as'product_count'
FROM gdb023.dim_product
group by segment
order by product_count desc

#4. Which segment had the most increase in unique products in 2021 vs 2020?
#output contains these fields: segment, product_count_2020, product_count_2021, difference

with cte1 as 
(SELECT segment, count(distinct dp1.product_code) as'product_count_2020'
FROM gdb023.dim_product dp1
join gdb023.fact_sales_monthly fm1
on dp1.product_code=fm1.product_code
where fiscal_year = 2020
group by segment),
cte2 as 
(SELECT segment, count(distinct dp2.product_code) as'product_count_2021'
FROM gdb023.dim_product dp2
join gdb023.fact_sales_monthly fm2
on dp2.product_code=fm2.product_code
where fiscal_year = 2021
group by segment)

select cte2.segment,product_count_2020, product_count_2021,
					(product_count_2021-product_count_2020) as product_difference
from cte1 join cte2 on cte1.segment=cte2.segment
order by product_difference desc

#5. Get the products that have the highest and lowest manufacturing costs.
# output should contain these fields:product_code, product manufacturing_cost
 
 (select d.product_code,d.product, sum(manufacturing_cost) as manufacturing_cost
 from gdb023.dim_product d
 join gdb023.fact_manufacturing_cost f
 on d.product_code = f.product_code
 group by d.product_code,d.product
 order by manufacturing_cost desc
 limit 1)
 union
 (select d.product_code,d.product, sum(manufacturing_cost) as manufacturing_cost
 from gdb023.dim_product d
 join gdb023.fact_manufacturing_cost f
 on d.product_code = f.product_code
 group by d.product_code,d.product
 order by manufacturing_cost 
 limit 1)
 
 #6. top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market. 
#output contains these fields: customer_code, customer, average_discount_percentage

SELECT dc.customer_code,dc.customer,round(avg(f.pre_invoice_discount_pct),3)x as 'average_discount_percentage'
from gdb023.dim_customer dc
join gdb023.fact_pre_invoice_deductions f
on dc.customer_code = f.customer_code
where f.fiscal_year = 2021 and dc.market = 'India'
group by dc.customer_code,dc.customer
order by average_discount_percentage desc
limit 5

#7.Gross sales amount for the customer “Atliq Exclusive” for each month . 
#This analysis helps to get an idea of low and high-performing months and take strategic decisions.
# The final report contains these columns: Month Year Gross sales Amount

SELECT concat(monthname(fm.date),' ',year(fm.date))as month_year,fm.fiscal_year,
		round((sum(fp.gross_price*fm.sold_quantity)/1000000),2) as 'Gross_Sales_Amount(Millions)'
FROM gdb023.dim_customer dc
join gdb023.fact_sales_monthly fm
on dc.customer_code = fm.customer_code
join gdb023.fact_gross_price fp
on fm.product_code = fp.product_code
where dc.customer = 'Atliq Exclusive'
group by concat(monthname(fm.date),' ',year(fm.date)), fm.fiscal_year
order by fiscal_year

#8.In which quarter of 2020, got the maximum total_sold_quantity?  
# output contains fields sorted by the total_sold_quantity, Quarter, total_sold_quantity 

select concat('Q','',case when month(fm.date) between 1 and 3 then 1
							when month(fm.date) between 4 and 6 then 2
                            when month(fm.date) between 7 and 9 then 3
                            when month(fm.date) between 10 and 12 then 4 end) as 'Quarter',
                             sum(sold_quantity) as 'Qty_sold'
from gdb023.dim_product dp
join gdb023.fact_sales_monthly fm
on dp.product_code = fm.product_code
group by concat('Q','',case when month(fm.date) between 1 and 3 then 1
							when month(fm.date) between 4 and 6 then 2
                            when month(fm.date) between 7 and 9 then 3
                            when month(fm.date) between 10 and 12 then 4 end)
order by Qty_sold desc


#9.Which channel helped to bring more gross sales in the fiscal year 2021&the percentage of contribution? 
#output contains these fields: channel, gross_sales_mln, percentage

with cte1 as
(select dc.channel, round((sum(fp.gross_price*fm.sold_quantity)/1000000),2) as gross_sales_mln
from gdb023.dim_customer dc
join gdb023.fact_sales_monthly fm
on dc.customer_code = fm.customer_code
join gdb023.fact_gross_price fp
on fm.product_code = fp.product_code
where fp.fiscal_year = 2021
group by dc.channel),
cte2 as
(select round((sum(fp.gross_price*fm.sold_quantity)/1000000),2) as gross_sales_total_mln
from gdb023.dim_customer dc
join gdb023.fact_sales_monthly fm
on dc.customer_code = fm.customer_code
join gdb023.fact_gross_price fp
on fm.product_code = fp.product_code
where fp.fiscal_year = 2021)

select cte1.channel, cte1.gross_sales_mln, 
	(gross_sales_mln/gross_sales_total_mln)*100 as pct_contribution
from cte1 join cte2
order by pct_contribution

#10.Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021?
#The final output contains fields, division, product_code

with cte as (select dp.product_code, dp.division, sum(fm.sold_quantity) as total_qty,
		rank()over(partition by division order by sum(fm.sold_quantity) desc) as 'ranks'
from gdb023.dim_product dp
join gdb023.fact_sales_monthly fm
on dp.product_code=fm.product_code
where fiscal_year = 2021
group by dp.product_code, dp.division)

select product_code,division,total_qty from cte where ranks <=3 




