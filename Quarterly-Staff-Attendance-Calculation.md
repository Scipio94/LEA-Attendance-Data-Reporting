~~~ SQL
CREATE TEMP TABLE t1 AS
SELECT 
  First_Name,
  Last_Name,
  In_Time,
  Out_Time,
  Hours,
  Pay_Code,
  EXTRACT(MONTH FROM In_Time) AS Month, -- Numeric Month Columns
  CASE -- School Days
    WHEN EXTRACT(MONTH FROM In_Time) = 10 THEN 21
    WHEN EXTRACT(MONTH FROM In_Time) = 11 THEN 16
    WHEN EXTRACT(MONTH FROM In_Time) = 12 THEN 16 
    END AS School_Days,
  COUNT(CASE WHEN Pay_Code IN ('ABSENT','SICK','PERSONAL,UNPAID TIME OFF','PERSONAL','LONGEVITY DAYS') THEN 'Unexcused' END) AS Unexcused_Abs, -- Unexcused Absence
  COUNT(CASE WHEN Pay_Code IN ('COVID SICK','HOLIDAY','JURY','PROFESSIONAL DEVELOPMENT','RELIGIOUS OBSERVATION','VACATION') THEN 'Excused' END) AS Excused_Abs, -- Excused Absence
  CASE -- Half or Whole Days
    WHEN Hours <= 4 THEN 0.5
    WHEN HOURS > 5 THEN 1
    WHEN Hours = 0 THEN 0
    END AS Days
FROM `my-data-project-36654.Quarterly_Attendance.Quarterly_Attendance_Oct_Dec`
WHERE Pay_Code != 'REGSAL' -- Filtering our pay days
GROUP BY Last_Name,First_Name,In_time,Out_time,Hours,Pay_Code;
  

/*Attendance Calculation*/
SELECT
  DISTINCT sub.first_name,
  sub.last_name,
  sub.Month,
  c.string_field_3 AS Campus,
  sub.school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc) AS Days_Present,
  sub.school_days - sub.Excused_Abs_Calc AS Possible_Days,
  ROUND((sub.school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc))/(sub.school_days - sub.Excused_Abs_Calc),2) AS Percentage,
  sub.School_Days
FROM
(SELECT 
  First_Name,
  Last_Name,
  Month,
  School_Days,
  SUM(t1.Excused_Abs * t1.Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days ) AS Excused_Abs_Calc,
  SUM(t1.Unexcused_Abs * t1.Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days ) AS Unexcused_Abs_Calc
FROM t1) AS sub
LEFT JOIN `my-data-project-36654.Quarterly_Attendance.Quarterly_Att_Campus` AS c
ON CONCAT(sub.First_Name,'|',Last_Name) = c.string_field_0 -- JOIN to return campus value
ORDER BY sub.first_name,sub.month
~~~~
