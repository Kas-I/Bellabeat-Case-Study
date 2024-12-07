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
-- The WITH clause creates a Common Table Expression (CTE) named combined_data.
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
SELECTthis image 
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
COUNT(*) <> 4;'''

This method involves creating a CTE to combine the four tables and then checking for inconsistencies.

![1](https://github.com/user-attachments/assets/a0433e13-df0d-4867-8474-4de08cc59666)








  



