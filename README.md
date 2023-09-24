# Credit Card Transaction Analysis

This is a SQL analysis of credit card transaction dataset.
With the use of this data we analysed the trends of transactions 
and how can we improve the engagement of credit card users in increasing transaction counts.

### To determine which time of the day did the credit card users shown the activity (early_morning, morning, late-morning, afternoon, evening, night, mid-night).

````sql
select 
SUM(CASE WHEN TIME(t.date) BETWEEN '03:00:00' and '06:00:00' THEN 1 ELSE 0 END) 'Early Morning' ,/*em = 3.00 am - 6.00 am*/
SUM(CASE WHEN TIME(t.date) BETWEEN '06:00:00' and '09:00:00' THEN 1 ELSE 0 END) 'Morning', /*m = 6.00 am - 9.00 am*/
SUM(CASE WHEN TIME(t.date) BETWEEN '09:00:00' and '12:00:00' THEN 1 ELSE 0 END) 'Late Morning', /*lm = 9.00 am - 12.00 pm*/
SUM(CASE WHEN TIME(t.date) BETWEEN '12:00:00' and '15:00:00' THEN 1 ELSE 0 END) 'Afternoon', /*af = 12.00 am - 3.00 pm*/
SUM(CASE WHEN TIME(t.date) BETWEEN '15:00:00' and '19:00:00' THEN 1 ELSE 0 END) 'Evening',/*e = 3.00 pm - 7.00 pm*/
SUM(CASE WHEN TIME(t.date) BETWEEN '19:00:00' and '24:00:00' THEN 1 ELSE 0 END) 'Night', /*n = 7.00 pm - 12.00 pm*/
SUM(CASE WHEN TIME(t.date) BETWEEN '00:01:00' and '03:00:00' THEN 1 ELSE 0 END) 'Mid Night', /*mn = 12.00 am - 3.00 am */
COUNT(*) TOTAL
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card 
JOIN merchant m JOIN merchant_category mc ON m.id_merchant_category = mc.id  
ON t.id_merchant = m.id
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/1ce07e8d-34e6-4825-9a04-30bc569c6e80)

It lets us know the trends related to transaction by users and when users are more active in a particular day of time. 

### To determine the count of total transactions done by a user  in a year with their average spending capacities and count of those transactions which amount to above average spendings.  

````sql
WITH query1 as 
(select ch.id, ch.name,round(t.amount,2) amount,
ROUND(avg(t.amount) OVER (partition by ch.id),2) avg_spending,
COUNT(ch.id) OVER(partition by ch.id) total_transaction
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card 
JOIN merchant m JOIN merchant_category mc ON m.id_merchant_category = mc.id  
ON t.id_merchant = m.id ) 

select id,name,avg_spending,total_transaction, transactions_above_avg
from (
select *, count(name) OVER(partition by name order by id) transactions_above_avg,   
ROW_NUMBER() OVER (partition by id order by amount DESC) row_num from query1
where avg_spending < amount
order by id) t2
where t2.row_num=1;
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/6fda47a6-d40f-4336-a44b-d5bc453b1f5d)

This result show the average spending capacity of a user, which in turn provides us a baseline for a user. It can provide information related to fraud transactions too as the transactions made in a day above average spending capacity has a good probability of a fraud transaction of a theft of card.

### To determine the transactions which were above average spending capacity of the specific user on a specific day which can be checked for the fraud payment.

````sql
WITH query1 as 
(select * from
(select ch.id,t.date, ch.name,round(t.amount,2) amount,
ROUND(avg(t.amount) OVER (partition by ch.id),2) avg_spending,
COUNT(ch.id) OVER(partition by ch.id) total_transaction
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card 
JOIN merchant m JOIN merchant_category mc ON m.id_merchant_category = mc.id  
ON t.id_merchant = m.id ) x
where x.avg_spending < x.amount)


select *, count(name) OVER(partition by name order by id) transactions_above_avg from query1
order by id

````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/74b521a3-100d-477b-8ade-333c4bc73237)


### To determine which merchant had a the maximum use of the credit card issued by the company, which can later be used to give discounts so that user use the cards more often on them.

````sql
select m.id,m.name,mc.name, ROUND(sum(t.amount),2) total_spending_annual
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card 
JOIN merchant m JOIN merchant_category mc ON m.id_merchant_category = mc.id  
ON t.id_merchant = m.id 
group by 1,2,3
order by 4 DESC;
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/f9a5a462-9ccd-4ce7-bed6-c0622532497c)

Merchant wise transactions and amounts of transactions can give us an insight about where the card is being mostly used and rarely used, depending on the credit card features we can allow some discounts or offers to improve in the rare merchant type or take more advantage from a specific merchant.

### To determine the category of merchant where the annual payments amounted highest, and this can allow us to check the areas that can provide us more profit by putting more exciting offers on specific card.

````sql
select mc.name, ROUND(sum(t.amount),2) total_spending_annual
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card 
JOIN merchant m JOIN merchant_category mc ON m.id_merchant_category = mc.id  
ON t.id_merchant = m.id 
group by 1
order by 2 DESC
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/236907e3-e895-4415-ae0e-034510989b38)


### To determine monthly transactions count with respect to merchant category, to keep a check on the usage of cards on a monthly basis.

````sql
select 
MONTHNAME(t.date) as MONTHS,
COUNT(CASE WHEN mc.name = 'pub' THEN 'PUB'END) PUB_TRANCSACTION_COUNT,
COUNT(CASE WHEN mc.name = 'restaurant' THEN 'RESTAURANT' END) RESTAURANT_TRANCSACTION_COUNT,
COUNT(CASE WHEN mc.name = 'bar' THEN 'BAR'END) BAR_TRANCSACTION_COUNT,
COUNT(CASE WHEN mc.name = 'food truck' THEN 'FOODTRUCK'END) FOODTRUCK_TRANCSACTION_COUNT,
COUNT(CASE WHEN mc.name = 'coffee shop' THEN 'COFFEE SHOP' END) COFFEESHOP_TRANCSACTION_COUNT,
COUNT(MONTHNAME(t.date)) as MONTHS_TOTAL
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card 
JOIN merchant m JOIN merchant_category mc ON m.id_merchant_category = mc.id  
ON t.id_merchant = m.id 
group by 1
order by 7 DESC;
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/1ba33b01-0972-469a-b6d0-65981b992eb8)


### To determine total monthly transactions with respect to merchant category, to keep a check on the usage of cards on a monthly basis and which month performed best category wise.

````sql
select 
MONTHNAME(t.date)as MONTHS,
ROUND(SUM(CASE WHEN mc.name = 'pub' THEN t.amount END),2) PUB_TRANCSACTION_AMOUNT,
ROUND(SUM(CASE WHEN mc.name = 'restaurant' THEN t.amount END),2) RESTAURANT_TRANCSACTION_AMOUNT,
ROUND(SUM(CASE WHEN mc.name = 'bar' THEN t.amount END),2) BAR_TRANCSACTION_AMOUNT,
ROUND(SUM(CASE WHEN mc.name = 'food truck' THEN t.amount END),2) FOODTRUCK_TRANCSACTION_AMOUNT,
ROUND(SUM(CASE WHEN mc.name = 'coffee shop' THEN t.amount  END),2) COFFEESHOP_TRANCSACTION_AMOUNT,
ROUND(SUM(t.amount),2) as MONTHS_TOTAL
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card 
JOIN merchant m JOIN merchant_category mc ON m.id_merchant_category = mc.id  
ON t.id_merchant = m.id 
group by 1
order by 7 DESC;
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/495106a5-ba1a-4038-a516-7a2c5cfdc7cf)


### To determine the number of transactions done by specific user on a monthly basis in a year to check the card usage trend.

````sql
select 
ch.name,
count(ch.id) total_transaction,
SUM(CASE 
WHEN MONTH(t.date) = '01' THEN 1 ELSE 0 END) JAN,
SUM(CASE
WHEN MONTH(t.date) = '02'THEN 1 ELSE 0 END) FEB, 
SUM(CASE 
WHEN MONTH(t.date) = '03' THEN 1 ELSE 0 END) MAR,
SUM(CASE
WHEN MONTH(t.date) = '04' THEN 1 ELSE 0 END) APR,
SUM(CASE 
WHEN MONTH(t.date) = '05' THEN 1 ELSE 0 END) MAY, 
SUM(CASE
WHEN MONTH(t.date) = '06' THEN 1 ELSE 0 END) JUN,
SUM(CASE 
WHEN MONTH(t.date) = '07' THEN 1 ELSE 0 END) JUL,
SUM(CASE
WHEN MONTH(t.date) = '08' THEN 1 ELSE 0 END) AUG,
SUM(CASE
WHEN MONTH(t.date) = '09' THEN 1 ELSE 0 END) SEP,
SUM(CASE
WHEN MONTH(t.date) = '10' THEN 1 ELSE 0 END) OCT,
SUM(CASE
WHEN MONTH(t.date) = '11' THEN 1 ELSE 0 END) NOV ,
SUM(CASE
WHEN MONTH(t.date) = '12' THEN 1 ELSE 0 END) 'DEC'

from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card 
JOIN merchant m JOIN merchant_category mc ON m.id_merchant_category = mc.id  
ON t.id_merchant = m.id 
group by 1
order by 2;
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/8b9e24f1-b64b-48a5-9c5d-6e9ee11d9e06)

This trend let us know about how the users have been interacting with the credit cards on a monthly basis, we can communicate with the users who use the card less and understand the reason for the rare usage of teh credit card.

### To determine highest credit card transacting user in each month with their card details and transaction count.

````sql
WITH query1 as (
select 
MONTHNAME(t.date)as months,
ch.id,ch.name,cc.card,
count(ch.id) over (partition by ch.id, month(t.date))  transaction_count
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card
order by 4 DESC)

select distinct months,id,name,card,transaction_count
from(
select *,
RANK() OVER(partition by months order by transaction_count DESC) rnk 
from query1
) x
where x.rnk =1
ORDER by 5 DESC;
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/07bb3118-c650-4f66-9233-f8f0e74c8e2a)

To retain users we can offer coupons to the best(maximum transacting user) credit card users for a specific month. 

###	To determine the number of credit card available to each user.

````sql
select 
ch.id,cc.card, ch.name,count(ch.id) OVER(partition by ch.id) count_of_creditcards
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card 
JOIN merchant m JOIN merchant_category mc ON m.id_merchant_category = mc.id  
ON t.id_merchant = m.id 
group by 1,2,3
order by 1,4;
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/d7bc8190-d8c1-4c96-bc9f-bbc309d14811)


### To determine the users which have less credit card usage in a quarter with count of transactions less than 5.

````sql
WITH query1 as(
select 
ch.name,monthname(t.date) Month,count(ch.name) OVER(partition by ch.id,MONTH(t.date)) usage_per_month,
CONCAT('Q',QUARTER(t.date)) Quarter_
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card 
JOIN merchant m JOIN merchant_category mc ON m.id_merchant_category = mc.id  
ON t.id_merchant = m.id )

select * from query1 
where usage_per_month <5
group by 1,2,3,4
order by 4;
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/d100bca3-3c2d-4d45-aae9-1215c532ca63)


### To determine the users which have used their credit cards only once in any month of the year.

````sql
WITH query1 as (
select 
MONTHNAME(t.date) Months,ch.id,ch.name, 
count(t.date) OVER(partition by ch.id,MONTH(t.date)) monthly_transaction_count
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card )

select Months,id,name,monthly_transaction_count from (
select *,ROW_NUMBER() OVER(partition by name order by Months) rn from query1
where monthly_transaction_count<2) tb;
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/c2a5a0af-037f-4257-a61a-45fc35eb7d7f)


### To determine monthly transactions counts in a year.

````sql
select 
MONTHNAME(t.date) Months, count(t.id) 
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card 
group by 1;
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/1e660f95-9034-4a4e-83ce-02afeee4fbdf)


### To determine percentage increase or decrease in transactions every month of the year.

````sql
WITH query1 as(
select 
MONTHNAME(t.date) Month,count(t.date) count
from transaction t 
JOIN  credit_card cc JOIN card_holder ch ON cc.id_card_holder = ch.id 
ON t.card = cc.card 
group by 1)

select month, count,LAG(Percentage_Change,1) OVER() Per_change from (
select month,count,CONCAT(ROUND(((LEAD(count,1) OVER() - count)/count)*100,2),' %') Percentage_Change from query1) tb;
````

**Results:**

![image](https://github.com/Sarvagyam/CreditCardAnalysis/assets/36585456/401d4876-4433-4401-bcee-eb1350028949)

This analysis of percentage of user transactions on MoM basis let us know about the months where the credit card usage increased or decreased compared to their previous months. Allows a product manager to understand the reasons of increase or decrease of the usage in a particular month.

