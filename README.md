/*
SQL-Portfolio-Project to Analyze Transactions Estimation

Skills used: Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types

*/

use the dataset : https://www.kaggle.com/datasets/thedevastator/analyzing-credit-card-spending-habits-in-india

--change the column names to lower case before importing data to sql server.Also replace space within column names with underscore.

alter table credit_card_transactions alter column transaction_id int
alter table credit_card_transactions alter column city varchar(30)
alter table credit_card_transactions alter column card_type varchar(20)
alter table credit_card_transactions alter column transaction_date date
alter table credit_card_transactions alter column exp_type varchar(20)
alter table credit_card_transactions alter column gender varchar(10)
alter table credit_card_transactions alter column amount INT

write 4-6 queries to explore the dataset and put your findings 
select * from credit_card_transactions
select top 5 * from credit_card_transactions
select distinct * from credit_card_transactions
select sum(amount) as total_amount from credit_card_transactions
select datepart(year,transaction_date) as yod from credit_card_transactions



---Question 1 : write a query to print top 5 cities with highest spends
--and their percentage contribution of total credit card spends

with A as(
select city,sum(amount) as total1 from credit_card_transactions
group by city),
B as(select sum(cast(amount as BIGINT)) as total2 from credit_card_transactions)
select top 5 A.*,round(total1 * 1.0 /total2 * 100,2) as percentage_distribution 
from A inner join B
on 1=1
order by total1 desc

---Question 2---write a query to print highest spend month and 
--amount spent in that month for each card type
use namastesql;


with A as(
select card_type,sum(cast(amount as BIGINT)) as t,
rank() over(partition by card_type order by sum(amount) desc) as rn,
datepart(year,transaction_date) as yo,datepart(month,transaction_date) as mo
from credit_card_transactions
group by card_type,datepart(year,transaction_date),
datepart(month,transaction_date))
select card_type,yo,mo,t,rn from A
where rn = 1




---Question 3 : write a query to print the transaction details(all columns from the table) 
--for each card type when it reaches a cumulative of 1000000 total spends
--(We should have 4 rows in the o/p one for each card type)
with A as(
select *,
sum(amount) over(partition by card_type order by transaction_date,
transaction_id) as running_sum from credit_card_transactions)
select * from (select *,rank() over(partition by card_type 
order by running_sum) as rn
from A where running_sum >= 1000000)a where rn = 1


---Question 4 : write a query to find city 
---which had lowest percentage spend for gold card type
use namastesql;
select * from credit_card_transactions
with cte as (
select top 1 city,card_type,sum(amount) as amount
,sum(case when card_type='Gold' then amount end) as gold_amount
from credit_card_transactions
group by city,card_type)
select 
city,sum(gold_amount)*1.0/sum(amount) as gold_ratio
from cte
group by city
order by gold_ratio

-----Question 5 :write a query to print 3 columns:  
---city, highest_expense_type , lowest_expense_type 
---(example format : Delhi , bills, Fuel)
use namastesql;
select * from credit_card_transactions
with A as(
select city,exp_type,sum(amount) as total from credit_card_transactions
group by city,exp_type)
select city,max(case when rn_asc=1 then exp_type end) as highest_exp_type,
min(case when rn_desc=1 then exp_type end) as lowest_exp_type
from
(select *,
rank() over(partition by city order by total desc) as rn_desc,
rank() over(partition by city order by total asc) as rn_asc
from A) B
group by city


 ----Question 6 :write a query to find percentage contribution of spends 
 ---by females for each expense type
 use namastesql;

 select exp_type,
 sum(case when gender = 'F' then amount else 0 end)*1.0/sum(amount) as per_dis
 from credit_card_transactions
 group by exp_type
order by per_dis desc


----Question 7 : which card and expense type combination saw highest month 
---over month growth in Jan-2014
use namastesql;
select * from credit_card_transactions



with A as
(select card_type,exp_type,datepart(month,transaction_date) as mo,
datepart(year,transaction_date) as yo,sum(amount) as total
from credit_card_transactions
group by datepart(month,transaction_date),datepart(year,transaction_date),
card_type,exp_type
)
select top 1 *,(total-prev_month) as diff_month from(select *,lag(total,1) 
over(partition by card_type,exp_type order by yo,mo) 
as prev_month from A)B
WHERE prev_month is not null and yo = 2014 AND mo = 1
order by diff_month desc


---Question 8 : during weekends which city has highest total spend 
--to total no of transcations ratio
use namastesql
select * from credit_card_transactions
select top 1 city,sum(amount) * 1.0 / count(*) as ratio
from credit_card_transactions
where datepart(weekday,transaction_date) in (1,7)
group by city
order by ratio desc


----Question 9 : which city took least number of days to reach its 500th transaction 
---after the first transaction in that city
use namastesql
with A as(
select *,row_number() over(partition by city order by transaction_date,transaction_id)
as rn from credit_card_transactions)
select top 1 city,datediff(day,min(transaction_date),max(transaction_date))
as datediff1 from A
where rn = 1 or rn = 500
group by city
having count(1) = 2
order by datediff1
