~~~ SQL
WITH t1 AS
(SELECT -- Attendance Data Calculation
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
    WHEN In_Time >= '2024-09-09' AND In_Time <= '2024-09-13' THEN 4
    WHEN In_Time >= '2024-09-16' AND In_Time <= '2024-09-20' THEN 5
    WHEN In_Time >= '2024-09-23' AND In_Time <= '2024-09-27' THEN 6
    WHEN In_Time >= '2024-09-30' AND In_Time <= '2024-10-04' THEN 7
    WHEN In_Time >= '2024-10-07' AND In_Time <= '2024-10-11' THEN 8
    WHEN In_Time >= '2024-10-14' AND In_Time <= '2024-10-18' THEN 9
    WHEN In_Time >= '2024-10-21' AND In_Time <= '2024-10-25' THEN 10
    WHEN In_Time >= '2024-10-28' AND In_Time <= '2024-11-01' THEN 11
    WHEN In_Time >= '2024-11-04' AND In_Time <= '2024-11-08' THEN 12
    WHEN In_Time >= '2024-11-11' AND In_Time <= '2024-11-15' THEN 13
    WHEN In_Time >= '2024-11-18' AND In_Time <= '2024-11-22' THEN 14
    WHEN In_Time >= '2024-12-02' AND In_Time <= '2024-12-06' THEN 16
    WHEN In_Time >= '2024-12-09' AND In_Time <= '2024-12-13' THEN 17
    WHEN In_Time >= '2024-12-16' AND In_Time <= '2024-12-20' THEN 18
    WHEN In_Time >= '2025-01-06' AND In_Time <= '2025-01-10' THEN 19
    WHEN In_Time >= '2025-01-13' AND In_Time <= '2025-01-17' THEN 20
    WHEN In_Time >= '2025-01-20' AND In_Time <= '2025-01-24' THEN 21
    WHEN In_Time >= '2025-01-27' AND In_Time <= '2025-01-31' THEN 22
    WHEN In_Time >= '2025-02-03' AND In_Time <= '2025-02-07' THEN 23
    WHEN In_Time >= '2025-02-10' AND In_Time <= '2025-02-14' THEN 24
    WHEN In_Time >= '2025-02-17' AND In_Time <= '2025-02-21' THEN 25
    WHEN In_Time >= '2025-02-24' AND In_Time <= '2025-02-28' THEN 26
    WHEN In_Time >= '2025-03-03' AND In_Time <= '2025-03-07' THEN 27
    WHEN In_Time >= '2025-03-10' AND In_Time <= '2025-03-14' THEN 28
    WHEN In_Time >= '2025-03-17' AND In_Time <= '2025-03-21' THEN 29
    WHEN In_Time >= '2025-03-24' AND In_Time <= '2025-03-28' THEN 30
    WHEN In_Time >= '2025-03-31' AND In_Time <= '2025-04-04' THEN 31
    WHEN In_Time >= '2025-04-07' AND In_Time <= '2025-04-11' THEN 32
    WHEN In_Time >= '2025-04-14' AND In_Time <= '2025-04-18' THEN 33
  END AS Week,
   CASE 
    WHEN In_Time >= '2024-08-19' AND  In_Time <='2024-08-23' THEN 5
    WHEN In_Time >= '2024-08-26' AND  In_time <= '2024-08-30' THEN 5
    WHEN In_Time >= '2024-09-03' AND In_Time <= '2024-09-06' THEN 4
    WHEN In_Time >= '2024-09-09' AND In_Time <= '2024-09-13' THEN 5
    WHEN In_Time >= '2024-09-16' AND In_Time <= '2024-09-20' THEN 5
    WHEN In_Time >= '2024-09-23' AND In_Time <= '2024-09-27' THEN 5
    WHEN In_Time >= '2024-09-30' AND In_Time <= '2024-10-04' THEN 5
    WHEN In_Time >= '2024-10-07' AND In_Time <= '2024-10-11' THEN 5
    WHEN In_Time >= '2024-10-14' AND In_Time <= '2024-10-18' THEN 4
    WHEN In_Time >= '2024-10-21' AND In_Time <= '2024-10-25' THEN 5
    WHEN In_Time >= '2024-10-28' AND In_Time <= '2024-11-01' THEN 5
    WHEN In_Time >= '2024-11-04' AND In_Time <= '2024-11-08' THEN 5
    WHEN In_Time >= '2024-11-11' AND In_Time <= '2024-11-15' THEN 5
    WHEN In_Time >= '2024-11-18' AND In_Time <= '2024-11-22' THEN 5
    WHEN In_Time >= '2024-12-02' AND In_Time <= '2024-12-06' THEN 5
    WHEN In_Time >= '2024-12-09' AND In_Time <= '2024-12-13' THEN 5
    WHEN In_Time >= '2024-12-16' AND In_Time <= '2024-12-20' THEN 5
    WHEN In_Time >= '2025-01-06' AND In_Time <= '2025-01-10' THEN 5
    WHEN In_Time >= '2025-01-13' AND In_Time <= '2025-01-17' THEN 5
    WHEN In_Time >= '2025-01-20' AND In_Time <= '2025-01-24' THEN 4
    WHEN In_Time >= '2025-01-27' AND In_Time <= '2025-01-31' THEN 5
    WHEN In_Time >= '2025-02-03' AND In_Time <= '2025-02-07' THEN 5
    WHEN In_Time >= '2025-02-10' AND In_Time <= '2025-02-14' THEN 4
    WHEN In_Time >= '2025-02-17' AND In_Time <= '2025-02-21' THEN 4
    WHEN In_Time >= '2025-02-24' AND In_Time <= '2025-02-28' THEN 5
    WHEN In_Time >= '2025-03-03' AND In_Time <= '2025-03-07' THEN 5
    WHEN In_Time >= '2025-03-10' AND In_Time <= '2025-03-14' THEN 5
    WHEN In_Time >= '2025-03-17' AND In_Time <= '2025-03-21' THEN 5
    WHEN In_Time >= '2025-03-24' AND In_Time <= '2025-03-28' THEN 4
    WHEN In_Time >= '2025-03-31' AND In_Time <= '2025-04-04' THEN 5
    WHEN In_Time >= '2025-04-07' AND In_Time <= '2025-04-11' THEN 5
    WHEN In_Time >= '2025-04-14' AND In_Time <= '2025-04-18' THEN 4
    END AS Week_School_Days,
  COUNT(CASE WHEN Contract_Type = '12 MTH' AND Pay_Code IN('ABSENT','SICK','UNPAID TIME OFF','PERSONAL','LONGEVITY DAYS') THEN 'Unexcused' END) AS Unexcused_Abs_12MTH,
  COUNT(CASE WHEN Contract_Type = '12 MTH' AND Pay_Code IN ('COVID SICK','JURY','PROFESSIONAL DEVELOPMENT','RELIGIOUS OBSERVATION','VACATION','WC') THEN 'Excused' END) AS Excused_Abs_12MTH,
  COUNT(CASE WHEN Contract_Type = '11 MTH' AND Pay_Code IN ('ABSENT','SICK','UNPAID TIME OFF','PERSONAL','LONGEVITY DAYS','VACATION') THEN 'Unexcused' END) AS Unexcused_Abs_11MTH,
  COUNT(CASE WHEN Contract_Type = '11 MTH' AND Pay_Code IN ('COVID SICK','JURY','PROFESSIONAL DEVELOPMENT','RELIGIOUS OBSERVATION','WC') THEN 'Excused' END) AS Excused_Abs_11MTH,
  CASE
    WHEN Hours <= 4 THEN 0.5
    WHEN HOURS >= 5 THEN 1
    WHEN Hours = 0 THEN 0
    END AS Days, 
Campus,
Contract_Type
FROM `my-data-project-36654.Quarterly_Attendance.Staff_Attendance_Agg`
GROUP BY Position_ID,First_Name,Last_Name,In_time,Out_time,Hours,Pay_Code, Campus,Contract_Type, Position_ID),

t2 AS
(SELECT -- 11 MTH Attendance Calculation
  DISTINCT sub.Position_ID, 
  sub.first_name,
  sub.last_name,
  sub.Campus,
  sub.Month,
  sub.Week,
  sub.Week_school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc) AS Days_Present,
  sub.Week_school_days - sub.Excused_Abs_Calc AS Possible_Days,
  ROUND((sub.Week_school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc))/(CASE WHEN sub.Week_school_days - sub.Excused_Abs_Calc = 0 THEN 1 ELSE sub.Week_school_days - sub.Excused_Abs_Calc END),2) AS Percentage,
  sub.Contract_Type
FROM
(SELECT
    Position_ID,
    First_Name,
    Last_Name,
    Month,
    Week,
    Week_School_Days,
    SUM(Excused_Abs_11MTH * Days) OVER (PARTITION BY First_Name,Last_Name,Month,Week,Week_School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc,
    SUM(Unexcused_Abs_11MTH * Days) OVER (PARTITION BY First_Name,Last_Name,Month,Week,Week_School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc,
    Campus,
    Contract_Type
FROM t1
WHERE Week IS NOT NULL AND Contract_Type = '11 MTH') AS sub),

t3 AS 
(SELECT -- 12 MTH Attendance Calculation
  DISTINCT sub.Position_ID, 
  sub.first_name,
  sub.last_name,
  sub.Campus,
  sub.Month,
  sub.Week,
  sub.Week_school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc) AS Days_Present,
  sub.Week_school_days - sub.Excused_Abs_Calc AS Possible_Days,
  ROUND((sub.Week_school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc))/(CASE WHEN sub.Week_school_days - sub.Excused_Abs_Calc = 0 THEN 1 ELSE sub.Week_school_days - sub.Excused_Abs_Calc END),2) AS Percentage,
  sub.Contract_Type
FROM
(SELECT
    Position_ID,
    First_Name,
    Last_Name,
    Month,
    Week,
    Week_School_Days,
    SUM(Excused_Abs_12MTH * Days) OVER (PARTITION BY First_Name,Last_Name,Month,Week,Week_School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc,
    SUM(Unexcused_Abs_12MTH * Days) OVER (PARTITION BY First_Name,Last_Name,Month,Week,Week_School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc,
    Campus,
    Contract_Type
FROM t1
WHERE Week IS NOT NULL AND Contract_Type = '12 MTH') AS sub)

/*Combining 11 MTH and 12 MTH Weekly Attedance Data Calculations*/
SELECT -- Weekly Attendance Percentage Agg
  sub.Campus,
  sub.Week,
  ROUND(AVG(sub.Percentage),2) AS Week_Att_Percentage 
FROM
(SELECT * 
FROM t2
UNION ALL
SELECT *
FROM t3) AS sub
WHERE sub.Week = 33 -- Filter for week
GROUP BY sub.Campus,sub.Week
ORDER BY sub.Week
~~~
