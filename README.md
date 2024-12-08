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

















  



