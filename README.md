# Creadit_Card_Transaction_Analysis
--1- write a query to print top 5 cities with highest spends and their percentage contribution of total credit card spends 

select top 5 city,sum(amount) as t_a ,
round((100.0*sum(amount)/(select sum(cast(amount as bigint)) from credit_card_transcations)), 2) as perc
from credit_card_transcations
group by city
order by t_a desc;


--2- write a query to print highest spend month and amount spent in that month for each card type
with a as (select card_type, sum(amount) as t_spend, datepart(month, transaction_date) as month_,datepart(yy, transaction_date)  as year_ from credit_card_transcations
group by card_type,datepart(yy, transaction_date), datepart(month, transaction_date))
select * from (select *, rank() over(partition by card_type order by t_spend desc) as r_ from a) b 
where r_ = 1;


--3- write a query to print the transaction details(all columns from the table) for each card type when
--it reaches a cumulative of 1000000 total spends(We should have 4 rows in the o/p one for each card type)
select t.* from 
(select *, sum(amount) over(partition by card_type order by transaction_date,transaction_id ) as t_spend from credit_card_transcations) t
inner join (select * from 
(select *, rank() over(partition by card_type order by t_sum asc) as r_ from 
(select card_type, sum(amount) over(partition by card_type order by transaction_date,transaction_id) as t_sum from credit_card_transcations) a
where t_sum >= 1000000) b
where r_ = 1) c on c.t_sum = t.t_spend;


--4- write a query to find city which had lowest percentage spend for gold card type
select top 1 city, card_type, sum(amount) as t_sum, round((1.0*sum(amount)/(select sum(amount) from credit_card_transcations
where card_type = 'Gold')) , 10) as perc 
from credit_card_transcations where card_type = 'Gold' group by city,card_type 
order by perc asc;



--5- write a query to print 3 columns:  city, highest_expense_type , lowest_expense_type (example format : Delhi , bills, Fuel)
select a.city,a.low_exp,b.high_exp from
(select city, exp_type as low_exp,sum(amount) as low_ex, 
rank() over(partition by city order by sum(amount)) as r_
from credit_card_transcations group by city, exp_type) a inner join
(select city, exp_type as high_exp,sum(amount) as high_ex,
rank() over(partition by city order by sum(amount)desc) as ra_ from credit_card_transcations group by city, exp_type) b
on a.city = b.city and a.r_ = b.ra_
where a.r_ = 1 and b.ra_ = 1;



--6- write a query to find percentage contribution of spends by females for each expense type
with a as(select exp_type, gender, sum(amount) as t_s from credit_card_transcations where gender = 'F' group by exp_type,gender)
,b as (select exp_type, sum(amount) as t_a from credit_card_transcations group by exp_type )
select a.*, round((1.0*a.t_s/b.t_a) , 2) as f_c_s from a join b on b.exp_type=a.exp_type
order by f_c_s desc;



--7- which card and expense type combination saw highest month over month growth in Jan-2014
with a as (select card_type,exp_type,sum(amount) as t_s,DATEPART(mm,transaction_date) as m,
DATEPART(yy,transaction_date) as y from credit_card_transcations 
group by exp_type,card_type,
DATEPART(yy,transaction_date),DATEPART(mm,transaction_date)),
b as (select *,lag(t_s,1) over(partition by card_type,exp_type order by y, m) as l_s from a)
,c as (select * from b )--where l_s is not null and m = 1 and  y =  2014) 
select top 1 *, (t_s - l_s) as mom  from c 
order by mom desc;



--9- during weekends which city has highest total spend to total no of transcations ratio 

with a as(select city, sum(amount) as t_a,count(city) as c 
from credit_card_transcations 
where datename(WEEKDAY,transaction_date) in ('Saturday','Sunday')
group by city)
select a.*,(1.0*t_a/c) as ratio from a 
order by ratio desc;



--10- which city took least number of days to reach its 500th transaction after the first transaction in that city
with a as (select city,transaction_date, count(transaction_id) over(partition by city order by transaction_date,transaction_id) 
as c  from credit_card_transcations)
,b as (select city,transaction_date  from a where c = 1)
, d as (select city,transaction_date from a where c = 500)
select b.*,d.transaction_date, DATEDIFF(day,b.transaction_date,d.transaction_date) as datediff_ from b inner join d on b.city = d.city
order by datediff_ asc;
