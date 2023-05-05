# Revou Labs : Individual SQL EcommerceU

## Milestone 1

### 1. Write a query to find a monthly new user based on their registration date in 2022 for users who lived in south jakarta. Which of the following statements is true regarding the result of the query?
- There are only 9 users from south jakarta who register to our platform in october 2022 **[TRUE]**
- EcommerceU has a really low number of new users from south jakarta, where each month the number of new users is not more than 30 **[TRUE]**
- South jakarta become a promising locating because the number of new users is increasing each month **[FALSE]**

``` sql
SELECT 
      extract(month from user.register_date)month_1,
      extract(year from user.register_date)year_1,
      count(distinct user.user_id) total_unique_users
FROM `da-labs-b4-ecommerce.b4_ecommerce_dataset.users` user
LEFT JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.locations` locations
ON user.locations_id=locations.locations_id
WHERE
      user.register_date between '2022-01-01' and '2022-12-31' and locations.location='Jakarta Selatan'
GROUP BY 
      month_1,year_1,location
ORDER BY
      year_1,month_1,location;
```

 **Table results:**
 
 ![image](https://user-images.githubusercontent.com/93022524/236400507-bc5cadb2-337a-44e7-a982-eda8613addeb.png)

 
### 2. Write a query to find monthly convertion rate visits to transactions and its growth in 2021. Which of the following statements is true regarding the result of the query?

- Highest convertion rate in 2021 is 12.31% **[TRUE]**
- There is no convertion rate of more than 15% in 2021 **[TRUE]**
- From june to july 2021, ecommerceU hit the lowest growth of convertion rate with -18% growth **[TRUE]**

``` sql
WITH CVR AS (
  SELECT 
    DATE_TRUNC(s.visits_timestamps, MONTH) month, 
    COUNT(DISTINCT s.sessions_id) visits, 
    COUNT(DISTINCT t.transactions_id) transactions
  FROM 
   `da-labs-b4-ecommerce.b4_ecommerce_dataset.sessions` s 
    LEFT JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.transactions` t 
    ON s.sessions_id = t.sessions_id
  WHERE FORMAT_DATE('%Y', DATE(s.visits_timestamps)) = '2021'
  GROUP BY month
), growth_query AS (
  SELECT 
    CVR.month, 
    CVR.visits, 
    CVR.transactions, 
    CVR.transactions / CVR.visits conversion_rate, 
    LAG((CVR.transactions/CVR.visits)) OVER (ORDER BY CVR.month) previous_conversion_rate
  FROM 
    CVR
)
SELECT 
  month, 
  round(conversion_rate*100,2) conversion_rate, 
  (conversion_rate - previous_conversion_rate) / previous_conversion_rate * 100 growth
FROM 
  growth_query
ORDER BY 
  month;
```

 **Table results:**

 ![image](https://user-images.githubusercontent.com/93022524/236400926-93a4d268-7005-495b-8862-9b04ea9b2640.png)

### 3. Write a query to find each product AOV and average product price on the complete transaction. sort from the highest AOV. Which of the following statements is true regarding the result of the query?
- The highest AOV and product price is electronic C, and it has the same amount **[TRUE]**
- Product hobbies C AOV (1,205,711) is higher than its price (750,000). It means, on average, the order quantity is more than 1 **[TRUE]**
- All the produt has higher AOV amount than price amount **[FALSE]**

``` sql
SELECT
      product.product_name,
      count(distinct trx_item.product_qty)order_quantity,
      trx_item.product_price,
      round(avg(trx_item.product_amount),2) AOV   
FROM 
      `da-labs-b4-ecommerce.b4_ecommerce_dataset.transactions` trx
JOIN
      `da-labs-b4-ecommerce.b4_ecommerce_dataset.transaction_items` trx_item ON trx.transactions_id=trx_item.transaction_items_id
JOIN
      `da-labs-b4-ecommerce.b4_ecommerce_dataset.products` product ON trx_item.product_id=product.product_id
WHERE
      trx.status='completed'
GROUP BY
      1,3
ORDER BY
      AOV DESC
```

 **Table results:**
 
![image](https://user-images.githubusercontent.com/93022524/236400705-20f1fca9-0db6-4ae5-b929-6afe37dc2a66.png)
 
## Milestone 2
 
### 1. Write a query to find number of user, and total visit (traffic) for each age group. Group definition are <=25 (Young Age), 26-40 (Mid Age) and >=41 (Old Age). Which of the following statements is true regarding the result of the query?
- Group of Young Age and Mid Age is dominating our user **[TRUE]**
- Young Age has the most visits, in total there are 70789 visits from them **[TRUE]**
- Mid Age has the highest number of users **[FALSE]**

``` sql
WITH user_age_group AS (
  SELECT 
    user_id,
    CASE 
      WHEN age <= 25 THEN 'Young Age'
      WHEN age >= 41 THEN 'Old Age'
      ELSE 'Mid Age'
    END AS age_group
  FROM 
    `da-labs-b4-ecommerce.b4_ecommerce_dataset.users`
),
traffics AS (
  SELECT 
    user_age_group.age_group,
    COUNT(DISTINCT user_age_group.user_id) AS number_of_user,
    COUNT(DISTINCT sessions.sessions_id) AS total_visits
  FROM 
    user_age_group
    JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.sessions` sessions
      ON user_age_group.user_id = sessions.user_id
  GROUP BY 
    user_age_group.age_group
)
SELECT 
  age_group,
  number_of_user,
  total_visits
FROM 
  traffics;
```

 **Table results:**
 
 ![image](https://user-images.githubusercontent.com/93022524/236402026-088b48a0-5432-4538-a93f-db0f22010a25.png)

### 2. Write a query to find the average transaction frequency per customer for each product category. Which of the following statements is true regarding the result of the query?
Hints: group by product category

- Groceries have the highest average transaction frequency. Each customer has around 2.32 transactions containing groceries product **[TRUE]**
- On average, transaction frequency per customer for clothing product is 3.5 **[FALSE]**
- On average, transaction frequency per customer for Electronic products is 2.5 **[FALSE]**

``` sql
SELECT 
  product.product_category,
  COUNT(DISTINCT trx.sessions_id) / COUNT(DISTINCT sessions.user_id) AS avg_trx_freq
FROM 
      `da-labs-b4-ecommerce.b4_ecommerce_dataset.transactions` trx
  JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.transaction_items` trx_i ON trx.transactions_id = trx_i.transactions_id
  JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.products` product ON trx_i.product_id = product.product_id
  JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.sessions` sessions ON trx.sessions_id = sessions.sessions_id
GROUP BY 
  product.product_category;
```

 **Table results:**
 
 ![image](https://user-images.githubusercontent.com/93022524/236402601-245424d3-58b6-42ae-b15a-0afbd96acba3.png)

### 3. Write q query to find the ABS (average basket size) for each product category with complete transactions. Which of the following statements is true regarding the result of the query?

- The highest ABS is Stationary product category **[TRUE]**
- Product category Electronics has 1.19 ABS value **[TRUE]**
- All the product has ABS value around 3 **[FALSE]**

``` sql
SELECT
  product.product_category,
  AVG(trx_i.product_qty) AS ABS
FROM
  `da-labs-b4-ecommerce.b4_ecommerce_dataset.transactions` trx
  JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.transaction_items` trx_i ON trx.transactions_id = trx_i.transactions_id
  JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.products` product ON trx_i.product_id = product.product_id
WHERE 
  trx.status = 'completed'
GROUP BY
  product.product_category
ORDER BY ABS desc
```

 **Table results:**
 
 ![image](https://user-images.githubusercontent.com/93022524/236402853-7a16feb9-dbbe-4ccc-8fae-8152109dfd5f.png)


 ## Milestone 3
 
 ### 1. Write a query to find the convertion rate from product view to complete in March 2020. Which of the following statements is true regarding the result of the query?
- The convertion rate is 5.46% **[TRUE]**
- The convertion rate is 7.83% **[FALSE]**
- The convertion rate is 8.60% **[FALSE]**

``` sql
WITH CVR AS (
  SELECT 
    DATE_TRUNC(e.timestamp, MONTH) month, 
    COUNT(DISTINCT e.sessions_id) visits, 
    COUNT(DISTINCT CASE WHEN event = 'complete' THEN t.transactions_id ELSE NULL END) transactions
  FROM 
   `da-labs-b4-ecommerce.b4_ecommerce_dataset.events` e 
    LEFT JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.transactions` t 
    ON e.sessions_id = t.sessions_id
  WHERE FORMAT_DATE('%Y-%m', DATE(e.timestamp)) = '2020-03'
  GROUP BY month
), growth_query AS (
  SELECT 
    CVR.month, 
    CVR.visits, 
    CVR.transactions, 
    CVR.transactions / CVR.visits conversion_rate
  FROM 
    CVR
)
SELECT 
  month, 
  round(conversion_rate*100,2) conversion_rate
FROM 
  growth_query
ORDER BY 
  month;
```

 **Table results:**
 
 ![image](https://user-images.githubusercontent.com/93022524/236406552-b2aac98e-d759-4a6d-8e78-a7a3da15e917.png)

 ### 2. Write a query to find the convertion rate from product view to complete for each traffic source. Sort from the hisghest convertion rate. Which of the following statements is true regarding the result of the query?
- Direct traffic sources have the hisghest convertion rate ith 13.88% **[TRUE]**
- Newsletter and Instagram are included as the top 3 traffic sources with the highest convertion rate **[TRUE]**
- Google has the lowest convertion rate **[FALSE]**

``` sql
WITH CVR AS (
  SELECT
    s.traffic_source, 
    COUNT(DISTINCT e.sessions_id) visits, 
    COUNT(DISTINCT CASE WHEN event = 'complete' THEN t.transactions_id ELSE NULL END) transactions
  FROM
    `da-labs-b4-ecommerce.b4_ecommerce_dataset.sessions` s 
  LEFT JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.events` e ON s.sessions_id = e.sessions_id
  LEFT JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.transactions` t ON s.sessions_id = t.sessions_id
  GROUP BY traffic_source
), CVR2 AS (
  SELECT 
    CVR.traffic_source,
    CVR.visits, 
    CVR.transactions, 
    CVR.transactions / CVR.visits conversion_rate
  FROM 
    CVR
)
SELECT 
  traffic_source, 
  round(conversion_rate*100,2) conversion_rate
FROM 
  CVR2
ORDER BY 
  1;
```

 **Table results:**
 
![image](https://user-images.githubusercontent.com/93022524/236406689-7706b122-fedc-48bd-b5d0-4f296c64635d.png)

 ### 3. Write a query to find the average needed (in seconds) for customer after view the cart to finish choosing their address. Which of the following statements is true regarding the result of the query?

- The average duration is 154.5 seconds **[TRUE]**
- The average duration is 260.1 seconds **[FALSE]**
- The average duration is 189.8 seconds **[FALSE]**

``` sql
SELECT ROUND(AVG(duration),2) AS avg_duration
FROM (
  SELECT 
  (DATETIME_DIFF((MAX(e.timestamp)),(MIN(e.timestamp)),second)) AS duration
    FROM `da-labs-b4-ecommerce.b4_ecommerce_dataset.events` e
    WHERE event IN ('viewcart', 'chooseaddress')
    GROUP BY e.sessions_id
    HAVING COUNT(DISTINCT event) = 2
  )
```

 **Table results:**
 
![image](https://user-images.githubusercontent.com/93022524/236406783-4141f645-560a-4a59-8076-724ce12a2c2f.png)


## Milestone 4

### 1. Write a query to find top 5 high-demand product based on complete transaction. Which of the following statements is true regarding the result of the query?
- Those 5 products are Mother&Care F, Health&Beauty C, Health&Beauty D, Health&Beauty E, and Clothing D **[TRUE]**
- Those 5 products are Mother&Care F, Health&Beauty C, Health&Beauty D, Clothing A, and Clothing D **[FALSE]**
- All those top 5 products are having demand (total transaction) above 1000 **[TRUE]**

``` sql
SELECT P.product_name,
      COUNT(DISTINCT T.transactions_id) AS Total_Transaction
FROM `da-labs-b4-ecommerce.b4_ecommerce_dataset.transactions` T
LEFT JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.transaction_items` TI ON T.transactions_id = TI.transactions_id
LEFT JOIN `da-labs-b4-ecommerce.b4_ecommerce_dataset.products` P ON TI.product_id = P.product_id
WHERE T.status = 'completed'
GROUP BY P.product_name
ORDER BY Total_Transaction DESC
LIMIT 5

```

 **Table results:**

![image](https://user-images.githubusercontent.com/93022524/236406006-11ec6ad7-5beb-434c-b3a6-d49febebef22.png)


### 2. Write a query to find the voucher usage rate in 2022 for all transactions status. Which of the following statements is true regarding the result of the query?
- The voucher usage rate is 68.37% **[TRUE]**
- The voucher usage rate is 80.45% **[FALSE]**
- The voucher usage rate is 47.88% **[FALSE]**

``` sql
WITH v1 AS 
(
  SELECT EXTRACT(year FROM transactions_timestamps) AS year,
         count(voucher_id) AS voucher_usg
  FROM `da-labs-b4-ecommerce.b4_ecommerce_dataset.transactions`
  WHERE voucher_id in (1,2,3) AND 
  transactions_timestamps between '2022-01-01' AND '2023-01-01'
  GROUP BY 1
),
v2 AS 
(
  SELECT EXTRACT(year FROM transactions_timestamps) AS year,
       count(transactions_id) AS total_trx
FROM `da-labs-b4-ecommerce.b4_ecommerce_dataset.transactions`
WHERE transactions_timestamps between '2022-01-01' AND '2023-01-01'
GROUP BY 1
)
SELECT v1.year,
       ROUND((v1.voucher_usg / v2.total_trx)*100,2) AS voucher_usage
FROM v1
JOIN v2
ON v1.year = v2.year

```

 **Table results:**
 
![image](https://user-images.githubusercontent.com/93022524/236406261-d26d0f5f-8b45-45db-8959-d47eb8631b3e.png)

### 3. Write a query to find average number of unique product per transaction for each month in 2022. Which of the following statements is true regarding the result of the query?
- The average unique product per transaction in Jan, Feb and March 2022 is around 3 products **[TRUE]**
- The average unique product per transaction in Jun, Jul and August 2022 is around 4 products **[FALSE]**
- There is no average unique product per transaction that is more than 6 each month **[TRUE]**

``` sql
SELECT 
    FORMAT_DATE('%Y-%m', transaction_date) AS month,
    AVG(unique_products) AS avg_unique_products_per_transaction
FROM (
    SELECT 
        t.transactions_id, 
        COUNT(DISTINCT ti.product_id) AS unique_products,
        DATE_TRUNC(t.transactions_timestamps, MONTH) AS transaction_date
    FROM 
        `da-labs-b4-ecommerce.b4_ecommerce_dataset.transactions` AS t
    JOIN
     `da-labs-b4-ecommerce.b4_ecommerce_dataset.transaction_items` AS ti
    ON
        t.transactions_id=ti.transactions_id
    WHERE 
        EXTRACT(YEAR FROM t.transactions_timestamps) = 2022
    GROUP BY 
        1, 3
)
GROUP BY 
    1
ORDER BY 
    1;

```

 **Table results:**

![image](https://user-images.githubusercontent.com/93022524/236405856-c8663c8b-0bfd-408e-9ae4-ce0db0962ceb.png)


## Milestone 5

