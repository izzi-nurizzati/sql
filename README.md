# sql

I have imported the data into SQL Server & I am using SSMS to query it


/*
Total Employees Surveyed - number of employee responded
SQL concepts used - FROM, SELECT, COUNT()
*/
SELECT
	COUNT(EmployeeID) AS TotalEmployeesSurveyed
FROM
	EmployeeData_2023;


/*
Overall Engagement Score - average of engagement responses
SQL concepts used - FROM, SELECT, CAST(), AVG()
*/
SELECT
	CAST(AVG(CAST(LikertScore AS DECIMAL(10,2))) / 5 * 100 AS DECIMAL (3,0)) AS OverallEngagementScore
FROM
	SurveyResponse_2023;


/*
Participation Rate - employees who completed the survey
SQL concepts used - FROM, LEFT JOIN, SELECT, CAST(), COUNT(), CASE
*/
SELECT
	CAST(COUNT(CASE WHEN sr.EmployeeID IS NOT NULL THEN 1 END) * 100 / COUNT(*) AS DECIMAL(3,0)) AS ParticipationRate
FROM
	EmployeeData_2023 AS ed
LEFT JOIN
    SurveyResponse_2023 AS sr
ON
	ed.EmployeeID = sr.EmployeeID;


/*
Headcount by Gender - count employees grouped by gender
SQL concepts used - FROM, GROUP BY, SELECT, COUNT()
*/
SELECT
	Gender,
	COUNT(EmployeeID) AS Headcount
FROM
	EmployeeData_2023
GROUP BY
	Gender;


/*
Engagement by Gender - average of engagement responses by gender
SQL concepts used: FROM, INNER JOIN, GROUP BY, SELECT, CAST(), AVG()
*/
SELECT
	ed.Gender,
	CAST(AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 AS DECIMAL (3,0)) AS EngagementScore
FROM
	EmployeeData_2023 AS ed
INNER JOIN
	SurveyResponse_2023 AS sr
ON
	ed.EmployeeID = sr.EmployeeID
GROUP BY
	ed.Gender;


/*
Headcount by Division sorted in DESC order
SQL concepts used - FROM, GROUP BY, SELECT, COUNT(), ORDER BY
*/
SELECT
	DivisionName,
	COUNT(EmployeeID) AS Headcount
FROM
	EmployeeData_2023
GROUP BY
	DivisionName
ORDER BY
	DivisionName DESC;


/*
Engagement by Division, sorted by Engagement Score in DESC order
SQL concepts used - FROM, INNER JOIN, GROUP BY, SELECT, CAST(), AVG(), ORDER BY
*/
SELECT
	ed.DivisionName,
	CAST(AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 AS DECIMAL (3,0)) AS
EngagementScore
FROM
	EmployeeData_2023 AS ed
INNER JOIN
	SurveyResponse_2023 AS sr
ON
	ed.EmployeeID = sr.EmployeeID
GROUP BY
	ed.DivisionName
ORDER BY
	EngagementScore DESC;


 /*
Headcount by Department, sorted by Headcount in DESC order
SQL concepts used - FROM, GROUP BY, SELECT, COUNT(), ORDER BY
*/
SELECT
	DepartmentName,
	COUNT(EmployeeID) AS Headcount
FROM
	EmployeeData_2023
GROUP BY
	DepartmentName
ORDER BY
	Headcount DESC;


 /*
 Engagement by Department, sorted by Engagement Score in DESC order
SQL concepts used - FROM, INNER JOIN, GROUP BY, SELECT, CAST(), AVG(), ORDER BY
*/
SELECT
	ed.DepartmentName,
	CAST(AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 AS DECIMAL (3,0)) AS EngagementScore
FROM
	EmployeeData_2023 AS ed
INNER JOIN
	SurveyResponse_2023 AS sr
ON
	ed.EmployeeID = sr.EmployeeID
GROUP BY
	DepartmentName
ORDER BY
	EngagementScore DESC;


/*
Headcount, Engagement Score & Engagement Level by Division, sorted by Engagement Score in DESC order
using CTE
*/
WITH CombinedData AS (
	SELECT
		ed.DivisionName,
		COUNT(DISTINCT ed.EmployeeID) AS Headcount,
		CAST(AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 AS DECIMAL (3,0)) AS EngagementScore,
	CASE
		WHEN AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 >= 80 THEN 'Highly Engaged'
		WHEN AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 >= 65 THEN 'Engaged'
		WHEN AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 >= 50 THEN 'Neutral'
		WHEN AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 >= 25 THEN 'Disengaged'
		ELSE 'Actively Disengaged'
	END AS 'EngagementLevel'
	FROM
		EmployeeData_2023 AS ed
	INNER JOIN
		SurveyResponse_2023 AS sr
	ON
		ed.EmployeeID = sr.EmployeeID
	GROUP BY
		ed.DivisionName)

SELECT
	DivisionName,
	Headcount,
	EngagementScore,
	EngagementLevel
FROM
	CombinedData
ORDER BY
	EngagementScore DESC;


/*
Headcount, Engagement Score & Engagement Level by Department, sorted by Engagement Score in DESC order
using CTE
*/
WITH CombinedData AS (
	SELECT
		ed.DepartmentName,
		COUNT(DISTINCT ed.EmployeeID) AS Headcount,
		CAST(AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 AS DECIMAL (3,0)) AS EngagementScore,
	CASE
		WHEN AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 >= 80 THEN 'Highly Engaged'
		WHEN AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 >= 65 THEN 'Engaged'
		WHEN AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 >= 50 THEN 'Neutral'
		WHEN AVG(CAST(sr.LikertScore AS DECIMAL(10,2))) / 5 * 100 >= 25 THEN 'Disengaged'
		ELSE 'Actively Disengaged'
	END AS 'EngagementLevel'
	FROM
		EmployeeData_2023 AS ed
	INNER JOIN
		SurveyResponse_2023 AS sr
	ON
		ed.EmployeeID = sr.EmployeeID
	GROUP BY
		ed.DepartmentName)

SELECT
	DepartmentName,
	Headcount,
	EngagementScore,
	EngagementLevel
FROM
	CombinedData
ORDER BY
	EngagementScore DESC;


 /*
Metric with custom order, via Mapping Table using INNER JOIN
*/
SELECT
	sqms.Metric
FROM
	SurveyQuestion_MarketScore_2023 AS sqms
INNER JOIN
	MetricMapping AS mm
ON
	sqms.Metric = mm.Metric
GROUP BY
	sqms.Metric,
	mm.MetricOrder
ORDER BY
	mm.MetricOrder;


 /* Average Score per Metric */
SELECT
	sqms.Metric,
	ROUND(AVG(CAST(sr.LikertScore AS FLOAT)), 2) AS AverageScore
FROM
	SurveyQuestion_MarketScore_2023 AS sqms
INNER JOIN
	MetricMapping AS mm
ON
	sqms.Metric = mm.Metric
INNER JOIN
	SurveyResponse_2023 AS sr
ON
	sqms.QuestionID = sr.QuestionID
GROUP BY
	sqms.Metric,
	mm.MetricOrder
ORDER BY
	mm.MetricOrder;


 /* Market Score per Metric */
SELECT
	sqms.Metric,
	ROUND(AVG(CAST(sqms.MarketScore AS FLOAT)), 2) AS MarketScore
FROM
	SurveyQuestion_MarketScore_2023 AS sqms
INNER JOIN
	MetricMapping AS mm
ON
	sqms.Metric = mm.Metric
INNER JOIN
	SurveyResponse_2023 AS sr
ON
	sqms.QuestionID = sr.QuestionID
GROUP BY
	sqms.Metric,
	mm.MetricOrder
ORDER BY
	mm.MetricOrder;


/*
Use ROW_NUMBER() with PARTITION BY Metric to rank questions within each Metric group, based on Average Score in DESC
PARTITION BY is to restart the row number within each Metric group
*/
WITH CombinedData AS (
	SELECT
		sqms.Metric,
		sqms.QuestionText,
		ROUND(AVG(CAST(sr.LikertScore AS FLOAT)), 2) AS AverageScore,
		mm.MetricOrder
	FROM
		SurveyQuestion_MarketScore_2023 AS sqms
	INNER JOIN
		MetricMapping AS mm
	ON
		sqms.Metric = mm.Metric
	INNER JOIN
		SurveyResponse_2023 AS sr
	ON
		sqms.QuestionID = sr.QuestionID
	GROUP BY
		sqms.Metric,
		sqms.QuestionText,
		mm.MetricOrder)

SELECT
	Metric,
	QuestionText,
	AverageScore,
	ROW_NUMBER() OVER(PARTITION BY Metric ORDER BY AverageScore DESC) AS RowNumber
FROM
	CombinedData
ORDER BY
	MetricOrder;


 /*
Top 3 questions for each Metric, multiple CTEs
*/
WITH CombinedData AS (
	SELECT
		sqms.Metric,
		sqms.QuestionText,
		ROUND(AVG(CAST(sr.LikertScore AS FLOAT)), 2) AS AverageScore,
		mm.MetricOrder
	FROM
		SurveyQuestion_MarketScore_2023 AS sqms
	INNER JOIN
		MetricMapping AS mm
	ON
		sqms.Metric = mm.Metric
	INNER JOIN
		SurveyResponse_2023 AS sr
	ON
		sqms.QuestionID = sr.QuestionID
	GROUP BY
		sqms.Metric,
		sqms.QuestionText,
		mm.MetricOrder),

RankedQuestion AS (
	SELECT
		Metric,
		QuestionText,
		AverageScore,
		ROW_NUMBER() OVER(PARTITION BY Metric ORDER BY AverageScore DESC) AS RowNumber,
		MetricOrder
	FROM
		CombinedData)

SELECT
	Metric,
	QuestionText,
	AverageScore
FROM
	RankedQuestion
WHERE
	RowNumber <= 3
ORDER BY
	MetricOrder;
 
