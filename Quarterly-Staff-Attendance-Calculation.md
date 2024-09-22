~~~ SQL
/*CTE with Attendance Data Calculations*/
WITH t1  AS
(SELECT -- Attendance Data Calculation
  Position_ID,
  First_Name,
  Last_Name,
  In_Time,
  Out_Time,
  Hours,
  Pay_Code,
  EXTRACT(MONTH FROM In_Time) AS Month, -- Numeric Month Columns
  CASE -- School Days
    WHEN EXTRACT(MONTH FROM In_Time) = 8 THEN 10
    END AS School_Days,
  CASE -- Staff in Service Days
    WHEN EXTRACT(MONTH FROM In_Time) = 8 THEN 10
    END AS Staff_In_Service_Days,
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

t2 AS (SELECT -- 11 MTH Staff Attendance Data Calculation
 sub.Position_ID,
  sub.first_name,
  sub.last_name,
  sub.Campus,
  sub.Month,
  sub.school_days - (sub.Excused_Abs_Calc_11MTH + sub.Unexcused_Abs_Calc_11MTH) AS Days_Present,
  sub.school_days - sub.Excused_Abs_Calc_11MTH AS Possible_Days,
  ROUND((sub.school_days - (sub.Excused_Abs_Calc_11MTH + sub.Unexcused_Abs_Calc_11MTH))/(sub.school_days - sub.Excused_Abs_Calc_11MTH),2) AS Percentage,
  sub.School_Days,
  sub.Staff_In_Service_Days,
  sub.Contract_Type
FROM
(SELECT 
  DISTINCT Position_ID,
  First_Name,
  Last_Name,
  Month,
  School_Days,
  Staff_In_Service_Days,
  SUM(Excused_Abs_12MTH * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc_12MTH,
  SUM(Excused_Abs_11MTH * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc_11MTH,
  SUM(Unexcused_Abs_12MTH * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc_12MTH,
  SUM(Unexcused_Abs_11MTH * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc_11MTH,
  Campus,
  Contract_Type
FROM t1
WHERE Contract_Type = '11 MTH') AS sub),

t3 AS (SELECT -- 12 MTH Staff Attendance Calculation
  sub.Position_ID,
  sub.first_name,
  sub.last_name,
  sub.Campus,
  sub.Month,
  sub.school_days - (sub.Excused_Abs_Calc_12MTH + sub.Unexcused_Abs_Calc_12MTH) AS Days_Present,
  sub.school_days - sub.Excused_Abs_Calc_12MTH AS Possible_Days,
  ROUND((sub.school_days - (sub.Excused_Abs_Calc_12MTH + sub.Unexcused_Abs_Calc_12MTH))/(sub.school_days - sub.Excused_Abs_Calc_12MTH),2) AS Percentage,
  sub.School_Days,
  sub.Staff_In_Service_Days,
  sub.Contract_Type
FROM
(SELECT 
  DISTINCT Position_ID,
  First_Name,
  Last_Name,
  Month,
  School_Days,
  Staff_In_Service_Days,
  SUM(Excused_Abs_12MTH * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc_12MTH,
  SUM(Excused_Abs_11MTH * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc_11MTH,
  SUM(Unexcused_Abs_12MTH * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc_12MTH,
  SUM(Unexcused_Abs_11MTH * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc_11MTH,
  Campus,
  Contract_Type
FROM t1
WHERE Contract_Type = '12 MTH') AS sub)


/*Combning 11 MTH and 12 MTH Attedance Data Calculations*/
SELECT * -- Combining Outputs
FROM t2
UNION ALL
SELECT *
FROM t3
~~~~
