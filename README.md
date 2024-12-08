# Bellabeat Case Study

## 1. Background

Bellabeat is a pioneering wellness technology company founded by Urška Sršen and Sando Mur in 2013, focusing on women's health and wellness. Known for its innovative wearable and non-wearable tech, Bellabeat offers products that include the Leaf wellness tracker, the Time wellness watch, and the Spring water bottle, all of which connect to the Bellabeat app. This app provides users with insights into their activity, sleep, stress, menstrual cycle, and mindfulness habits.
With a mission to empower women by providing valuable health information and motivational tips, Bellabeat fosters a community centered on holistic well-being. The company has rapidly grown since its inception, launching multiple products and expanding its global presence. Bellabeat employs a marketing analytics team, which includes junior data analysts, to analyze consumer data and inform marketing strategies.
Sršen, as Chief Creative Officer, combines her artistic background with technology to create beautifully designed products that inspire women worldwide. By collecting data on activity, sleep, stress, and reproductive health, Bellabeat enables women to better understand their health and habits. Through its subscription-based membership program, users gain personalized guidance on nutrition, activity, sleep, health, beauty, and mindfulness tailored to their lifestyles and goals. Overall, Bellabeat represents a movement to seamlessly integrate wellness into everyday life, helping women navigate their unique health journeys.


## 2. ASK Section

As a junior data analyst on the marketing analytics team at Bellabeat, my role is to support the company's growth in the competitive global smart device market. Co-founder Urška Sršen has identified a significant opportunity to leverage smart device usage data to better understand consumer behavior and enhance our marketing strategies.
I have been tasked with analyzing Fitbit Fitness Tracking Data to uncover insights on how consumers use Bellabeat smart devices. These insights will be crucial in guiding marketing strategies for one of Bellabeat's products. My findings will be presented to the executive team along with high-level recommendations.

## 3. PREPARE Section

This dataset features a public available Fitbit dataset, which includes fitness tracker data from users who consented to share their daily activity, steps, heart rate, and sleep patterns. The data is structured in a long format, allowing for easy tracking of changes over time for each user. However, the sample size is small, varying from 8 to 33 users across the different files, which introduces limitations in terms of representativeness and potential biases related to user demographics.
It's important to highlight that the dataset does not meet ROCCC standards (Reliable, Original, Comprehensive, Current, and Cited), because the data does not originate from Bellabeat, which raises potential concerns regarding bias and credibility. Additionally, the dataset is limited to April and May of 2016, meaning that the analysis may not accurately represent current trends in smart device usage. and is licensed under CC0: Public Domain, allowing unrestricted use. 
After downloading and reviewing all 18 files, I conducted the first phase of data cleaning using Excel. This included initial checks for completeness, accuracy, consistency, and reliability. I identified that only ten files contain 33 unique user IDs: `dailyActivity_merged`, ‘dailyCalories_merged’, ‘dailyIntensities_merged’,  ‘dailySteps_merged’, `hourlyCalories_merged`, ‘hourlyIntensities_merged’, `hourlySteps_merged`, `minuteCaloriesWide_merged`, `minuteIntensitiesWide_merged`, and `minuteStepsWide_merged`. For my analysis, I decided to focus on six key files: dailyActivity_merged`, ‘dailyCalories_merged’, ‘dailyIntensities_merged’,  ‘dailySteps_merged’, `hourlyCalories_merged`, `hourlySteps_merged, as they best align with my business objectives.
The initial data cleaning process was conducted using Google Sheets, following a structured approach for each dataset.
1. Sorting and Filtering: I began by sorting and filtering the data to get a clear view of its structure and identify any irregularities.
2. Checking for Blanks and Duplicates: Next, I systematically checked for blank cells to ensure completeness across all columns.
3. Formatting Date and Time: I reformatted the date and time columns to maintain consistency:
   - Dates were formatted as `MM/DD/YYYY`.
   - Time columns were formatted as `hh:mm:ss AM/PM`.
4. Formatting Numeric Data: Fields like steps and calories were formatted as numerical values to avoid any issues during calculations or aggregations.
 After completing this Google sheets-based cleaning process, I moved on to SQL for further analysis and more advanced data manipulation.

## 4. PROCESS Section

After the first cleaning with google sheets I upload the csv files that I decide that I will need ‘dailyActivity_merged’, ‘dailyCalories_merged’, ‘dailyIntensities_merged’,  ‘dailySteps_merged’, `hourlyCalories_merged`, `hourlySteps_merged,  to BigQuery under the project my-project-2024-423122.bellabeat. 

- i take a closer look at the tables dailyactivity, dailycalories, dailyintensities, and dailysteps with similar columns id and ActivityDay, to confirm whether the data were the same based on their user ids and activityday I ran the following query:

```sql
--The WITH clause creates a Common Table Expression (CTE) named combined_data.
WITH combined_data AS (
SELECT id, activityDay FROM `my-project-2024-423122.bellabeat.dailyActivity_merged`
--The UNION ALL operator combines all rows from the four tables, including duplicates.
UNION ALL
SELECT id, activityDay FROM `my-project-2024-423122.bellabeat.dailyCalories_merged`
UNION ALL
SELECT id, activityDay FROM `my-project-2024-423122.bellabeat.dailyIntensities_merged`
UNION ALL
SELECT id, activityDay FROM `my-project-2024-423122.bellabeat.dailySteps_merged`
)
SELECT
id,
activityDay,
--The COUNT(*) function counts the number of occurrences of each unique id and activityDay pair.
COUNT(*) AS count
FROM
combined_data
--The GROUP BY clause groups the combined data by id and activityDay.
GROUP BY
id,
activityDay
--The HAVING clause filters the results to only include groups where the count is not equal to 4. This indicates that there are missing or inconsistent rows for that specific id and activityDay combination.
HAVING
COUNT(*) <> 4;
```

![1](https://github.com/user-attachments/assets/6a1eb5de-8f90-4763-8dee-ed2964f803dd)
This method involves creating a CTE to combine the four tables and then checking for inconsistencies.
This query effectively checks for inconsistencies in the id and ActivityDay columns across the four tables. The query results in no data, it means that there are no rows in the combined dataset that have inconsistent Id and ActivityDay values across the four tables. This indicates that the data in these columns is consistent across all four tables.

- I check if the calories column in the dailyActivity_merged table is the same as the calories column in the dailyCalories_merged table in BigQuery, so I run the following query:

```sql
SELECT 
    da.id,
    da.activityday,
    da.calories AS calories_dailyActivity,
    dc.calories AS calories_dailyCalories,
    CASE 
        WHEN da.calories = dc.calories THEN "Match"
        ELSE "Mismatch"
    END AS comparison_result
FROM 
    `my-project-2024-423122.bellabeat.dailyActivity_merged` AS da
JOIN 
    `my-project-2024-423122.bellabeat.dailyCalories_merged` AS dc
ON 
    da.id = dc.id
    AND da.activityday = dc.activityday
    WHERE da.calories != dc.calories
```

![2](https://github.com/user-attachments/assets/22793eb9-ce68-4c3c-a64a-dc6cb97f0dcf)

This returns no results, that means that the calories columns are the same for all matched rows.

- I check if the TotalSteps column in the dailyActivity_merged table is the same as the StepTotal column in the dailySteps_merged table in BigQuery, I run the following query:

```sql
1.	SELECT 
2.	    da.id,
3.	    da.activityday,
4.	    da.TotalSteps AS total_steps_dailyActivity,
5.	    dc.StepTotal AS total_steps_dailySteps,
6.	    CASE 
7.	        WHEN da.TotalSteps = dc.StepTotal THEN "Match"
8.	        ELSE "Mismatch"
9.	    END AS comparison_result
10.	FROM 
11.	    `my-project-2024-423122.bellabeat.dailyActivity_merged` AS da
12.	JOIN 
13.	    `my-project-2024-423122.bellabeat.dailySteps_merged` AS dc
14.	ON 
15.	    da.id = dc.id
16.	    AND da.activityday = dc.activityday
17.	    WHERE da.TotalSteps != dc.StepTotal
```

![3](https://github.com/user-attachments/assets/c6baba1a-b2c7-4765-bba7-3ce0a6e9d1c2)
This returns no results, that means that these dy columns are the same for all matched rows.

- Then, I check if these columns SedentaryMinutes, LightlyActiveMinutes, FairlyActiveMinutes, VeryActiveMinutes,	SedentaryActiveDistance, LightActiveDistance, ModeratelyActiveDistance, VeryActiveDistance are same in the  tables dailyActivity_merged and dailyIntensities_merged that have two same columns id and activityday I use a SQL query that joins the tables on id and activityday and then compares each of the columns.

```sql
1.	SELECT
2.	  t1.id,
3.	  t1.activityday,
4.	  -- Check if each column matches and flag mismatches
5.	  IF(t1.SedentaryMinutes = t2.SedentaryMinutes, TRUE, FALSE) AS SedentaryMinutes_match,
6.	  IF(t1.LightlyActiveMinutes = t2.LightlyActiveMinutes, TRUE, FALSE) AS LightlyActiveMinutes_match,
7.	  IF(t1.FairlyActiveMinutes = t2.FairlyActiveMinutes, TRUE, FALSE) AS FairlyActiveMinutes_match,
8.	  IF(t1.VeryActiveMinutes = t2.VeryActiveMinutes, TRUE, FALSE) AS VeryActiveMinutes_match,
9.	  IF(t1.SedentaryActiveDistance = t2.SedentaryActiveDistance, TRUE, FALSE) AS SedentaryActiveDistance_match,
10.	  IF(t1.LightActiveDistance = t2.LightActiveDistance, TRUE, FALSE) AS LightActiveDistance_match,
11.	  IF(t1.ModeratelyActiveDistance = t2.ModeratelyActiveDistance, TRUE, FALSE) AS ModeratelyActiveDistance_match,
12.	  IF(t1.VeryActiveDistance = t2.VeryActiveDistance, TRUE, FALSE) AS VeryActiveDistance_match,
13.	  -- Count mismatches
14.	  CASE
15.	    WHEN t1.SedentaryMinutes = t2.SedentaryMinutes
16.	      AND t1.LightlyActiveMinutes = t2.LightlyActiveMinutes
17.	      AND t1.FairlyActiveMinutes = t2.FairlyActiveMinutes
18.	      AND t1.VeryActiveMinutes = t2.VeryActiveMinutes
19.	      AND t1.SedentaryActiveDistance = t2.SedentaryActiveDistance
20.	      AND t1.LightActiveDistance = t2.LightActiveDistance
21.	      AND t1.ModeratelyActiveDistance = t2.ModeratelyActiveDistance
22.	      AND t1.VeryActiveDistance = t2.VeryActiveDistance THEN TRUE
23.	    ELSE FALSE
24.	  END AS all_columns_match
25.	FROM
26.	  `my-project-2024-423122.bellabeat.dailyActivity_merged` AS t1
27.	JOIN
28.	  `my-project-2024-423122.bellabeat.dailyIntensities_merged` AS t2
29.	ON
30.	  t1.id = t2.id
31.	  AND t1.activityday = t2.activityday
```

![4](https://github.com/user-attachments/assets/db772706-7fbb-4177-898a-87287ff0d824)
The results are all true, which means that these columns are the same in these two tables.
After all these results I decided for my analysis to use only one table, the dailyActivity_merged table.


## 5.	ANALYZE and SHARE Section

In this phase, a thorough analysis of key insights on user engagement with Bellabeat smart devices was conducted using the BigQuery console.User Activity Summary Statistics

### 5.1 User Data Logging Frequency Distribution
First, I wanted to see how many times the users wore/used the FitBit tracker:

```sql
1.	WITH log_counts AS (
2.	    SELECT Id,
3.	           COUNT(Id) AS Id_logs
4.	    FROM `my-project-2024-423122.bellabeat.dailyActivity_merged`
5.	    GROUP BY Id
6.	)
7.	
8.	SELECT Id_logs AS `number of times logged data`,
9.	       COUNT(Id) AS `number of users`
10.	FROM log_counts
11.	GROUP BY Id_logs
12.	ORDER BY Id_logs;
```

![5](https://github.com/user-attachments/assets/0d57cade-6ee3-4edc-a984-2870f03268e7)

21 of users tracked their data throughout the entire period from April 12, 2016, to May 12, 2016. When including users who missed only 1 to 3 days, the number increases to 27 indicating that this portion of users logged data or consistently wore their FitBit Tracker over the month.

### 5.2 User Activity Summary Statistics
Next, I aimed to analyze the minimum, maximum, and average values for total steps, total distance, calories, and activity levels by ID.

```sql
1.	(SELECT Id,
2.	MIN(TotalSteps) AS Min_Total_Steps,
3.	MAX(TotalSteps) AS Max_Total_Steps, 
4.	AVG(TotalSteps) AS Avg_Total_Stpes,
5.	MIN(TotalDistance) AS Min_Total_Distance, 
6.	MAX(TotalDistance) AS Max_Total_Distance, 
7.	AVG(TotalDistance) AS Avg_Total_Distance,
8.	MIN(Calories) AS Min_Total_Calories,
9.	MAX(Calories) AS Max_Total_Calories,
10.	AVG(Calories) AS Avg_Total_Calories,
11.	MIN(VeryActiveMinutes) AS Min_Very_Active_Minutes,
12.	MAX(VeryActiveMinutes) AS Max_Very_Active_Minutes,
13.	AVG(VeryActiveMinutes) AS Avg_Very_Active_Minutes,
14.	MIN(FairlyActiveMinutes) AS Min_Fairly_Active_Minutes,
15.	MAX(FairlyActiveMinutes) AS Max_Fairly_Active_Minutes,
16.	AVG(FairlyActiveMinutes) AS Avg_Fairly_Active_Minutes,
17.	MIN(LightlyActiveMinutes) AS Min_Lightly_Active_Minutes,
18.	MAX(LightlyActiveMinutes) AS Max_Lightly_Active_Minutes,
19.	AVG(LightlyActiveMinutes) AS Avg_Lightly_Active_Minutes,
20.	MIN(SedentaryMinutes) AS Min_Sedentary_Minutes,
21.	MAX(SedentaryMinutes) AS Max_Sedentary_Minutes,
22.	AVG(SedentaryMinutes) AS Avg_Sedentary_Minutes
23.	From `my-project-2024-423122.bellabeat.dailyActivity_merged`
24.	Group BY Id
25.	)
```

So the values varies for: 
- Min_Total_Steps 0-4790
- Max_Total_Steps 3790-36019
- Avg_Total_Steps 916-16040
- Min_Total_Distance 0-3.64
- Max_Total_Distance 2.62-28.03
- Avg_Total_Distance 0.63-13.21
- Min_Total_Calories 0-1976
- Max_Total_Calories 1760-4900
- Avg_Total_Calories1483-3436
- Min_Very_Active_Minutes 0
- Max_Very_Active_Minutes 2-210
- Avg_Very_Active_Minutes 0.09-87.33
- Min_Fairly_Active_Minutes 0
- Max_Fairly_Active_Minutes 6-143
- Avg_Fairly_Active_Minutes 0.25-61.23
- Min_Lightly_Active_Minutes 0-172
- Max_Lightly_Active_Minutes 164-518
- Avg_Lightly_Active_Minutes 38.58-327.9
- Min_Sedentary_Minutes 0-1193
- Max_Sedentary_Minutes 851-1440
- Avg_Sedentary_Minutes 662.32-1317.41

### 5.3 Average Active Minutes per ID
Next, I focused on calculating the averages of various activity minutes by ID.

```sql
1.	SELECT Id, 
2.	avg(VeryActiveMinutes) AS Avg_Very_Active_Minutes,
3.	avg(FairlyActiveMinutes) AS Avg_Fairly_Active_Minutes,
4.	avg(LightlyActiveMinutes) AS Avg_Lightly_Active_Minutes,
5.	avg(SedentaryMinutes) AS Avg_Sedentary_Minutes,
6.	FROM `my-project-2024-423122.bellabeat.dailyActivity_merged`
7.	GROUP BY Id
```

![6](https://github.com/user-attachments/assets/82dd94f6-481b-48aa-a41f-d26244ff6dfe)

The average number of minutes spent in the Sedentary activity level was the highest for each unique ID.

### 5.4 Average Active Minutes per Weekday
 I wanted to analyze the average number of active minutes per weekday.

 ```sql
1.	SELECT 
2.	  activityday,
3.	  FORMAT_DATE('%A', activityday) AS weekday,
4.	  ROUND(AVG(VeryActiveMinutes)) AS Avg_Very_Active_Minutes,
5.	  ROUND(AVG(FairlyActiveMinutes)) AS Avg_Fairly_Active_Minutes,
6.	  ROUND(AVG(LightlyActiveMinutes)) AS Avg_Lightly_Active_Minutes,
7.	  ROUND(AVG(SedentaryMinutes)) AS Avg_Sedentary_Minutes
8.	FROM  
9.	  `my-project-2024-423122.bellabeat.dailyActivity_merged`
10.	GROUP BY 
11.	  activityday, weekday
12.	ORDER BY 
13.	  Activityday
```

![7](https://github.com/user-attachments/assets/4f57fd3f-72e5-4e0a-9906-ba1eeaacf0f1)

The findings indicate that Sedentary Minutes account for the highest number of active minutes. Notably, there isn’t much variation in the types of active minutes across different weekdays; users tend to accumulate a consistent amount of active minutes each day.
This suggests that Bellabeat has an opportunity to help users achieve their activity goals. Since users appear to be striving to meet their daily activity targets, Bellabeat could encourage them to set higher goals, potentially increasing the number of very active and fairly active minutes they achieve each day.


### 5.5 ID Compliance with CDC Recommended Criteria
The CDC recommends substantial health benefits, adults should do at least 150 minutes (2 hours and 30 minutes) to 300 minutes (5 hours) a week of moderate-intensity. [Physical Activity Guidelines for Americans, 2nd edition](https://odphp.health.gov/sites/default/files/2019-09/Physical_Activity_Guidelines_2nd_edition.pdf)
So, I wanted to add up the sum minutes of Very Active, Fairly Active for a week to see if each unique ID was meeting the CDC’s guidelines for activity.

 ```sql
1.	SELECT Id, 
2.	SUM(VeryActiveMinutes + FairlyActiveMinutes) AS Total_Avg_Active_Minutes,
3.	CASE 
4.	WHEN SUM(VeryActiveMinutes + FairlyActiveMinutes) >= 150 THEN 'Meets CDC Recommendation'
5.	WHEN SUM(VeryActiveMinutes + FairlyActiveMinutes) <150 THEN 'Does Not Meet CDC Recommendation'
6.	END CDC_Recommendations
7.	FROM `my-project-2024-423122.bellabeat.dailyActivity_merged`
8.	WHERE ActivityDay BETWEEN '2016-04-17' AND '2016-04-23'
9.	GROUP BY Id
```

![8](https://github.com/user-attachments/assets/337aa923-3020-4c6a-9311-355bdd84e31f)

Out of 33 users, 18 met the CDC Recommendations, 14 did not, and 1 did not have data from this time period.

### 5.6 Active vs. Inactive Based on Average Steps
The baseline number of steps per day has varied across studies but the typical amount is about 5,000 steps a day. It is estimated that 80 percent of daily steps among less active people are light intensity. Most research studies designed to increase physical activity have focused on increasing both the amount and intensity of physical activity above basic movement from daily life activities. 
Studies that focus on steps often set targets of 10,000 steps a day or a percentage increase in steps a day to encourage people to increase their amount of moderate-to-vigorous physical activity. [Physical Activity Guidelines for Americans, 2nd edition](https://odphp.health.gov/sites/default/files/2019-09/Physical_Activity_Guidelines_2nd_edition.pdf)

```sql
1.	SELECT Id,
2.	avg(TotalSteps) AS Avg_Total_Steps,
3.	CASE
4.	WHEN avg(TotalSteps) <= 5000 THEN 'Inactive'
5.	WHEN avg(TotalSteps) > 5000 THEN 'Active User'
6.	END User_Type
7.	FROM `my-project-2024-423122.bellabeat.dailyActivity_merged`
8.	GROUP BY Id
```

![9](https://github.com/user-attachments/assets/3e443c57-32c9-42e7-a8bb-f87b0173c4d7)

25 users are active users and only 8 are inactive users.

### 5.7 Active Minutes, Calories Burned, and Steps Logged
I am interested to see what their logged calories tell us about how many steps they take and how long they are active.

```sql
1.	SELECT Id, 
2.	Sum(TotalSteps) AS Sum_total_steps,
3.	SUM(Calories) AS Sum_Calories, 
4.	SUM(VeryActiveMinutes + FairlyActiveMinutes) AS Sum_Active_Minutes
5.	FROM `my-project-2024-423122.bellabeat.dailyActivity_merged`
6.	GROUP BY Id
```

![10](https://github.com/user-attachments/assets/05fd4f58-9901-419c-976e-d45561042f3b)

This graph illustrates the relationship between three metrics: total calories burned, total active minutes, and total steps taken by individual users. The data shows that these metrics generally follow the same directional trend, indicating a strong correlation between physical activity, calorie expenditure, and step count.
As active minutes increase, there is a corresponding rise in both calories burned and steps taken, highlighting the interdependence of physical activity intensity and its outcomes. This pattern is consistent across users, with some variations and outliers that may warrant further exploration.

### 5.8 Average Steps by Day  
Next, I wanted to analyze the average number of steps taken each day to determine if certain days of the week are busier than others.

```sql
1.	SELECT 
2.	activityday,
3.	FORMAT_DATE('%A', activityday) AS weekday,
4.	ROUND (avg(TotalSteps), 2) AS Average_Total_Steps,
5.	FROM  
6.	`my-project-2024-423122.bellabeat.dailyActivity_merged`
7.	GROUP BY 
8.	activityday, weekday
9.	ORDER BY 
10.	Activityday
```

![11](https://github.com/user-attachments/assets/ddca68f4-4ddb-47c4-a8ff-6cdbfe2f1362)

The activity levels are relatively consistent throughout the week, with higher activity on Tuesdays, Wednesdays, Thursdays, and Saturdays, and lower activity on Mondays, Fridays, and Sundays. This suggests a routine with similar activity levels most days, with potential for relaxation or leisure on weekends.

### 5.9 Total Steps by Hour
Next, I want to analyze total steps by hour to identify when our users are most active.

```sql
1.	SELECT
2.	  EXTRACT(HOUR FROM Activity) AS hour,
3.	  SUM(StepTotal) AS Total_Steps_By_Hour
4.	FROM
5.	  `my-project-2024-423122.bellabeat.hourlySteps_merged`
6.	GROUP BY
7.	  hour
8.	ORDER BY
9.	  Total_Steps_By_Hour DESC;
```

![12](https://github.com/user-attachments/assets/7ab13177-0322-430f-8259-125696d6a5ff)

The top 5 hours of steps recorded were:

 - 18:00:00 (6pm) — 542,848 steps
 - 19:00:00 (7pm) — 528,552 steps
 - 12:00:00 (12pm) — 505,848 steps
 - 17:00:00 (5pm) — 498,511 steps
 - 14:00:00 (2pm) — 497,813 steps

### 5.10 Average Calories by Day 

```sql
1.	SELECT 
2.	activityday,
3.	FORMAT_DATE('%A', activityday) AS weekday,
4.	ROUND (avg(Calories), 2) AS Average_Total_Calories,
5.	FROM  
6.	`my-project-2024-423122.bellabeat.dailyActivity_merged`
7.	GROUP BY 
8.	activityday, weekday
9.	ORDER BY 
10.	Activityday
```

The activity levels are relatively consistent throughout the week, with higher activity on Tuesdays, which is the day that have spent more calories

![13](https://github.com/user-attachments/assets/01a9d4fb-8312-4587-a3e0-592e3c87c563)

### 5.11 Total calories by Hour

```sql
1.	SELECT
2.	EXTRACT(HOUR FROM Activity) AS hour,
3.	SUM(Calories) AS Total_Calories_By_Hour
4.	FROM
5.	`my-project-2024-423122.bellabeat.hourlyCalories_merged`
6.	GROUP BY
7.	hour
8.	ORDER BY
9.	Total_Calories_By_Hour DESC;
```

![14](https://github.com/user-attachments/assets/33f03d1e-c233-4a1e-82e1-6777035e109f)

The top 5 hours of calories recorded were:

 - 18:00:00 (6pm) — 111884
 - 17:00:00 (7pm) — 111214
 - 19:00:00 (12pm) — 110065
 - 12:00:00 (5pm) — 108056
 - 14:00:00 (2pm) — 106590

![15](https://github.com/user-attachments/assets/84aeac4a-ad33-4a89-bd83-4d71d8fb7873)

This image consists of four visualizations summarizing data on calories burned and steps taken:

1. **Top-Left: Average Calories by Day**  
   - Displays the average calories burned each day of the week.  
   - The highest average calories burned are on Tuesday (2,352.9), while the lowest are on Thursday (2,129.7).

2. **Top-Right: Average Steps by Weekday**  
   - Highlights the average steps taken on each weekday.  
   - Tuesday has the highest average steps (8,126), while Sunday has the lowest (6,937).

3. **Bottom-Left: Total Steps by Hour**  
   - Line chart showing the total steps by hour of the day.  
   - Peaks are observed around midday (12 PM) and early evening (5–7 PM).

4. **Bottom-Right: Total Calories by Hour**  
   - Line chart showing the total calories burned by hour of the day.  
   - Calories burned gradually increase through the day, peaking between 5–7 PM.

These visualizations provide insights into daily and hourly trends in activity and calorie expenditure.

### 5.12 Correlation of Steps with Active Minutes

```sql
1.	WITH activity_data AS (
2.	  SELECT 1000 AS total_steps, 20 AS VeryActiveMinutes
3.	 UNION ALL
4.	  SELECT 1500, 25 UNION ALL
5.	  SELECT 2000, 30 UNION ALL
6.	  SELECT 2500, 35 UNION ALL
7.	  SELECT 3000, 40
8.	)
9.	
10.	SELECT 
11.	  CORR(total_steps,VeryActiveMinutes) AS correlation_coefficient
12.	FROM 
13.	  activity_data
```
**1** indicates a perfect positive correlation (as one increases, so does the other






























  



