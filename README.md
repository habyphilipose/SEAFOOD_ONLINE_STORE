# SEAFOOD_ONLINE_STORE
SQL PROJECT 

In this case study we are required to analyse the dataset of a seafood online store and come up with creative solutions to calculate funnel fallout rates. 
# Entity Relationship Diagram
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/26d528e6-a199-4feb-833f-c2bf70631661)

### Table: users

![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/9a6d12df-cd11-4985-9c9b-ec892ec30a89)


### Table: events

![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/acbdc61f-4c2b-4789-9dc0-235e7cc6e164)


### Table: event_identifier

![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/2a502a91-0b49-45da-a901-646a7f4119ce)


### Table: page_hierarchy

![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/b819c492-b007-4cfe-865e-2993b208b3e4)


### Table: campaign_identifier

![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/25d6fad3-5817-46b9-ac71-27ead972c93f)

## A. DIGITAL ANALYSIS
### 1. How many users are there?

```sql
SELECT COUNT(DISTINCT user_id) AS Total_users 
FROM `SEAFOOD_ONLINE_STORE.users`
```

![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/4f7b7982-cd32-4cbb-ac43-265ca63f1afd)

### 2. How many cookies does each user have on average?

```sql
WITH cte AS (SELECT user_id,COUNT(cookie_id) AS cookie_id_count 
             FROM `SEAFOOD_ONLINE_STORE.users` 
             GROUP BY user_id) 

SELECT ROUND(AVG(cookie_id_count),2) AS avg_cookie_per_user 
FROM cte 
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/0bcbeda6-870f-459c-926c-cce6efb31dc7)


### 3. What is the unique number of visits by all users per month?

```sql
SELECT EXTRACT(month FROM event_time) AS month, 
       COUNT(DISTINCT visit_id) AS unique_visit_count 
FROM `SEAFOOD_ONLINE_STORE.events' 
GROUP BY EXTRACT(month FROM event_time); 
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/fdfe2a3c-4c1f-40d7-b522-8a454d0d8309)


### 4.	What is the number of events for each event type?
```sql
SELECT event_type, COUNT(*) AS event_count
FROM `SEAFOOD_ONLINE_STORE.events'
GROUP BY event_type
ORDER BY event_type;
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/89f4e700-c773-4a83-bc27-f2ff833d86a8)

### 5.	What is the percentage of visits which have a purchase event?

```sql
SELECT ROUND(100 * COUNT(DISTINCT e.visit_id)/
    (SELECT COUNT(DISTINCT visit_id) FROM 'SEAFOOD_ONLINE_STORE.events'),2) AS percentage_purchase
FROM 'SEAFOOD_ONLINE_STORE.events' AS e
JOIN 'SEAFOOD_ONLINE_STORE.event_identifier' AS ei
  ON e.event_type = ei.event_type
WHERE ei.event_name = 'Purchase';
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/a5e6772a-802b-453d-b5c7-d8d4769c5e28)

### 6.	 What is the percentage of visits which view the checkout page but do not have a purchase event?

```sql
WITH cte AS (
SELECT visit_id,
  MAX(CASE WHEN event_type = 1 AND page_id = 12 THEN 1 ELSE 0 END) AS checkout,
  MAX(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase
FROM 'SEAFOOD_ONLINE_STORE.events’
GROUP BYvisit_id)

SELECT ROUND(((1 - (SUM(purchase)/SUM(checkout)))*100.0),1) AS percentage_checkout_view_with_no_purchase
FROM cte
```

![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/5efaac4a-2609-4886-86a3-cb2855bd11e6)


### 7.	What are the top 3 pages by number of views ?

```sql
WITH cte AS (
  SELECT 
    p.product_id,
    p.page_name,
    COUNT(e.visit_id) AS Num_of_visits
  FROM 'SEAFOOD_ONLINE_STORE.events' AS e
  JOIN 'SEAFOOD_ONLINE_STORE.page_hierarchy' AS  p 
  ON p.page_id = e.page_id
  GROUP BY 2
),
cte2 AS (
  SELECT page_name,
         Num_of_visits,
         RANK() OVER(ORDER BY Num_of_visits DESC) AS rnk
  FROM cte
)
SELECT page_name,
       Num_of_visits
FROM cte2
WHERE rnk <= 3;
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/2d70160d-637a-4cf1-905e-c0120bb68ad7)

### 8.	What is the number of views and cart add's for each product category?
```sql
SELECT ph.product_category, 
  SUM(CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END) AS page_views,
  SUM(CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds
FROM 'SEAFOOD_ONLINE_STORE.events' AS e
JOIN 'SEAFOOD_ONLINE_STORE.page_hierarchy' AS ph
ON e.page_id = ph.page_id
WHERE ph.product_category IS NOT NULL
GROUP BY ph.product_category
ORDER BY page_views DESC;
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/95f37c95-4e5e-4843-996e-8ad2e4753974)

## Product Funnel Analysis:

### A.	Using a single SQL query — create a new output table which has the following details:
•	How many times was each product viewed?
•	How many times was each product added to cart?
•	How many times was each product added to a cart but not purchased (abandoned)?
•	How many times was each product purchased?

```sql
WITH cte AS (
  SELECT 
    e.visit_id,
    e.cookie_id,
    e.event_type,
    p.page_id,
    p.page_name,
    p.product_category,
    p.product_id
  FROM 'SEAFOOD_ONLINE_STORE.events' AS e
  JOIN 'SEAFOOD_ONLINE_STORE.page_hierarchy' AS p 
  ON p.page_id = e.page_id
), 
cte2 AS (
  SELECT 
    page_name,
    CASE WHEN event_type = 2 THEN visit_id END AS cart_id,
    CASE WHEN event_type = 1 THEN visit_id END AS viewed 
  FROM cte
  WHERE product_id IS NOT NULL
),
cte3 AS (
  SELECT 
    visit_id AS purchase_id 
  FROM 'SEAFOOD_ONLINE_STORE.events'
  WHERE event_type = 3
)
SELECT 
  a.page_name,
  COUNT(a.viewed) AS Page_views,
  COUNT(a.cart_id) AS Added_to_cart,
  COUNT(a.cart_id) - COUNT(b.purchase_id) AS Abandoned,
  COUNT(b.purchase_id) AS purchase
FROM cte2 AS a
LEFT JOIN cte3 AS b 
ON b.purchase_id = a.cart_id
GROUP BY 1 
ORDER BY COUNT(b.purchase_id) DESC;
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/962e8551-57a1-4044-af9d-5cefa586deaf)

### A-1) Which product had the most views, cart adds and purchases?
```sql
WITH product_info as 
(WITH cte AS (
  SELECT 
    e.visit_id,
    e.cookie_id,
    e.event_type,
    p.page_id,
    p.page_name,
    p.product_category,
    p.product_id
  FROM 'SEAFOOD_ONLINE_STORE.events' AS e
  JOIN 'SEAFOOD_ONLINE_STORE.page_hierarchy' AS p 
  ON p.page_id = e.page_id
), 
cte2 AS (
  SELECT 
    page_name,
    CASE WHEN event_type = 2 THEN visit_id END AS cart_id,
    CASE WHEN event_type = 1 THEN visit_id END AS viewed 
  FROM cte
  WHERE product_id IS NOT NULL
),
cte3 AS (
  SELECT 
    visit_id AS purchase_id 
  FROM 'SEAFOOD_ONLINE_STORE.events'
  WHERE event_type = 3
)
SELECT 
  a.page_name,
  COUNT(a.viewed) AS Page_views,
  COUNT(a.cart_id) AS Added_to_cart,
  COUNT(a.cart_id) - COUNT(b.purchase_id) AS Abandoned,
  COUNT(b.purchase_id) AS purchase
FROM cte2 AS a
LEFT JOIN cte3 AS b 
ON b.purchase_id = a.cart_id
GROUP BY 1 
ORDER BY COUNT(b.purchase_id) DESC;)
--MOST VIEWS
SELECT page_name AS Most_viewed  
FROM product_info
WHERE Page_views = (SELECT MAX(Page_views) FROM product_info);

--MOST CART ADDS
SELECT page_name AS Most_cart_adds  
FROM product_info
WHERE Added_to_cart =(SELECT MAX(Added_to_cart) FROM product_info);

--MOST PURCHASES
SELECT page_name AS Most_purchased  FROM product_info
WHERE purchase =(SELECT MAX(purchase) FROM product_info);

```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/e911b607-3a56-4ff8-869f-2591ed954aec)
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/97071ffb-68d2-4772-b542-d71495bbd0c4)
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/d092e344-7064-413b-98a6-8ab899a94b08)

###  A-2) Which product was most likely to be abandoned?
```sql
SELECT 
page_name AS most_likely_abandoned 
FROM product_info
WHERE Abandoned =(SELECT  MAX(Abandoned) FROM product_info);
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/647cb023-e2d3-4578-b635-e8bda80ff94e)

### A-3) Which product has highest purchase-to-view percentage?

```sql
SELECT
page_name, CONCAT(ROUND( 100.0*(purchase/Page_views),2), '%') AS View_purchase_percentage
FROM product_info
ORDER BY 2 DESC 
LIMIT 1
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/f8bf7c05-d31d-497c-b239-fab3c653b55b)

### A-4) What is the AVERAGE CONVERSION RATE  from view to cart add?

```sql
SELECT
ROUND(AVG(100.0*(Added_to_cart/Page_views)),2) as Avg_conversion_rate 
FROM product_info
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/6a325ce9-5e4f-4aee-b048-1bcbb3120c53)

### A-5) What is the AVERAGE CONVERSION RATE  from cart add to purchase?
```sql
SELECT 
ROUND(AVG(100.0*(purchase/Added_to_cart)),2) as Avg_conversion_rate 
FROM product_info
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/5bcac18f-87d4-4323-9da6-03a687fe7e77)

## Campaign's Analysis
### Generate a table that has 1 single row for every unique visit_id record and has the following columns:
•	user_id<be>
•	visit_id<be>
•	visit_start_time: the earliest event_time for each visit <be>
•	page_views: count of page views for each visit <be>
•	cart_adds: count of product cart add events for each visit <be>
•	purchase: 1/0 flag if a purchase event exists for each visit <be>
•	campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date <be>
•	impression: count of ad impressions for each visit <be>
•	click: count of ad clicks for each visit <be>
•	(Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number) <be>
```sql
SELECT 
    user_id,
    visit_id,
    MIN(e.event_time) AS visit_start_time,
    COUNT(CASE WHEN event_type = 1 THEN visit_id END) AS Page_views,
    COUNT(CASE WHEN event_type = 2 THEN visit_id END) AS Cart_adds,
    COUNT(CASE WHEN event_type = 3 THEN visit_id END) AS Purchase,
    CASE WHEN u.start_date BETWEEN ci.start_date AND ci.end_date THEN ci.campaign_name END AS Campaign_name,
    COUNT(CASE WHEN e.event_type = 4 THEN e.visit_id END) AS Impression,
    COUNT(CASE WHEN e.event_type = 5 THEN e.visit_id END) AS Click,
    STRING_AGG(CASE WHEN event_type = 2 AND product_id IS NOT NULL THEN Page_name ELSE NULL END,','
        ORDER BY e.sequence_number) AS cart_products
FROM
    'SEAFOOD_ONLINE_STORE.users' AS u
        JOIN
    'SEAFOOD_ONLINE_STORE.events' AS e 
    ON u.cookie_id = e.cookie_id
        JOIN
    'SEAFOOD_ONLINE_STORE.campaign_identifier' AS ci 
    ON e.event_time BETWEEN ci.start_date AND ci.end_date
        JOIN
    'SEAFOOD_ONLINE_STORE.page_hierarchy' AS ph 
    ON ph.page_id = e.page_id
GROUP BY e.visit_id
ORDER BY user_id
```

![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/eb0103ae-fc00-47e2-8c22-93284df9f09a)

