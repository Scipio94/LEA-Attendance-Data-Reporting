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
  END AS Week,
   CASE 
    WHEN In_Time >= '2024-08-19' AND  In_Time <='2024-08-23' THEN 5
    WHEN In_Time >= '2024-08-26' AND  In_time <= '2024-08-30' THEN 5
    WHEN In_Time >= '2024-09-03' AND In_Time <= '2024-09-06' THEN 4
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
FROM `my-data-project-36654.Quarterly_Attendance.Att_08_24`
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

/*Combning 11 MTH and 12 MTH Weekly Attedance Data Calculations*/
SELECT -- Weekly Attendance Percentage Agg
  sub.Week,
  ROUND(AVG(sub.Percentage),2) AS Week_Att_Percentage 
FROM
(SELECT * 
FROM t2
UNION ALL
SELECT *
FROM t3) AS sub
GROUP BY sub.Week
ORDER BY sub.Week
~~~
