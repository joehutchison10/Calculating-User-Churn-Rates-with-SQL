# Calculating-User-Churn-Rates-with-SQL

Churn Rate measures the percentage of customers who have canceled their subscription of a company's product or service. Churn rate is a logical way to track the performance of a business which relies on profit from subscriptions.       

In this example, Churn Rate was calculated each month. Mathematically this was computed as the number of user cancelations in a month, divided by the number of active subscribers, multiplied by 100. Cancelations were defined as a subscription termination occurring between the starting and ending date of a given month. Moreover, the number of active users were counted up to the start date of a given month. 



I will now break down this query to explain my approach to this problem. 




## To see the column names, and then to identify which months Churn Rate can be calculated for. 

SELECT *
 FROM subscriptions
 LIMIT 100; 

 SELECT MIN(subscription_start), MAX(subscription_start)
 FROM subscriptions
 LIMIT 100; 



## I have identified that we have complete months for January, February, and March. Thus we can create a temporary table for them, and CROSS JOIN (combine) it with our orginal subscription table. This will create a another temporary table, where each row of our subscription table will be combined with a start and end date for each month! 


WITH months AS 
 (SELECT 
 "2017-01-01" as first_day,
 "2017-01-31" as last_day
 UNION
 SELECT 
 "2017-02-01" as first_day,
 "2017-02-28" as last_day
 UNION
 SELECT
 "2017-03-01" as first_day,
 "2017-03-31" as last_day)

cross_join AS
(SELECT *
FROM subscriptions
CROSS JOIN months),


## Now we can compare the start and end date for each month, with the specific subscription start and end for each ID (customer) in our original subscription table. When the subscription start occured before the start of the month, we can assign a 1. When the subscription start occured after the start of the month, we can assign a 0. Similarly, if the subscription end occured between the starting and ending day of a month, we can assign a 1 and if not, 0.  


status AS
(SELECT id, first_day as month,
CASE
  WHEN (subscription_start < first_day)
    AND ((subscription_end > first_day) OR (subscription_end IS NULL))
    AND (segment = 87)
  THEN 1
  ELSE 0
END as is_active_87,
CASE 
  WHEN (subscription_start < first_day)
    AND ((subscription_end > first_day) OR (subscription_end IS NULL))
    AND (segment = 30)
  THEN 1
  ELSE 0
END as is_active_30,
CASE 
  WHEN subscription_end BETWEEN first_day AND last_day AND segment = 87 THEN 1
  ELSE 0
END as is_canceled_87,
CASE 
  WHEN subscription_end BETWEEN first_day AND last_day AND segment = 30 THEN 1
  ELSE 0
END as is_canceled_30
FROM cross_join),



## Now we have a binary status for customers who are subscribed and unsubscribed, during January, February and March. Thus we can tally unsubscribed and subscribed customers for each month using the "GROUP BY" month command, and then do the maths!     



status_aggregate AS
(SELECT
  month,
  SUM(is_active_87) as sum_active_87,
  SUM(is_active_30) as sum_active_30,
  SUM(is_canceled_87) as sum_canceled_87,
  SUM(is_canceled_30) as sum_canceled_30
FROM status
GROUP BY month)
SELECT month, 1.0 * sum_canceled_87 / sum_active_87 as churn_rate_87, 1.0 * sum_canceled_30 / sum_active_30 as churn_rate_30
FROM status_aggregate; 





