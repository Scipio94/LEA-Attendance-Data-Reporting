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
  CASE -- School Days
    WHEN EXTRACT(MONTH FROM In_Time) = 8 THEN 9
    WHEN EXTRACT(MONTH FROM In_Time) = 9 THEN 19
    WHEN EXTRACT(MONTH FROM In_Time) = 10 THEN 21
    WHEN EXTRACT(MONTH FROM In_Time) = 11 THEN 16
    WHEN EXTRACT(MONTH FROM In_Time) = 12 THEN 16 
    WHEN EXTRACT(MONTH FROM In_Time) = 1 THEN 21 
    WHEN EXTRACT(MONTH FROM In_Time) = 2 THEN 17
    END AS School_Days,
  COUNT(CASE WHEN Pay_Code IN ('ABSENT','SICK','UNPAID TIME OFF','PERSONAL','LONGEVITY DAYS') THEN 'Unexcused' END) AS Unexcused_Abs, -- Unexcused Absence
  COUNT(CASE WHEN Pay_Code IN ('COVID SICK','JURY','PROFESSIONAL DEVELOPMENT','RELIGIOUS OBSERVATION','VACATION') THEN 'Excused' END) AS Excused_Abs, -- Excused Absence
  CASE -- Half or Whole Days
    WHEN Hours <= 4 THEN 0.5
    WHEN HOURS >= 5 THEN 1
    WHEN Hours = 0 THEN 0
    END AS Days, 
Campus,
Contract_Type
FROM `my-data-project-36654.Quarterly_Attendance.Staff_Att_08_23_02_24`
WHERE Pay_Code != 'REGSAL' AND Contract_Type != '11 MTH' -- Filtering out pay days
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
  EXTRACT(MONTH FROM In_Time) AS Month, -- Numeric Month Columns
  CASE -- School Days
    WHEN EXTRACT(MONTH FROM In_Time) = 8 THEN 9
    WHEN EXTRACT(MONTH FROM In_Time) = 9 THEN 19
    WHEN EXTRACT(MONTH FROM In_Time) = 10 THEN 21
    WHEN EXTRACT(MONTH FROM In_Time) = 11 THEN 16
    WHEN EXTRACT(MONTH FROM In_Time) = 12 THEN 16 
    WHEN EXTRACT(MONTH FROM In_Time) = 1 THEN 21 
    WHEN EXTRACT(MONTH FROM In_Time) = 2 THEN 17
    END AS School_Days,
  COUNT(CASE WHEN Pay_Code IN ('ABSENT','SICK','UNPAID TIME OFF','PERSONAL','LONGEVITY DAYS','VACATION') THEN 'Unexcused' END) AS Unexcused_Abs, -- Unexcused Absence
  COUNT(CASE WHEN Pay_Code IN ('COVID SICK','JURY','PROFESSIONAL DEVELOPMENT','RELIGIOUS OBSERVATION') THEN 'Excused' END) AS Excused_Abs, -- Excused Absence
  CASE -- Half or Whole Days
    WHEN Hours <= 4 THEN 0.5
    WHEN HOURS >= 5 THEN 1
    WHEN Hours = 0 THEN 0
    END AS Days, 
Campus,
Contract_Type
FROM `my-data-project-36654.Quarterly_Attendance.Staff_Att_08_23_02_24`
WHERE Pay_Code != 'REGSAL' AND Contract_Type != '12 MTH' -- Filtering out pay days
GROUP BY First_Name,Last_Name,In_time,Out_time,Hours,Pay_Code, Campus,Contract_Type, Position_ID;
  
/*Attendance Calculation tw_mth*/
SELECT
  DISTINCT sub.Position_ID,
  sub.first_name,
  sub.last_name,
  sub.Campus,
  sub.Month,
  sub.school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc) AS Days_Present,
  sub.school_days - sub.Excused_Abs_Calc AS Possible_Days,
  ROUND((sub.school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc))/(sub.school_days - sub.Excused_Abs_Calc),2) AS Percentage,
  sub.School_Days,
  sub.Contract_Type
FROM
(SELECT 
  Position_ID,
  First_Name,
  Last_Name,
  Month,
  School_Days,
  SUM(Excused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc,
  SUM(Unexcused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc,
  Campus,
  Contract_Type
FROM tw_mth) AS sub
ORDER BY sub.First_Name, sub.Month;


/*Attendance Calculation el_mth*/
SELECT
  DISTINCT sub.Position_ID,
  sub.first_name,
  sub.last_name,
  sub.Campus,
  sub.Month,
  sub.school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc) AS Days_Present,
  sub.school_days - sub.Excused_Abs_Calc AS Possible_Days,
  ROUND((sub.school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc))/(sub.school_days - sub.Excused_Abs_Calc),2) AS Percentage,
  sub.School_Days,
  sub.Contract_Type
FROM
(SELECT 
  Position_ID,
  First_Name,
  Last_Name,
  Month,
  School_Days,
  SUM(Excused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc,
  SUM(Unexcused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc,
  Campus,
  Contract_Type
FROM el_mth) AS sub
ORDER BY sub.First_Name, sub.Month;

/*Combining Outputs*/

/*Attendance Calculation tw_mth*/
SELECT
  DISTINCT sub.Position_ID,
  sub.first_name,
  sub.last_name,
  sub.Campus,
  sub.Month,
  sub.school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc) AS Days_Present,
  sub.school_days - sub.Excused_Abs_Calc AS Possible_Days,
  ROUND((sub.school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc))/(sub.school_days - sub.Excused_Abs_Calc),2) AS Percentage,
  sub.School_Days,
  sub.Contract_Type
FROM
(SELECT 
  Position_ID,
  First_Name,
  Last_Name,
  Month,
  School_Days,
  SUM(Excused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc,
  SUM(Unexcused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc,
  Campus,
  Contract_Type
FROM tw_mth) AS sub
UNION ALL -- Combining outputs
/*Attendance Calculation el_mth*/
SELECT
  DISTINCT sub.Position_ID,
  sub.first_name,
  sub.last_name,
  sub.Campus,
  sub.Month,
  sub.school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc) AS Days_Present,
  sub.school_days - sub.Excused_Abs_Calc AS Possible_Days,
  ROUND((sub.school_days - (sub.Excused_Abs_Calc + sub.Unexcused_Abs_Calc))/(sub.school_days - sub.Excused_Abs_Calc),2) AS Percentage,
  sub.School_Days,
  sub.Contract_Type
FROM
(SELECT 
  Position_ID,
  First_Name,
  Last_Name,
  Month,
  School_Days,
  SUM(Excused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID ) AS Excused_Abs_Calc,
  SUM(Unexcused_Abs * Days) OVER (PARTITION BY First_Name,Last_Name,Month,School_Days,Campus,Contract_Type,Position_ID) AS Unexcused_Abs_Calc,
  Campus,
  Contract_Type
FROM el_mth) AS sub;
~~~~
