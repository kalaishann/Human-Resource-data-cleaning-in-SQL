# Human-Resource-data-cleaning-in-SQL

CREATE DATABASE HR_data;

USE HR_data;

SELECT * FROM HR_data;

-- data cleaning and preprocessing--

ALTER TABLE HR_data
CHANGE COLUMN ï»¿id emp_id VARCHAR(20) NULL;

DESCRIBE HR_data

SET sql_safe_updates = 0;

UPDATE HR_data
SET birthdate = CASE
		WHEN birthdate LIKE '%/%' THEN date_format(str_to_date(birthdate,'%m/%d/%Y'),'%Y-%m-%d')
        WHEN birthdate LIKE '%-%' THEN date_format(str_to_date(birthdate,'%m-%d-%Y'),'%Y-%m-%d')
        ELSE NULL
		END;
	
ALTER TABLE HR_data
MODIFY COLUMN birthdate DATE;


-- change the data format and datatype of hire_date column

UPDATE HR_data
SET hire_date = CASE
		WHEN hire_date LIKE '%/%' THEN date_format(str_to_date(hire_date,'%m/%d/%Y'),'%Y-%m-%d')
        WHEN hire_date LIKE '%-%' THEN date_format(str_to_date(hire_date,'%m-%d-%Y'),'%Y-%m-%d')
        ELSE NULL
		END;
        
ALTER TABLE HR_data
MODIFY COLUMN hire_date DATE;

-- change the date format and datatpye of termdate column
UPDATE HR_data
SET termdate = date(str_to_date(termdate, '%Y-%m-%d %H:%i:%s UTC'))
WHERE termdate IS NOT NULL AND termdate !='';

UPDATE HR_data
SET termdate = NULL
WHERE termdate = '';


-- create age column
ALTER TABLE HR_data
ADD column age INT;

UPDATE HR_data
SET age = timestampdiff(YEAR,birthdate,curdate())

SELECT min(age), max(age) FROM HR_data

-- 1. What is the gender breakdown of employees in the company
SELECT * FROM HR_data

SELECT gender, COUNT(*) AS count 
FROM HR_data
WHERE termdate IS NULL
GROUP BY gender;

-- 2. What is the race breakdown of employees in the company
SELECT race , COUNT(*) AS count
FROM HR_data
WHERE termdate IS NULL
GROUP BY race

-- 3. What is the age distribution of employees in the company ?
SELECT 
	CASE
		WHEN age>=18 AND age<=24 THEN '18-24'
        WHEN age>=25 AND age<=34 THEN '25-34'
        WHEN age>=35 AND age<=44 THEN '35-44'
        WHEN age>=45 AND age<=54 THEN '45-54'
        WHEN age>=55 AND age<=64 THEN '55-64'
        ELSE '65+'
	END AS age_group,
    COUNT(*) AS count
    FROM HR_data
    WHERE termdate IS NULL
    GROUP BY age_group
    ORDER BY age_group;
    
-- 4. How many employees work at HQ vs remote ?
SELECT location,COUNT(*) AS count
FROM HR_data
WHERE termdate IS NULL
GROUP BY location;


-- 5. What is the average length of employement who have been teminated ?
SELECT ROUND(AVG(year(termdate) - year(hire_date)),0) AS length_of_emp
FROM HR_data
WHERE termdate IS NOT NULL AND termdate <= curdate()

-- 6. How does the gender distribution vary acorss dept. and job titles
SELECT *  FROM HR_data

SELECT department,jobtitle,gender,COUNT(*) AS count
FROM HR_data
WHERE termdate IS NOT NULL
GROUP BY department, jobtitle,gender
ORDER BY department, jobtitle,gender;

SELECT department,gender,COUNT(*) AS count
FROM HR_data
WHERE termdate IS NOT NULL
GROUP BY department,gender
ORDER BY department,gender;

-- 7. What is the distribution of jobtitles acorss the company
SELECT jobtitle, COUNT(*) AS count
FROM HR_data
WHERE termdate IS NULL
GROUP BY jobtitle;

-- 8. Which dept has the higher turnover/termination rate

SELECT * FROM HR_data;

SELECT department,
		COUNT(*) AS total_count,
        COUNT(CASE
				WHEN termdate IS NOT NULL AND termdate <= curdate() THEN 1 
				END) AS terminated_count,
		ROUND((COUNT(CASE
					WHEN termdate IS NOT NULL AND termdate <= curdate() THEN 1 
                    END)/COUNT(*))*100,2) AS termination_rate
		FROM HR_data
        GROUP BY department
        ORDER BY termination_rate DESC;
        
        
-- 9. What is the distribution of employees across location_state ?
SELECT location_state, COUNT(*) AS count
FROM HR_data
WHERE termdate IS NULL
GROUP BY location_state;

SELECT location_city, COUNT(*) AS count
FROM HR_data
WHERE termdate IS NULL
GROUP BY location_city;

-- 10. How has the companys employee count changed over time based on hire and termination date.
SELECT * FROM HR_data;

SELECT year,
		hires,
        terminations,
        hires-terminations AS net_change,
        (terminations/hires)*100 AS change_percent
	FROM(
			SELECT YEAR(hire_date) AS year,
            COUNT(*) AS hires,
            SUM(CASE 
					WHEN termdate IS NOT NULL AND termdate <= curdate() THEN 1 
				END) AS terminations
			FROM HR_data
            GROUP BY YEAR(hire_date)) AS subquery
GROUP BY year
ORDER BY year;

-- 11. What is the tenure distribution for each dept.
SELECT department, round(avg(datediff(termdate,hire_date)/365),0) AS avg_tenure
FROM HR_data
WHERE termdate IS NOT NULL AND termdate<= curdate()
GROUP BY department;
