/*
SQL-Portfolio-Project to Analyze Sales Growth  

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
