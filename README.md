# Recreating tables and solving problem sets on PostgreSQL/pgAdmin
[![](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/download/windows/)

Problem sets sourced from [DataLemur](https://datalemur.com/questions?category=SQL)

## Uber - User's Third Transaction
**"Write a query to obtain the third transaction of every user"**
```
CREATE TABLE transactions(
user_id	SMALLINT NOT NULL,
spend NUMERIC NOT NULL,
transaction_date TIMESTAMP NOT NULL);


INSERT INTO transactions VALUES
(111, 100.50, '2022-01-08 12:30:00'),
(111, 55.00, '2022-01-10 2:20:00'),
(121, 36.00, '2022-01-18 11:00:00'),
(145, 24.99, '2022-01-26 12:15:00'),
(111, 89.60, '2022-02-05 5:50:00'),
(145, 45.30, '2022-02-28 4:32:00'),
(121, 22.20, '2022-04-01 6:10:00'),
(121, 67.90, '2022-04-03 8:50:00'),
(263, 156.00, '2022-04-11 11:11:00'),
(230, 78.30, '2022-06-14 1:30:00'),
(263, 68.12, '2022-07-11 3:30:00'),
(263, 100.00, '2022-07-12 9:42:00');


WITH rankings AS 
(SELECT *, RANK() OVER(PARTITION BY user_id ORDER BY transaction_date) AS ranks
FROM transactions)
SELECT user_id, spend, transaction_date
FROM rankings
WHERE ranks = 3;

```

## Snapchat - Sending vs. Opening Snaps
**"Write a query to obtain a breakdown of the time spent sending vs. opening snaps as a percentage of total time spent on these activities grouped by age group"**
```
CREATE TABLE activities(
activity_id SMALLINT PRIMARY KEY,
user_id	SMALLINT NOT NULL,
activity_type TEXT
	CONSTRAINT activity_type_c CHECK (activity_type IN ('send', 'open', 'chat')),
time_spent NUMERIC NOT NULL,
activity_date TIMESTAMP NOT NULL);

INSERT INTO activities VALUES
(7274, 123, 'open', 4.50, '2022-06-22 10:00:00'),
(2425, 123, 'send', 3.50, '2022-06-22 12:50:00'),
(1413, 456, 'send', 5.67, '2022-06-23 2:45:00'),
(2536, 456, 'open', 3.00, '2022-06-25 3:20:00'),
(8564, 456, 'send', 8.24, '2022-06-26 7:19:00'),
(5235, 789, 'send', 6.24, '2022-06-28 4:25:00'),
(4251, 123, 'open', 1.25, '2022-07-01 5:55:00'),
(1414, 789, 'chat', 11.00, '2022-06-25 7:21:00'),
(1314, 123, 'chat', 3.15, '2022-06-26 12:00:00'),
(1435, 789, 'open', 5.25, '2022-07-02 1:11:00');

CREATE TABLE age_breakdown(
user_id SMALLINT PRIMARY KEY,
age_bucket TEXT NOT NULL
	CONSTRAINT age_bucket_c CHECK (age_bucket IN('21-25','26-30','31-35')));
	
INSERT INTO age_breakdown VALUES
(123, '31-35'),
(456, '26-30'),
(789, '21-25');


SELECT age_bucket,
  ROUND(100*sum(time_spent) FILTER (WHERE activity_type = 'send')/SUM(time_spent),2) AS time_sending,
  ROUND(100*sum(time_spent) FILTER (WHERE activity_type = 'open')/SUM(time_spent),2) AS time_opening
FROM activities AS a
JOIN age_breakdown AS ab
ON a.user_id = ab.user_id
WHERE activity_type IN ('send', 'open')
GROUP BY age_bucket;
```
## New York Times - Laptop vs. Mobile Viewership
**"Write a query that calculates the total viewership for laptops and mobile devices where mobile is defined as the sum of tablet and phone viewership"**
```
CREATE TABLE viewership(
user_id SMALLINT NOT NULL,
device_type TEXT NOT NULL,
	CONSTRAINT device_typec CHECK (device_type IN ('laptop', 'tablet', 'phone')),
view_time TIMESTAMP NOT NULL);

INSERT INTO viewership VALUES
(123, 'tablet', '2022-01-02 00:00:00'),
(125, 'laptop', '2022-01-07 00:00:00'),
(128, 'laptop', '2022-02-09 00:00:00'),
(129, 'phone', '2022-02-09 00:00:00'),
(145, 'tablet', '2022-02-24 00:00:00');

SELECT 
COUNT(device_type) FILTER (WHERE device_type = 'laptop') AS laptop_views,
COUNT(device_type) FILTER (WHERE device_type IN ('phone', 'tablet')) AS mobile_views
FROM viewership;

-- or

WITH lv AS
(SELECT COUNT(device_type) AS laptop_views
FROM viewership
WHERE device_type = 'laptop'),
mv AS (SELECT COUNT(device_type) AS mobile_views
FROM viewership
WHERE device_type IN ('tablet', 'phone'))
SELECT *
FROM lv, mv;

-- or

SELECT
SUM(CASE WHEN device_type = 'laptop' THEN 1 ELSE 0 END) AS laptop_views,
SUM(CASE WHEN device_type IN ('tablet','phone') THEN 1 ELSE 0 END) AS mobile_views
FROM viewership;
```

## Facebook - Average Post Hiatus (Part 1)
**"Write a query to find the number of days between each user’s first post of the year and last post of the year in the year 2021"**
```
CREATE TABLE posts(
user_id INTEGER NOT NULL,
post_id INTEGER NOT NULL,
post_content TEXT NOT NULL,
post_date TIMESTAMP NOT NULL);

INSERT INTO posts VALUES
(151325, 451464, 'Garage sale this Saturday 1 PM. All welcome to check out!', '2021-10-25 10:00:00'),
(151325, 613451, 'Happy new year all my friends!', '2022-01-01 11:00:00'),
(151325, 987562, 'The global surface temperature for June 2022 was the sixth-highest in the 143-year record. This is definitely global warming happening.', '2022-07-01 10:00:00'),
(151652, 111766, 'it''s always winter, but never Christmas.', '2021-12-01 11:00:00'),
(151652, 599415, 'Need a hug', '2021-01-28 00:00:00'),
(151652, 994156, 'Does anyone have an extra iPhone charger to sell?', '2021-04-01 10:00:00'),
(178425, 157336, 'I''m so done with these restrictions - I want to travel!!!', '2021-03-24 11:00:00'),
(423967, 784254, 'Just going to cry myself to sleep after watching Marley and Me.', '2021-05-05 00:00:00'),
(661093, 624356, 'Happy valentines!', '2021-02-14 00:00:00'),
(661093, 674356, 'Can''t wait to start my freshman year - super excited!', '2021-08-18 10:00:00'),
(661093, 442560, 'Bed. Class 8-12. Work 12-3. Gym 3-5 or 6. Then class 6-10. Another day that''s gonna fly by. I miss my girlfriend', '2021-09-08 10:00:00');


WITH days AS
(SELECT user_id, (MAX(post_date::DATE) - MIN(post_date::DATE)) AS days_between_posts
FROM posts
WHERE EXTRACT(YEAR FROM post_date) = 2021
GROUP BY user_id)
SELECT user_id, days_between_posts
FROM days
WHERE days_between_posts > 0;
```

## Microsoft - Teams Power Users
**"Write a query to identify the top 2 Power Users who sent the highest number of messages on Microsoft Teams in August 2022"**
```
CREATE TABLE messages(
message_id SMALLINT NOT NULL,
sender_id SMALLINT NOT NULL,
receiver_id SMALLINT NOT NULL,
content TEXT NOT NULL,
sent_date TIMESTAMP NOT NULL);

INSERT INTO messages VALUES
(901, 3601, 4500, 'You up?', '2022-08-03 16:43:00'),
(743, 3601, 8752, 'Let''s take this offline', '2022-06-14 14:30:00'),
(888, 3601, 7855, 'DataLemur has awesome user base!', '2022-08-12 08:45:00'),
(100, 2520, 6987, 'Send this out now!', '2021-08-16 00:35:00'),
(898, 2520, 9630, 'Are you ready for your upcoming presentation?', '2022-08-13 14:35:00'),
(990, 2520, 8520, 'Maybe it was done by the automation process.', '2022-08-19 06:30:00'),
(819, 2310, 4500, 'What''s the status on this?', '2022-07-10 15:55:00'),
(922, 3601, 4500, 'Get on the call', '2022-08-10 17:03:00'),
(942, 2520, 3561, 'How much do you know about Data Science?', '2022-08-17 13:44:00'),
(966, 3601, 7852, 'Meet me in five!', '2022-08-17 02:20:00'),
(902, 4500, 3601, 'Only if you''re buying', '2022-08-03 06:50:00');


SELECT sender_id, COUNT(sender_id) AS message_count
FROM messages
WHERE EXTRACT(YEAR FROM sent_date) = 2022 AND EXTRACT(MONTH FROM sent_date) = 8
GROUP BY sender_id
ORDER BY COUNT(sender_id) DESC
LIMIT 2;
```

## Robinhood - Cities With Completed Trades
**"Write a query to retrieve the top three cities that have the highest number of completed trade orders listed in descending order"**
```
CREATE TABLE users(
user_id SMALLINT PRIMARY KEY,
city VARCHAR(30) NOT NULL,
email VARCHAR(50) NOT NULL,
signup_date TIMESTAMP);

INSERT INTO users VALUES
(111, 'San Francisco', 'rrok10@gmail.com', '2021-08-03 12:00:00'),
(148, 'Boston', 'sailor9820@gmail.com', '2021-08-20 12:00:00'),
(178, 'San Francisco', 'harrypotterfan182@gmail.com', '2022-01-05 12:00:00'),
(265, 'Denver', 'shadower_@hotmail.com', '2022-02-26 12:00:00'),
(300, 'San Francisco', 'houstoncowboy1122@hotmail.com', '2022-06-30 12:00:00'),
(488, 'New York', 'empire_state78@outlook.com', '2022-07-03 12:00:00');

CREATE TABLE trades(
order_id INTEGER NOT NULL,
user_id SMALLINT REFERENCES users(user_id) NOT NULL,
quantity SMALLINT NOT NULL,
status TEXT NOT NULL,
	CONSTRAINT status_c CHECK (status IN ('Completed', 'Cancelled')),
date TIMESTAMP NOT NULL,
price NUMERIC NOT NULL);

INSERT INTO trades VALUES
(100101, 111, 10, 'Cancelled', '2022-08-17 12:00:00', 9.80),
(100102, 111, 10, 'Completed', '2022-08-17 12:00:00', 10.00),
(100264, 148, 40, 'Completed', '2022-08-26 12:00:00', 4.80),
(100305, 300, 15, 'Completed', '2022-09-05 12:00:00', 10.00),
(100909, 488, 1, 'Completed', '2022-07-05 12:00:00', 6.50),
(100259, 148, 35, 'Completed', '2022-08-25 12:00:00', 5.10),
(100900, 148, 50, 'Completed', '2022-07-14 12:00:00', 9.78),
(101432, 265, 10, 'Completed', '2022-08-16 12:00:00', 13.00),
(102533, 488, 25, 'Cancelled', '2022-11-10 12:00:00', 22.40),
(100565, 265, 2, 'Completed', '2022-09-27 12:00:00', 8.70),
(100400, 178, 32, 'Completed', '2022-09-17 12:00:00', 12.00),
(100777, 178, 60, 'Completed', '2022-07-25 17:47:00', 3.56);


SELECT city, COUNT(city) AS total_orders
FROM users JOIN trades ON
users.user_id = trades.user_id
WHERE status = 'Completed'
GROUP BY CITY
ORDER BY COUNT(city) DESC
LIMIT 3;
```

## Amazon - Average Review Ratings
**"Write a query to retrieve the average star rating for each product, grouped by month"**
```
CREATE TABLE reviews(
review_id SMALLINT NOT NULL,
user_id	SMALLINT NOT NULL,
submit_date DATE NOT NULL,
product_id INTEGER NOT NULL,
stars SMALLINT NOT NULL,
	CONSTRAINT stars_c CHECK (stars IN (1,2,3,4,5)));

INSERT INTO reviews VALUES
(6171, 123, '2022-06-08', 50001, 4),
(7802, 265, '2022-06-10', 69852, 4),
(5293, 362, '2022-06-18', 50001, 3),
(6352, 192, '2022-07-26', 69852, 3),
(4517, 981, '2022-07-05', 69852, 2),
(2501, 142, '2022-06-21', 12580, 5),
(4582, 562, '2022-06-15', 12580, 4),
(2536, 136, '2022-07-04', 11223, 5),
(1200, 100, '2022-05-17', 25255, 4),
(2555, 232, '2022-05-31', 25600, 4),
(2556, 167, '2022-05-31', 25600, 5),
(1301, 120, '2022-05-18', 25600, 4);


SELECT 
	EXTRACT (MONTH FROM submit_date) AS month,
	product_id,
	ROUND(AVG(stars),2) AS avg_reviews
FROM reviews
GROUP BY EXTRACT (MONTH FROM submit_date), product_id
ORDER BY month, product_id;
```

## Facebook - App Click-through Rate (CTR)
**"Write a query to calculate the click-through rate for the app in 2022 and round the results to 2 decimal places"**
```
CREATE TABLE events(
app_id	SMALLINT NOT NULL,
event_type TEXT NOT NULL,
	CONSTRAINT event_type_c CHECK (event_type IN ('click', 'impression')),
timestamp TIMESTAMP NOT NULL);


INSERT INTO events VALUES
(123, 'impression', '2022-07-18 11:36:12'),
(123, 'impression', '2022-07-18 11:37:12'),
(123, 'click', '2022-07-18 11:37:42'),
(234, 'impression', '2022-08-18 14:15:12'),
(234, 'click', '2022-08-18 14:16:12'),
(123, 'impression', '2021-10-23 12:11:56'),
(123, 'click', '2021-10-23 12:22:12'),
(123, 'impression', '2022-04-06 13:13:13'),
(123, 'click', '2022-04-07 12:20:30'),
(234, 'impression', '2022-02-09 10:05:02'),
(234, 'impression', '2022-05-20 12:00:00');



WITH imp AS (SELECT app_id, COUNT(event_type) AS ic
FROM events
WHERE event_type = 'impression' AND EXTRACT(YEAR FROM timestamp) = 2022
GROUP BY app_id),
click AS (SELECT app_id, COUNT(event_type) AS cc
FROM events
WHERE  event_type = 'click' AND EXTRACT(YEAR FROM timestamp) = 2022
GROUP BY app_id)
SELECT click.app_id, 100*cc/ic
FROM click JOIN imp on click.app_id = imp.app_id;

-- or

SELECT app_id,
ROUND(100.0 * SUM(CASE WHEN event_type = 'click' THEN 1 ELSE 0 END)
/SUM(CASE WHEN event_type = 'impression' THEN 1 ELSE 0 END),2) AS ctr
FROM events
WHERE EXTRACT(YEAR FROM timestamp) = 2022
GROUP BY app_id;
```

## TikTok - Second Day Confirmation
**"Write a query to display the user IDs of those who did not confirm their sign-up on the first day, but confirmed on the second day"**
```
CREATE TABLE emails(
email_id SMALLINT NOT NULL,
user_id SMALLINT NOT NULL,
signup_date DATE NOT NULL);

INSERT INTO emails VALUES
(125, 7771, '2022-06-14'),
(236, 6950, '2022-07-01'),
(433, 1052, '2022-07-09'),
(450, 8963, '2022-08-02'),
(555, 8963, '2022-08-09'),
(741, 1235, '2022-07-25');

CREATE TABLE texts(
text_id	SMALLINT NOT NULL,
email_id SMALLINT NOT NULL,
signup_action TEXT NOT NULL,
	CONSTRAINT signup_action_c CHECK (signup_action IN ('Confirmed', 'Not confirmed')),
action_date	DATE NOT NULL);

INSERT INTO texts VALUES
(6878, 125, 'Confirmed', '2022-06-14'),
(6997, 433, 'Not confirmed', '2022-07-09'),
(7000, 433, 'Confirmed', '2022-07-10'),
(9841, 236, 'Confirmed', '2022-07-01'),
(2800, 555, 'Confirmed', '2022-08-11'),
(1568, 741, 'Confirmed',' 2022-07-26'),
(1255, 555, 'Not confirmed', '2022-08-09'),
(1522, 741, 'Not confirmed', '2022-07-25');




WITH days_past AS (SELECT user_id, (action_date::DATE) - (signup_date::DATE) AS days
FROM emails 
JOIN texts ON emails.email_id = texts.email_id
WHERE signup_action = 'Confirmed')
SELECT user_id
FROM days_past
WHERE days = 1;

-- or 

SELECT emails.user_id
FROM emails 
JOIN texts ON emails.email_id = texts.email_id
WHERE action_date = signup_date + INTERVAL '1 day'
AND signup_action = 'Confirmed';
```

## Twitter - Tweets' Rolling Averages
**"Given a table of tweet data over a specified time period, calculate the 3-day rolling average of tweets for each user"**
```
CREATE TABLE twitter(
user_id	SMALLINT NOT NULL,
tweet_date DATE NOT NULL,
tweet_count SMALLINT NOT NULL);

INSERT INTO twitter VALUES
(111, '2022-06-01', 2), (111, '2022-06-02', 1), (111, '2022-06-03', 3), (111, '2022-06-04', 4),
(111, '2022-06-05', 5), (111, '2022-06-06', 4), (111, '2022-06-07', 6),
(199, '2022-06-01', 7), (199, '2022-06-02', 5), (199, '2022-06-03', 9), (199, '2022-06-04', 1),
(199, '2022-06-05', 8), (199, '2022-06-06', 2), (199, '2022-06-07', 2),
(254, '2022-06-01', 1), (254, '2022-06-02', 1), (254, '2022-06-03', 2), (254, '2022-06-04', 1),
(254, '2022-06-05', 3), (254, '2022-06-06', 1), (254, '2022-06-07', 3);

SELECT 
	user_id, tweet_date, 
	ROUND(AVG(tweet_count) 
		  OVER(PARTITION BY user_id ORDER BY tweet_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW),2) AS three_day_rolling_avg
FROM twitter
```


## CVS Health Pharmacy Analytics
**(Part 1) "Write a query to find the top 3 most profitable drugs sold, and how much profit they made"   
(Part 2) "Write a query to identify the manufacturers associated with the drugs that resulted in losses for CVS Health and calculate the total amount of losses incurred"   
(Part 3) "Write a query to calculate the total drug sales for each manufacturer. Round the answer to the nearest million and report your results in descending order of total sales"**

```
CREATE TABLE pharmacy_sales(
product_id SMALLINT NOT NULL,
units_sold INTEGER NOT NULL,
total_sales NUMERIC NOT NULL,
cogs NUMERIC NOT NULL,
manufacturer VARCHAR(25) NOT NULL,
drug VARCHAR(50) NOT NULL);

INSERT INTO pharmacy_sales VALUES
(156,89514,3130097,3427421.73,'Biogen','Acyclovir'), (25,222331,2753546,2974975.36,'AbbVie','Lamivudine and Zidovudine'), (50,90484,2521023.73,2742445.9,'Eli Lilly','Dermasorb TA Complete Kit'), (41,189925,3499574.92,3692136.66,'AbbVie','Clarithromycin'), (63,93513,2104765,2462370.76,'Johnson & Johnson','Pepcid AC Acid Reducer'), (8,177270,2930134.52,3035522.06,'Johnson & Johnson','Nicorobin Clean and Clear'), (75,164674,1184664.57,1285326.93,'Eli Lilly','RED GINSENG FERMENTED ESSENCE BB'), (91,97765,1115255.32,1201044.27,'Roche','Hydrochlorothiazide'), (26,126866,1499768.09,1573992.41,'Eli Lilly','LBel'), (16,51707,1304837.86,1378790.53,'Roche','Topcare Tussin'), (80,61467,3740527.69,3804542.2,'Biogen','Losartan Potassium'), (148,104637,837620.18,931084.25,'Johnson & Johnson','Motrin'), (95,128494,723841.23,779520.88,'Biogen','Wal-Zan'), (56,86598,1755300.92,1806344.97,'Eli Lilly','Spot Repairing Serum'), (71,126265,2564743.39,2593528.67,'Bayer','ENALAPRIL MALEATE'), (35,87449,86938.27,99811.26,'Johnson & Johnson','Sanitary Wipes Plus'), (70,167190,1119479.36,1313174.69,'Johnson & Johnson','Zyrtec Ultra-Strength'), (15,118901,2717420.96,2707620.02,'Biogen','Clotrimazole'), (33,149895,949514.05,921206.75,'Bayer','Levofloxacin'), (179,125006,1825970,1769907.97,'Biogen','Lancome Paris Renergie Lift Volumetry'), (21,50550,697276.33,640063.57,'Pfizer','Venlafaxine Hydrochloride'), (125,101102,566696,508144.71,'AbbVie','Lidocaine Hydrochloride and Epinephri'), (76,87699,3257976.38,3389863.82,'Johnson & Johnson','EltaMD SPF 150 Sun Screen'), (136,144814,1084258,1006447.73,'Biogen','Burkhart'), (34,94698,600997.19,521182.16,'AstraZeneca','Surmontil'), (61,77023,500101.61,419174.97,'Biogen','Varicose Relief'), (9,37410,293452.54,208876.01,'Eli Lilly','Zyprexa'), (67,87756,1112253.82,1021908.39,'Biogen','Pramipexole Dihydrochloride'), (105,97736,537795,426539.59,'Pfizer','Diaper Rash Skin Protectant Crema Cer'), (107,160617,2538701.5,2414037.51,'Biogen','N - TIME SINUS'), (89,61832,1084996.13,940593.68,'Sanofi','Locoid'), (30,142661,1615518.35,1439533.27,'Sanofi','Oxaprozin'), (47,130448,461623.76,282038.76,'Eli Lilly','Night Time Cherry Syrup'), (146,71159,1778024,1598276.66,'AstraZeneca','PDI Sani-Hands for Kids'), (52,47310,1151498.6,956693.99,'AstraZeneca','Armour Thyroid'), (183,155058,1220029.58,1023275.76,'AbbVie','Lexapro'), (113,122655,1358711.57,1161623.36,'Novartis','Famotidine'), (29,217652,3106931.54,2898165.87,'AstraZeneca','MiraLAX'), (171,177686,632705.44,413382.39,'AstraZeneca','lansoprazole'), (2,284705,523311.9,302721.56,'Novartis','Imatinib'), (126,165665,586961.45,365016.1,'AbbVie','Hamamelis Virginiana Kit Refill'), (122,187110,681903.65,458208.23,'GlaxoSmithKline','eszopiclone'), (144,110672,841911.12,616632.58,'Bayer','Acyclovir Sodium'), (14,55657,933692.06,707891.52,'Bayer','Triple Complex Brain Tonic'), (97,128893,1114438.97,887895.38,'AbbVie','Sodium Iodide I 123'), (111,152231,2708053.52,2467067.02,'Eli Lilly','Divalproex Sodium'), (31,94969,1104640.13,848011.39,'Biogen','Orsythia'), (51,95572,2582349,2322161.57,'Pfizer','Methadone Hydrochloride'), (181,104441,1045454.54,772824.85,'Merck','FLU KIDS RELIEF'), (135,160565,2524229,2249571,'Bayer','Meloxicam'), (22,143516,3147031.19,2864248.62,'Sanofi','Synthroid'), (37,232228,507628.5,212607.07,'Johnson & Johnson','Hand wash'), (154,96249,1408872.02,1105672.68,'Eli Lilly','Potassium Chloride in Dextrose and So'), (59,215282,3059122.36,2755697.86,'Johnson & Johnson','Common Sagebrush'), (81,117947,1559816,1233306.52,'Merck','DHEA'), (3,192608,2851411.66,2522812.85,'Eli Lilly','Treflan'), (53,121093,1460189.98,1124982.66,'Biogen','Active-Medicated specimen collection'), (160,97520,1415772.18,1067854.28,'AbbVie','Olay Total Effects Pore Minimizing CC'), (46,189293,1528747.03,1172798.04,'Pfizer','Stay Awake'), (85,112969,2996318.23,2614556.03,'Bayer','Lovastatin'), (143,180697,1364046.09,961852.31,'Eli Lilly','Western Family Pain Relieving'), (83,106994,1803199.39,1397575.13,'Eli Lilly','IOPE RETIGEN MOISTURE FOUNDATION'), (165,121043,961716.2,553770.53,'Bayer','PROMETHAZINE HYDROCHLORIDE'), (42,116447,2429913,2002106.22,'Biogen','Gelato Topical Anesthetic'), (157,197527,2204939.12,1749148.23,'Biogen','Non Aspirin PM'), (11,135549,2792775.1,2327541.25,'Biogen','AVAPRO'), (49,95307,700562.13,221803.73,'Biogen','Brucella Remedy'), (115,102792,821214.25,321027.88,'Pfizer','Methylphenidate Hydrochloride'), (18,58580,1342015.48,838168.71,'AstraZeneca','Enalapril Maleate'), (119,214488,657085.12,152614.97,'Johnson & Johnson','Hand Sanitizer with Moisturizers'), (137,128175,822716.26,316753.86,'AbbVie','Warfarin Sodium'), (169,95653,1270441.66,757724.19,'Eli Lilly','Multi Symptom Cold'), (164,89816,1080726.88,565920.33,'Biogen','RED ORANGE SUN'), (117,191289,778332.7,256621.14,'Bayer','Diphenhydramine HCL'), (106,103889,1698544.77,1164637.24,'Roche','Crab'), (120,148170,1094440.19,560320.28,'Bayer','Gabapentin'), (10,123832,1518343,981993.64,'Novartis','Amoxicillin and Clavulanate Potassium'), (6,64028,1253627.93,715430.59,'Bayer','Riociguat'), (186,150221,3116140.22,2560951.38,'Novartis','ANASTROZOLE'), (139,38522,674397.16,92756.59,'Merck','METHOCARBAMOL'), (100,212506,3091720.63,2509926.79,'Pfizer','Ropinirole Hydrochloride'), (84,174098,2752163,2146503.01,'Pfizer','Tranexamic Acid'), (45,144352,708891.28,95912.97,'Roche','Dorzolamide HCl'), (112,92168,1194250.01,575583.41,'Novartis','Alimta'), (167,186778,904145,240493.13,'Biogen','Monistat Complete Care Instant Itch R'), (28,113370,1379945.1,712265.84,'AstraZeneca','Thyroid Assist'), (188,123956,1658835.03,991099.63,'Roche','Nivolumab'), (94,132362,2041758.41,1373721.7,'Biogen','UP and UP'), (98,110746,813188.82,140422.87,'Biogen','Medi-Chord'),
(65,151901,967632.61,278350.32,'AbbVie','Moxifloxacin Hydrochloride'), (124,59167,1033654.77,344270,'Bayer','MooreBrand Ibuprofen'), (23,72264,1892705.42,1202177.41,'Roche','Cloves'), (149,171975,1727624.67,1024897.45,'Johnson & Johnson','SunZone Work Sunscreen SPF-60'), (64,69702,1425027.99,710573.44,'AbbVie','Allopurinol'), (118,131788,1336647.6,595485.21,'AbbVie','Chlorzoxazone'), (123,125721,1796908.48,1041602.54,'Novartis','Diltiazem Hydrochloride'), (88,175710,1443369.73,670011.51,'Bayer','Oxygen'), (24,289121,937901.5,163610.19,'AstraZeneca','BANANA BOAT SUNSCREEN'), (60,260823,2341257.97,1553229.42,'Eli Lilly','Metoclopramide'), (172,78481,985931,192743.12,'Bayer','Xarelto'), (161,175833,1201626.71,407210.29,'AbbVie','Citalopram'), (184,40007,875514.67,64948.76,'Biogen','Imraldi'), (127,139627,2912543.88,2095983.81,'AstraZeneca','Pravastatin Sodium'), (20,201740,2937100,2117864.7,'Eli Lilly','Levothyroxine Sodium'), (180,88653,2115658,1278017.17,'Pfizer','Ciprofloxacin'), (69,72195,1862729.02,1021659.26,'Biogen','Prednisone'), (72,159597,1752274.54,907074.49,'Johnson & Johnson','Flu-Cold'),
(1,166050,1857681.77,999626.88,'Eli Lilly','Keflex'), (48,157217,2878612.16,2020543.63,'Sanofi','Persantine'), (27,153025,1793428.19,908662.72,'Eli Lilly','Cefaclor'), (99,104368,1146201.6,250683.09,'AbbVie','Hydralazine Hydrochloride'), (43,176445,1355861.1,412421.37,'Bayer','Citalopram Hydrobromide'), (176,71329,2529937.41,1584233.81,'Biogen','RHIZOPUS ARRHIZUS VAR ARRHIZUS'), (40,136973,2723073.91,1765245.63,'Pfizer','Selegiline Hydrochloride'), (131,48864,1540350.32,569738.58,'Biogen','ESIKA'), (116,76115,2510105,1505831.15,'Biogen','Namenda'), (93,175376,2218026.75,1206180.64,'Biogen','ProBLEN Estrogen and Progesterone'), (170,107101,1958490.97,914881.77,'AstraZeneca','Double Antibiotic'), (54,68593,1545347.97,482212.61,'Eli Lilly','Diclofenac Sodium'), (44,131514,3080474.64,2014858.59,'Biogen','Alprazolam'), (17,121564,1466145.44,387524.97,'AstraZeneca','SPRYCEL'),
(86,147885,1808978.69,720231.01,'Pfizer','Divalproex Sodium Extended-Release'), (178,109829,2786292.44,1693474.52,'Roche','Motion Sickness II'), (87,110248,1485507.76,352858.53,'AbbVie','Night-Time Original'), (104,182090,3217507.49,2069390.45,'AbbVie','Stavudine'), (109,118696,1433109.5,263857.96,'Eli Lilly','Tizanidine Hydrochloride'), (150,201167,2468436,1298844.51,'Biogen','Nefazodone Hydrochloride'), (155,167357,2257132.08,1072766.97,'AstraZeneca','Zavesca'), (128,103246,2700252,1511930.53,'Biogen','QUETIAPINE FUMARATE'), (96,102627,1879590.38,669228.79,'AstraZeneca','Listerine Ultraclean Antiseptic'), (141,113510,2748508,1519635.54,'Eli Lilly','Kentucky Bluegrass (June), Standardiz'), (57,139567,2928590.94,1698018.23,'AbbVie','Valproic Acid'), (140,210120,2944223.22,1649161.98,'Roche','Lucentis'),
(66,134990,1734572,436399.12,'Novartis','Antiseptic Hand Gel'), (77,66608,1406533.69,107902.51,'Johnson & Johnson','Budpak Petroleum Jelly'), (185,137096,2582356,1281909.78,'Pfizer','clobetasol propionate'), (138,154713,1743859.96,398395.47,'Eli Lilly','Treatment Set TS347116'), (142,122444,1428293.39,82371.13,'Roche','Herceptin'), (134,37293,3485649.76,2128714.17,'Eli Lilly','Eldepryl'), (159,109063,1780520.27,423335.39,'Eli Lilly','Levetiracetam'), (102,71484,1442991.59,82418.43,'AstraZeneca','Naproxen Sodium'), (5,231084,1825783.98,453989.65,'Biogen','DIPHENHYDRAMINE HYDROCHLORIDE'), (78,107503,2108286,694560.97,'Eli Lilly','Rimmel London'), (90,129019,2543749.63,1096231.45,'Biogen','KADCYLA'), (38,85086,3358913.43,1898929.25,'Eli Lilly','STEMPHYLIUM SOLANI'), (187,144880,2604075,1130014.83,'Eli Lilly','DIVALPROEX SODIUM'), (151,204188,3208125,1731552.31,'Johnson & Johnson','Niacin'), (168,118682,2292546.89,789582.98,'Biogen','Lamotrigine'), (147,164693,3023426,1485390.52,'Biogen','Nortriptyline Hydrochloride'), (174,99442,3303186.78,1754344.77,'Eli Lilly','Clobetasol Propionate'), (13,216001,2807831.05,1242490.08,'Johnson & Johnson','Gold Bond Ultimate Healing Concentrate'), (152,75583,3379737.8,1774390.9,'GlaxoSmithKline','Isoniazid'), (7,224342,2313335.25,588699.33,'Johnson & Johnson','Ibuprofen'), (110,155316,1893117.08,134244.15,'Johnson & Johnson','Remicade'), (153,178254,1995651.94,123085.04,'Biogen','Haloperidol'), (12,164685,2938804.92,1065324.19,'AstraZeneca','SKIN FOUNDATION MINERAL MAKEUP'), (79,140129,3560527.35,1582876.55,'AstraZeneca','Green Guard Stomach Relief'), (166,62913,4701990.22,2705112.11,'Bayer','Amitriptyline Hydrochloride'), (129,116649,2933877,934986.87,'Biogen','Cuvposa'),(36,128104,2896459.68,862377.49,'Eli Lilly','Pfizerpen'), (162,130946,3189843.38,1058358.52,'Eli Lilly','Morphine Sulfate'), (132,92089,2443119.04,225171.93,'Eli Lilly','OLUX-E'), (62,140625,3417877.93,1177021.82,'Pfizer','MUCOR PLUMBEUS'), (173,147808,2738142.96,493523.99,'AstraZeneca','Furosemide'), (73,117857,2517600.95,195721.74,'Biogen','Fumaderm'), (163,165666,2843262.48,509536.85,'Pfizer','TAMSULOSIN HYDROCHLORIDE'), (103,102027,3025655,675055.5,'Eli Lilly','Androgel'), (108,136804,3401782.44,992181.28,'Bayer','Sheep Sorrel Pollen'), (114,86112,2825302,412654.26,'Bayer','Claritin'), (55,161870,2835232.18,384241.13,'Johnson & Johnson','Childrens Ibuprofen'), (130,150704,3556698.88,1083810.04,'Johnson & Johnson','Medi-First Cold Relief'), (92,109858,3283823,679925.49,'Eli Lilly','Naratriptan'),
(19,128656,3179533.5,410405.24,'Bayer','Ibuprofen PM'), (4,206134,3786377,796869.55,'Sanofi','Lovenox'), (121,140287,3329242.48,261606.11,'AbbVie','Glipizide'), (39,244224,3581454.82,417476.74,'Johnson & Johnson','JUNIPERUS ASHEI POLLEN'), (32,124071,3765829,156664.74,'Johnson & Johnson','XtraCare Foaming Facial Cleanser'), (158,79869,7925929.18,3825914.37,'Merck','Divalproex sodium'), (192,498342,9654492.33,3189863.82,'Eli Lilly','Cialis'), (177,123213,14759462.01,7083504.56,'Novartis','Xanax'), (191,274342,12654492.33,1437439.99,'Sanofi','Dupixent'), (190,240704,13759462.01,2137439.99,'Merck','Keytruda'), (189,99858,84759462.01,3243809.46,'AbbVie','Humira');


-- part 1

SELECT drug, (total_sales - cogs) AS profit
FROM pharmacy_sales
ORDER BY profit DESC
LIMIT 3;
 

-- part 2

SELECT manufacturer, COUNT(drug) AS drug_count, ABS(SUM(total_sales - cogs)) AS total_loss
FROM pharmacy_sales
WHERE (total_sales - cogs) < 0
GROUP BY manufacturer
ORDER BY total_loss DESC;

-- part 3

SELECT manufacturer, CONCAT('$',ROUND(SUM(total_sales)/1000000), ' million') AS complete_total_sales
FROM pharmacy_sales
GROUP BY manufacturer
ORDER BY SUM(total_sales) DESC;
```

## Amazon - Highest-Grossing Items
**"Write a query to identify the top two highest-grossing products within each category in the year 2022"**
```
CREATE TABLE product_spend(
category VARCHAR(25) NOT NULL,
product	VARCHAR(50) NOT NULL,
user_id	SMALLINT NOT NULL,
spend NUMERIC NOT NULL,
transaction_date TIMESTAMP NOT NULL);

INSERT INTO product_spend VALUES
('appliance', 'washing machine', 123, 219.80, '2022-03-02 11:00:00'),
('electronics', 'vacuum', 178, 152.00, '2022-04-05 10:00:00'),
('electronics', 'wireless headset', 156, 249.90, '2022-07-08 10:00:00'),
('electronics', 'vacuum', 145, 189.00, '2022-07-15 10:00:00'),
('electronics', 'computer mouse', 195, 45.00, '2022-07-01 11:00:00'),
('appliance', 'refrigerator', 165, 246.00, '2021-12-26 12:00:00'),
('appliance', 'refrigerator', 123, 299.99, '2022-03-02 11:00:00'),
('appliance', 'washing machine', 123, 220.00, '2022-07-27 04:00:00'),
('electronics', 'vacuum', 156, 145.66, '2022-08-10 04:00:00'),
('electronics', 'wireless headset', 145, 198.00, '2022-08-04 04:00:00'),
('electronics', 'wireless headset', 215, 19.99, '2022-09-03 16:00:00'),
('appliance', 'microwave', 169, 49.99, '2022-08-28 16:00:00'),
('appliance', 'microwave', 101, 34.49, '2023-03-01 17:00:00'),
('electronics', '3.5mm headphone jack', 101, 7.99, '2022-10-07 16:00:00'),
('appliance', 'microwave', 101, 64.95, '2023-07-08 16:00:00');

select * from product_spend order by category

WITH rankings AS
(SELECT category, product, SUM(spend) AS total_sum, RANK() OVER (PARTITION BY category ORDER BY SUM(spend) DESC) AS rank
FROM product_spend
WHERE EXTRACT(YEAR FROM transaction_date) = 2022
GROUP BY category, product)
SELECT category, product, total_sum
FROM rankings
WHERE rank IN (1,2);
```

## Spotify - Top 5 Artists
**"Write a query to find the top 5 artists whose songs appear most frequently in the Top 10. Display the top 5 artist names"**

```
CREATE TABLE artists(
artist_id SMALLINT NOT NULL,
artist_name VARCHAR(30) NOT NULL,
label_owner VARCHAR(40) NOT NULL);

INSERT INTO artists VALUES
(101, 'Ed Sheeran', 'Warner Music Group'),
(120, 'Drake', 'Warner Music Group'),
(125, 'Bad Bunny', 'Rimas Entertainment'),
(145, 'Lady Gaga', 'Interscope Records'),
(160, 'Chris Brown', 'RCA Records'),
(200, 'Adele', 'Columbia Records'),
(240, 'Katy Perry', 'Capitol Records'),
(250, 'The Weeknd', 'Universal Music Group'),
(260, 'Taylor Swift', 'Universal Music Group'),
(270, 'Ariana Grande', 'Universal Music Group');

CREATE TABLE songs(
song_id	INTEGER NOT NULL,
artist_id	SMALLINT NOT NULL,
name VARCHAR(50));

INSERT INTO songs VALUES
(55511, 101, 'Perfect'), (45202, 101, 'Shape of You'), (22222, 120, 'One Dance'),
(19960, 120, 'Hotline Bling'), (12636, 125, 'Mia'), (69820, 125, 'Dakiti'),
(44552, 125, 'Callaita'), (11254, 145, 'Bad Romance'), (33101, 160, 'Go Crazy'),
(23299, 200, 'Hello'), (89633, 240, 'Last Friday Night'), (28079, 200, 'Someone Like You'),
(13997, 120, 'Rich Flex'), (14525, 260, 'Cruel Summer'), (23689, 260, 'Blank Space'),
(54622, 260, 'Wildest Dreams'), (62887, 260, 'Anti-Hero'), (56112, 270, '7 Rings'),
(86645, 270, 'Thank U, Next'), (87752, 260, 'Karma'), (23339, 250, 'Blinding Lights');

CREATE TABLE global_song_rank(
day INTEGER
	CONSTRAINT day_c CHECK(day BETWEEN 1 AND 52),
song_id	INTEGER NOT NULL,
rank INTEGER NOT NULL);

INSERT INTO global_song_rank VALUES
(1,45202,2), (3,45202,2), (15,45202,6), (2,55511,2), (1,19960,3), (9,19960,15),
(23,12636,9), (24,12636,7), (2,12636,23), (29,12636,7), (1,69820,1), (17,44552,8),
(11,44552,16), (11,11254,5), (12,11254,16), (3,33101,16), (6,23299,1), (14,89633,2),
(9,28079,9), (7,28079,10), (40,11254,1), (37,23299,5), (19,11254,10), (23,89633,10),
(52,33101,7), (20,55511,10), (7,22222,8), (8,44552,1), (1,54622,34), (2,44552,1),
(2,19960,3), (3,260,1), (3,22222,35), (3,56112,3), (4,14525,1), (4,23339,29),
(4,13997,5), (13,87752,1), (14,87752,1), (1,11254,12), (51,13997,1), (52,28079,75),
(15,87752,1), (5,14525,1), (6,14525,2), (7,14525,1), (40,33101,13), (1,54622,84),
(7,62887,2), (50,89633,67), (50,13997,1), (33,13997,3), (1,23299,9);


WITH best AS
(SELECT artist_name, COUNT(artist_name) AS top_ten_count, DENSE_RANK() OVER(ORDER BY COUNT(artist_name) DESC) AS top_ten_rank
FROM artists AS a
JOIN songs AS s
ON a.artist_id = s.artist_id
JOIN global_song_rank AS g
ON s.song_id = g.song_id
WHERE rank <=10
GROUP BY artist_name)
SELECT artist_name, top_ten_rank
FROM best
WHERE top_ten_rank <=5;
```



