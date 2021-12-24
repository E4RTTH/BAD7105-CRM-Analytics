# Cross-selling
[![](https://img.shields.io/badge/-Python-blue)](#)

**Data Set** [Customer Preference Survey]()

## Method
---

### 1. Add Data set into BigQuery

![image](./supermarket_dataset.png)

### 2. Create MAU_CUSTOMER table 
[![](https://img.shields.io/badge/-SQL-orange)](#)



```
SELECT 
  PARSE_DATE('%Y%m%d',  CAST( SHOP_DATE AS STRING )) AS DATE,
  CUST_CODE,
  1 AS FLAG_MAU
FROM `crm-analytics-331409.Churn.Supermarket` SHOP_WEEKDAY
WHERE CUST_CODE IS NOT NULL
ORDER BY CUST_CODE, DATE
```
</br>

![image](./MAU_CUSTOMER.png)

### 3. Create CUSTOMER_FIRST_JOIN table
[![](https://img.shields.io/badge/-SQL-orange)](#)

```
SELECT
   MIN(DATE),
   CUST_CODE
FROM `crm-analytics-331409.Churn.MAU_CUSTOMER`
GROUP BY CUST_CODE
```
</br>

![image](./Customer_first_join.png)

### 4. Create Churn_dashboard table with store procedure
[![](https://img.shields.io/badge/-SQL-orange)](#)

```
BEGIN

  DECLARE first_date_this_month DATE;
  DECLARE last_date_this_month DATE;
  DECLARE first_date_last_month DATE;
  DECLARE last_date_last_month DATE;
  
  DECLARE x DATE;
  DECLARE y DATE;
  
  SET x = DATE("2006-04-01");
  SET y = DATE("2008-07-06");
  
  WHILE x <= y DO
  
   SET first_date_this_month = DATE_TRUNC(x, MONTH);
   SET last_date_this_month =  DATE_ADD(DATE_ADD(DATE_TRUNC(x, MONTH), INTERVAL 1 MONTH), INTERVAL -1 DAY);
   SET first_date_last_month = DATE_ADD(x, INTERVAL -1 MONTH);
   SET last_date_last_month = DATE_ADD(x, INTERVAL -1 DAY);
   
   CREATE OR REPLACE TEMP TABLE curr
   AS
   SELECT *
   FROM `crm-analytics-331409.Churn.MAU_CUSTOMER`
   WHERE DATE BETWEEN first_date_this_month AND last_date_this_month;
   
   CREATE OR REPLACE TEMP TABLE prev
   AS
   SELECT *
   FROM `crm-analytics-331409.Churn.MAU_CUSTOMER`
   WHERE DATE BETWEEN first_date_last_month AND last_date_last_month;
   
   CREATE OR REPLACE TEMP TABLE bf
   AS
   SELECT *
   FROM `crm-analytics-331409.Churn.MAU_CUSTOMER`
   WHERE DATE < first_date_last_month;
   
   CREATE OR REPLACE TEMP TABLE curr_mau
   AS
   SELECT
    CUST_CODE, 
    IF(SUM(FLAG_MAU)>=1,1,0) AS MAU 
   FROM curr 
   GROUP BY CUST_CODE;
   
   CREATE OR REPLACE TEMP TABLE prev_mau
   AS
   SELECT
    CUST_CODE, 
    IF(SUM(FLAG_MAU)>=1,1,0) AS MAU 
   FROM prev 
   GROUP BY CUST_CODE;
   
   CREATE OR REPLACE TEMP TABLE bf_mau
   AS
   SELECT 
    CUST_CODE, 
    IF(SUM(FLAG_MAU)>=1,1,0) AS MAU 
   FROM bf 
   GROUP BY CUST_CODE;
   
   CREATE OR REPLACE TEMP TABLE customer
   AS
   SELECT 
    *
   FROM `crm-analytics-331409.Churn.CUSTOMER_FIRST_JOIN`
   WHERE FIRST_DATE <= last_date_this_month;
   
   CREATE OR REPLACE TEMP TABLE final
   AS
   SELECT
    first_date_this_month AS MONTH,
    customer.CUST_CODE,
    IF(IFNULL(curr_mau.MAU,0) = 1 AND IFNULL(prev_mau.MAU,0) = 0 AND IFNULL(bf_mau.MAU,0) = 0, 1, 0) AS FLAG_NEW,
    IF(curr_mau.MAU >= 1 AND prev_mau.MAU >= 1, 1, 0) as FLAG_REPEAT,
    IF(curr_mau.MAU >= 1 AND (prev_mau.MAU is null OR prev_mau.MAU = 0) AND bf_mau.MAU >= 1, 1, 0) as FLAG_REACTIVE,
	  IF((curr_mau.MAU IS NULL OR curr_mau.MAU = 0) AND prev_mau.MAU >= 1 , 1, 0) AS FLAG_CHURN
   FROM customer
   LEFT JOIN curr_mau
   ON customer.CUST_CODE = curr_mau.CUST_CODE
   LEFT JOIN bf_mau
   ON customer.CUST_CODE = bf_mau.CUST_CODE
   LEFT JOIN prev_mau
   ON customer.CUST_CODE = prev_mau.CUST_CODE;
   
   INSERT INTO `crm-analytics-331409.Churn.Churn_dashboard`
   (MONTH, CUST_CODE, FLAG_NEW, FLAG_REPEAT, FLAG_REACTIVE, FLAG_CHURN)
   SELECT MONTH, CUST_CODE, FLAG_NEW, FLAG_REPEAT, FLAG_REACTIVE, FLAG_CHURN
   FROM final;
   
   SET x = DATE_ADD(x, INTERVAL 1 MONTH);
   
   END WHILE;
   
END
```
##### Call store procedure to create table

```
CALL `crm-analytics-331409.Churn.churn`();
```
</br>

![image](./Churn_table.png)


## Result
---
</br>

![image](./Churn_Dashboard.png)