~~~ SQL
/*Temp Table for 12 MTH employees*/
CREATE TEMP TABLE tw_mth  AS
SELECT 
  Position_ID,
  First_Name,
  Last_Name,
  In_Time,
  Out_Time,
  Hours,
  Pay_Code,
  EXTRACT(MONTH FROM In_Time) AS Month, -- Numeric Month Columns
  CASE 
    WHEN In_Time >= '2024-08-19' AND  In_Time <='2024-08-23'THEN 1
    WHEN In_Time >= '2024-08-26' AND  In_time <= '2024-08-30' THEN 2
    WHEN In_Time >= '2024-09-03' AND In_Time <= '2024-09-06' THEN 3

    END AS Week,
  CASE 
    WHEN In_Time >= '2024-08-19' AND  In_Time <='2024-08-23' THEN 5
    WHEN In_Time >= '2024-08-26' AND  In_time <= '2024-08-30' THEN 5
    WHEN In_Time >= '2024-09-03' AND In_Time <= '2024-09-06' THEN 4
    END AS Week_School_Days,
  COUNT(CASE WHEN Pay_Code IN ('ABSENT','SICK','UNPAID TIME OFF','PERSONAL','LONGEVITY DAYS') THEN 'Unexcused' END) AS Unexcused_Abs, -- Unexcused Absence
  COUNT(CASE WHEN Pay_Code IN ('COVID SICK','JURY','PROFESSIONAL DEVELOPMENT','RELIGIOUS OBSERVATION','VACATION','WC') THEN 'Excused' END) AS Excused_Abs, -- Excused Absence
  CASE -- Half or Whole Days
    WHEN Hours <= 4 THEN 0.5
    WHEN HOURS >= 5 THEN 1
    WHEN Hours = 0 THEN 0
    END AS Days, 
Campus,
Contract_Type
FROM `my-data-project-36654.Quarterly_Attendance.Att_08_24`
WHERE Contract_Type = '12 MTH' -- Filtering Contract Type and School Days
GROUP BY Position_ID,First_Name,Last_Name,In_time,Out_time,Hours,Pay_Code, Campus,Contract_Type, Position_ID;

/*Temp Table for 11 MTH employees*/ -- VACATION is Unexcused
CREATE TEMP TABLE el_mth  AS
SELECT 
  Position_ID,
  First_Name,
  Last_Name,
  In_Time,
  Out_Time,
  Hours,
  Pay_Code,
   CASE 
    WHEN In_Time BETWEEN '2024-08-19' AND '2024-08-23'THEN 1
    WHEN In_Time BETWEEN '2024-08-26' AND '2024-08-30' THEN 2
    END AS Week,
  CASE 
    WHEN In_Time BETWEEN '2024-08-19' AND '2024-08-23'THEN 5
    WHEN In_Time BETWEEN '2024-08-26' AND '2024-08-30' THEN 5
    END AS Week_School_Days,
  EXTRACT(MONTH FROM In_Time) AS Month, -- Numeric Month Columns
  COUNT(CASE WHEN Pay_Code IN ('ABSENT','SICK','UNPAID TIME OFF','PERSONAL','LONGEVITY DAYS','VACATION') THEN 'Unexcused' END) AS Unexcused_Abs, -- Unexcused Absence
  COUNT(CASE WHEN Pay_Code IN ('COVID SICK','JURY','PROFESSIONAL DEVELOPMENT','RELIGIOUS OBSERVATION','WC') THEN 'Excused' END) AS Excused_Abs, -- Excused Absence
  CASE -- Half or Whole Days
    WHEN Hours <= 4 THEN 0.5
    WHEN HOURS >= 5 THEN 1
    WHEN Hours = 0 THEN 0
    END AS Days, 
Campus,
Contract_Type
FROM `my-data-project-36654.Quarterly_Attendance.Att_08_24`
WHERE Contract_Type = '11 MTH' -- Filtering Contract Type
GROUP BY First_Name,Last_Name,In_time,Out_time,Hours,Pay_Code, Campus,Contract_Type, Position_ID;

/*Attendance Calculation tw_mth*/
SELECT --12 Month Weekly Attendance Calculation
sub1.first_name,
sub1.last_name,
sub1.Campus,
sub1.Month,
sub1.Week,
sub1.Days_Present,
sub1.Possible_Days,
ROUND(sub1.Days_Present/sub1.Possible_Days,2) AS Percentage, 
sub1.Week_School_days,
sub1.Contract_Type
FROM
(SELECT
  DISTINCT sub.Position_ID,
  sub.first_name,
  sub.last_name,
  sub.Campus,
  sub.Month,
  sub.Week,
  sub.Week_school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc) AS Days_Present,
  CASE 
    WHEN sub.Week_school_days - sub.Excused_Abs_Calc = 0 THEN 1
    ELSE sub.Week_school_days - sub.Excused_Abs_Calc END AS Possible_Days, -- Replacing 0 possible days w/ 1 for dvision
  sub.Week_school_days,
  sub.Contract_Type
FROM
(
  SELECT 
    Position_ID,
    First_Name,
    Last_Name,
    Month,
    Week,
    Week_School_Days,
    SUM(Excused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,Week,Week_School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc,
    SUM(Unexcused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,Week,Week_School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc,
    Campus,
    Contract_Type
FROM tw_mth
WHERE Week IS NOT NULL) AS sub -- Filtering out non-school days
ORDER BY sub.First_Name, sub.Month, sub.Week) AS sub1
WHERE sub1.First_Name = 'Tyrone';

/*Attendance Calculation el_mth*/
SELECT -- 11 Month Weekly Attendance Calculation
sub1.first_name,
sub1.last_name,
sub1.Campus,
sub1.Month,
sub1.Week,
sub1.Days_Present,
sub1.Possible_Days,
ROUND(sub1.Days_Present/sub1.Possible_Days,2) AS Percentage, 
sub1.Week_School_days,
sub1.Contract_Type
FROM
(SELECT
  DISTINCT sub.Position_ID,
  sub.first_name,
  sub.last_name,
  sub.Campus,
  sub.Month,
  sub.Week,
  sub.Week_school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc) AS Days_Present,
  CASE 
    WHEN sub.Week_school_days - sub.Excused_Abs_Calc = 0 THEN 1
    ELSE sub.Week_school_days - sub.Excused_Abs_Calc END AS Possible_Days, -- Replacing 0 possible days w/ 1 for dvision
  sub.Week_school_days,
  sub.Contract_Type
FROM
(
  SELECT 
    Position_ID,
    First_Name,
    Last_Name,
    Month,
    Week,
    Week_School_Days,
    SUM(Excused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,Week,Week_School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc,
    SUM(Unexcused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,Week,Week_School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc,
    Campus,
    Contract_Type
FROM el_mth
WHERE Week IS NOT NULL) AS sub -- Filtering out non-school days
ORDER BY sub.First_Name, sub.Month, sub.Week) AS sub1;

/*Combining Outputs*/
SELECT -- COMBINED OUTPUTS
  DISTINCT Month,
  Week,
  ROUND(AVG(Percentage) OVER (PARTITION BY Month,Week),2) AS Overall_Pct
FROM -- subquery 
(/*Attendance Calculation tw_mth*/
SELECT -- Combined Output
sub1.Month,
sub1.Week,
AVG(ROUND(sub1.Days_Present/sub1.Possible_Days,2)) OVER (PARTITION BY sub1.Month, sub1.Week) AS Percentage
FROM
(SELECT
  DISTINCT sub.Position_ID,
  sub.first_name,
  sub.last_name,
  sub.Campus,
  sub.Month,
  sub.Week,
  sub.Week_school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc) AS Days_Present,
  CASE 
    WHEN sub.Week_school_days - sub.Excused_Abs_Calc = 0 THEN 1
    ELSE sub.Week_school_days - sub.Excused_Abs_Calc END AS Possible_Days, -- Replacing 0 possible days w/ 1 for dvision
  sub.Week_school_days,
  sub.Contract_Type
FROM
(
  SELECT 
    Position_ID,
    First_Name,
    Last_Name,
    Month,
    Week,
    Week_School_Days,
    SUM(Excused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,Week,Week_School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc,
    SUM(Unexcused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,Week,Week_School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc,
    Campus,
    Contract_Type
FROM tw_mth
WHERE Week IS NOT NULL) AS sub -- Filtering out non-school days
ORDER BY sub.First_Name, sub.Month, sub.Week) AS sub1
UNION ALL -- Combining outputs
/*Attendance Calculation el_mth*/
SELECT
sub1.Month,
sub1.Week,
AVG(ROUND(sub1.Days_Present/sub1.Possible_Days,2)) OVER (PARTITION BY sub1.Month, sub1.Week) AS Percentage
FROM
(SELECT
  DISTINCT sub.Position_ID,
  sub.first_name,
  sub.last_name,
  sub.Campus,
  sub.Month,
  sub.Week,
  sub.Week_school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc) AS Days_Present,
  CASE 
    WHEN sub.Week_school_days - sub.Excused_Abs_Calc = 0 THEN 1
    ELSE sub.Week_school_days - sub.Excused_Abs_Calc END AS Possible_Days, -- Replacing 0 possible days w/ 1 for dvision
  sub.Week_school_days,
  sub.Contract_Type
FROM
(
  SELECT 
    Position_ID,
    First_Name,
    Last_Name,
    Month,
    Week,
    Week_School_Days,
    SUM(Excused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,Week,Week_School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc,
    SUM(Unexcused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,Week,Week_School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc,
    Campus,
    Contract_Type
FROM el_mth
WHERE Week IS NOT NULL) AS sub -- Filtering out non-school days
ORDER BY sub.First_Name, sub.Month, sub.Week) AS sub1) -- subquery
ORDER BY Week
~~~
