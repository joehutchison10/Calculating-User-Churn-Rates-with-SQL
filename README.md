# Calculating-User-Churn-Rates-with-SQL

Churn Rate measures the percentage of customers who have canceled their subscription of a company's product or service. Churn rate is a logical way to track the performance of a business which relies on profit from subscriptions.       

In this project, Churn Rate was calculated for January, Februrary and March for two hypothetical segments of customers, 30 and 87. In the real world, these segments could represent customer populations with different demographic characteristics. Mathematically, Churn Rate was computed as the number of user cancelations from the start to end of a given month, divided by the number of active subscribers up to the start date of that month, multiplied by 100. 

The database used was self-created, which is named subscriptions. It has the columns: id, subscription_start, subscription_end, and segment.  

I will now break down this query to explain my approach to this problem. 

### To see the column names of the dataset, and then to identify which months Churn Rate can be calculated for. 

SELECT *
 FROM subscriptions
 LIMIT 100; 

 SELECT MIN(subscription_start), MAX(subscription_start)
 FROM subscriptions
 LIMIT 100; 


### We have complete months for January, February, and March. Thus we can create a temporary table for them, and CROSS JOIN (combine) it with our orginal subscription table. This will create a new temporary table called cross_join, where each row of our subscription table will be combined with a start and end date for each month! 


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


### This code assesses our cross_join temporary table, and sorts customers according to their segments and subscription start/end dates. We have two different CASE statements, which are designed to uniquely sort customers into segment 87 or 30, and assess if the customers subscription start date occurs before the start date of a given month: if it does we assign a 1, if it doesn't we assign a 0. The output of these CASE statements are stored as either is_active_87 or is_active_30 column in our new status table. Similarly, we have two different CASE statements which assess whether a customer's subscription end occured between the starting and ending day of a month, where a 1 is assigned, or a 0 if not. Again, these CASE statements sort customers into their respective segments, with their output stored in the is_canceled_87 or is_canceled_30 columns of this new status table. 


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



### Now we have a binary status for each customer, who is sorted by their segment, representing if they are subscribed or unsubscribed, during January, February and March. Using SUM, we can now tally unsubscribed and subscribed customers for each month using the "GROUP BY" month command, and plug the output into our Churn Rate equation!. 



status_aggregate AS
(SELECT
  month,
  SUM(is_active_87) as sum_active_87,by 
  SUM(is_active_30) as sum_active_30,
  SUM(is_canceled_87) as sum_canceled_87,
  SUM(is_canceled_30) as sum_canceled_30
FROM status
GROUP BY month)
SELECT month, 1.0 * sum_canceled_87 / sum_active_87 as churn_rate_87, 1.0 * sum_canceled_30 / sum_active_30 as churn_rate_30
FROM status_aggregate; 





