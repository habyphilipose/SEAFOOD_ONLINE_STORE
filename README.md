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
SELECT COUNT(DISTINCT user_id) AS Total_users <br>
FROM `SEAFOOD_ONLINE_STORE.users`
```

![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/4f7b7982-cd32-4cbb-ac43-265ca63f1afd)

### 2. How many cookies does each user have on average?

```sql
WITH cte AS (SELECT user_id,COUNT(cookie_id) AS cookie_id_count <br>
             FROM `SEAFOOD_ONLINE_STORE.users` <br>
             GROUP BY user_id) <br> <br>

SELECT ROUND(AVG(cookie_id_count),2) AS avg_cookie_per_user <br>
FROM cte <be>
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/0bcbeda6-870f-459c-926c-cce6efb31dc7)


### 3. What is the unique number of visits by all users per month?

```sql
SELECT EXTRACT(month FROM event_time) AS month, <br>
       COUNT(DISTINCT visit_id) AS unique_visit_count <br>
FROM `SEAFOOD_ONLINE_STORE.events' <br>
GROUP BY EXTRACT(month FROM event_time); <br>
```
![image](https://github.com/habyphilipose/SEAFOOD_ONLINE_STORE/assets/31076902/fdfe2a3c-4c1f-40d7-b522-8a454d0d8309)






