
# 💼 Data Science Job Trends Analysis using SQL

## Project Overview

This project presents a detailed SQL-based analysis of Data Science job data. It covers business problems ranging from salary insights, remote work adoption, domain transitions, and recruitment strategies — all solved using structured SQL queries.

![Data Science](https://cdn-icons-png.flaticon.com/512/1055/1055687.png)

---

## Objectives

1. Import and analyze structured job data from the Data Science industry.
2. Solve real-world business problems using SQL.
3. Investigate trends across salary, experience level, job titles, and more.
4. Provide decision-support for HR, analysts, and consultants.
5. Deliver actionable insights from raw datasets.

---

## 📥 Dataset Import

To load the dataset into your SQL database, the following command was used:

```sql
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/salaries.csv'
INTO TABLE campusx.salaries
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 ROWS;
```

---

## Business Tasks & SQL Solutions


---

### 🧠 Task 1: You're a Compensation analyst employed by a multinational corporation.  	 Your Assignment is to Pinpoint Countries who give work fully remotely, for the title 	 'managers’ Paying salaries Exceeding $90,000 USD

```sql
select distinct company_location from salaries
where job_title like '%Manager%' and salary_in_usd > 90000 and remote_ratio = 100;
```


---

### 🧠 Task 2: AS a remote work advocate Working for a progressive HR tech      startup who place their freshers’ clientsIN large tech firms.you're tasked WITH Identifying top 5 Country 	 Having greatest count of large (company size) number of companies.

```sql
select company_location , count(*) as cnt from (
select * from salaries where experience_level = 'EN' and company_size = 'L'
) t
group by company_location
order by cnt desc limit 5;
```


---

### 🧠 Task 3: Picture yourself AS a data scientist Working for a workforce management platform. 	 Your objective is to calculate the percentage of employees. 	 Who enjoy fully remote roles WITH salaries Exceeding $100,000 USD, 	 Shedding light ON the attractiveness of high-paying remote positions IN today's job market.

```sql
DO $$
DECLARE
    count_val INT;
    total_val INT;
    percentage NUMERIC;
BEGIN
    SELECT COUNT(*) INTO count_val
    FROM salaries
    WHERE salary_in_usd > 100000 and remote_ratio = 100;

    SELECT COUNT(*) INTO total_val
    FROM salaries  WHERE salary_in_usd > 100000;

    percentage := (count_val::NUMERIC / total_val) * 100;

    RAISE NOTICE 'High salary percentage: %', percentage;
END $$;
```


---

### 🧠 Task 4: Imagine you're a data analyst Working for a global recruitment agency. 	Your Task is to identify the Locations where entry-level average salaries exceed the average salary for that job title 	IN market for entry level, helping your agency guide candidates towards lucrative opportunities.

```sql
select t.job_title , company_location , avg_salary , per_country_avg_salary from (
		select job_title , avg(salary_in_usd) as avg_salary from salaries
		group by job_title
) t
inner join
(
	select company_location , job_title  , avg(salary_in_usd) as per_country_avg_salary from salaries 
	group by job_title , company_location
) m

on t.job_title = m.job_title
where per_country_avg_salary > avg_salary
order by job_title asc , company_location asc;
```


---

### 🧠 Task 5: You've been hired by a big HR Consultancy to look at how much people get paid IN different Countries. 	 Your job is to Find out for each job title which. Country pays the maximum average salary.  	 This helps you to place your candidates IN those countries.

```sql
select t.job_title ,  t.company_location , max(avg_salary) as maxi_avg from(
	select job_title , company_location , avg(salary_in_usd) as avg_salary
	from salaries 
	group by job_title , company_location
	order by job_title asc
) t
group by 
t.job_title , t.company_location;
```


---

### 🧠 Task 6: AS a data-driven Business consultant, you've been hired by a multinational corporation to analyze salary trends across  	different company Locations. Your goal is to Pinpoint Locations WHERE the average salary Has consistently Increased over 	the Past few years (Countries WHERE data is available for 3 years Only(present year and past two years) providing Insights  	into Locations experiencing Sustained salary growth.

```sql
-- step 1. This part filters out any company locations that don’t have salary data for all 3 years (2022, 2023, and 2024).
WITH my_table AS (
    SELECT * 
    FROM salaries 
    WHERE company_location IN (
        SELECT company_location 
        FROM (
            SELECT 
                company_location,  
                ROUND(AVG(salary_in_usd), 2) AS avg_salary, 
                COUNT(DISTINCT work_year) AS cnt
            FROM salaries 
            WHERE (work_year + 1) >= (EXTRACT(YEAR FROM CURRENT_DATE) - 2)
            GROUP BY company_location
            HAVING COUNT(DISTINCT work_year) = 3
        ) t
    )
)

-- step 3. This is a pivot operation: you’re converting year-wise rows into columns.
SELECT 
    company_location,
    MAX(CASE WHEN work_year = 2022 THEN average END) AS avg_salary_2022,
    MAX(CASE WHEN work_year = 2023 THEN average END) AS avg_salary_2023,
    MAX(CASE WHEN work_year = 2024 THEN average END) AS avg_salary_2024
FROM (
    --step 2. Now, for each valid company location, you're calculating the average salary for each year (2022–2024).
    SELECT 
        company_location,
        work_year,
        Round(AVG(salary_in_usd),2) AS average
    FROM my_table 
    GROUP BY company_location, work_year
) AS avg_by_year
GROUP BY company_location
having MAX(CASE WHEN work_year = 2024 THEN average END) > MAX(CASE WHEN work_year = 2023 THEN average END) and
MAX(CASE WHEN work_year = 2023 THEN average END) > MAX(CASE WHEN work_year = 2022 THEN average END);
```


---

### 🧠 Task 7: Picture yourself AS a workforce strategist employed by a global HR tech startup. Your Mission is to Determine the percentage 	of fully remote work for each experience level IN 2021 and compare it WITH the corresponding figures for 2024, 	Highlighting any significant Increases or decreases IN remote work Adoption over the years.

```sql
select x.experience_level , remote_ratio_2021 , remote_ratio_2024 from (

	SELECT *, 
	       Round((cnt::DECIMAL / total::DECIMAL) * 100,2) AS remote_ratio_2021
	FROM (
	    SELECT 
	        t.experience_level, 
	        t.total, 
	        m.cnt
	    FROM (
	        SELECT experience_level, COUNT(*) AS total
	        FROM salaries
	        WHERE work_year = 2021
	        GROUP BY experience_level
	    ) t
	    INNER JOIN (
	        SELECT experience_level, COUNT(*) AS cnt
	        FROM salaries
	        WHERE work_year = 2021 AND remote_ratio = 100
	        GROUP BY experience_level
	    ) m ON t.experience_level = m.experience_level
	) l
)x
inner join
(
	SELECT *, 
	       Round((cnt::DECIMAL / total::DECIMAL) * 100,2) AS remote_ratio_2024
	FROM (
	    SELECT 
	        t.experience_level, 
	        t.total, 
	        m.cnt
	    FROM (
	        SELECT experience_level, COUNT(*) AS total
	        FROM salaries
	        WHERE work_year = 2024
	        GROUP BY experience_level
	    ) t
	    INNER JOIN (
	        SELECT experience_level, COUNT(*) AS cnt
	        FROM salaries
	        WHERE work_year = 2024 AND remote_ratio = 100
	        GROUP BY experience_level
	    ) m ON t.experience_level = m.experience_level
	) l
) y
on x.experience_level = y.experience_level;
```


---

### 🧠 Task 8: AS a Compensation specialist at a Fortune 500 company, you're tasked WITH analyzing salary trends over time. 	  Your objective is to calculate the average salary increase percentage for each experience level and job title between  	  the years 2023 and 2024, helping the company stay competitive IN the talent market.

```sql
WITH salary_2023 AS (
    SELECT 
        experience_level,
        job_title,
        AVG(salary_in_usd) AS avg_salary_2023
    FROM salaries
    WHERE work_year = 2023
    GROUP BY experience_level, job_title
),
salary_2024 AS (
    SELECT 
        experience_level,
        job_title,
        AVG(salary_in_usd) AS avg_salary_2024
    FROM salaries
    WHERE work_year = 2024
    GROUP BY experience_level, job_title
)

SELECT 
    s23.experience_level,
    s23.job_title,
    ROUND(((s24.avg_salary_2024 - s23.avg_salary_2023) / s23.avg_salary_2023) * 100, 2) AS avg_salary_increase_percent
FROM salary_2023 s23
JOIN salary_2024 s24
ON s23.experience_level = s24.experience_level
AND s23.job_title = s24.job_title
ORDER BY avg_salary_increase_percent DESC;
```


---

### 🧠 Task 9: You're a database administrator tasked with role-based access control for a company's employee database. 	Your goal is to implement a security measure where employees in different experience level (e.g. Entry Level, Senior level etc.)  	can only access details relevant to their respective experience level, ensuring data confidentiality and minimizing the  	risk of unauthorized access.

```sql

```


---

### 🧠 Task 10: Picture yourself as a data architect responsible for database management. Companies in US and AU(Australia) decided to 	create a hybrid model for employees they decided that employees earning salaries exceeding $90000 USD, will be given work from home. 	You now need to update the remote work ratio for eligible employees, ensuring efficient remote work management 	while implementing appropriate error handling mechanisms for invalid input parameters.

```sql
DO $$
DECLARE
    v_salary_threshold NUMERIC := 90000;
    v_updated_count INT;
BEGIN
    -- Error handling: Check if salary threshold is valid
    IF v_salary_threshold <= 0 THEN
        RAISE EXCEPTION 'Invalid salary threshold: must be greater than 0';
    END IF;

    -- Perform update for employees in US or AU with salary above the threshold
    UPDATE salaries
    SET remote_ratio = 100
    WHERE company_location IN ('US', 'AU')
      AND salary_in_usd > v_salary_threshold;

    GET DIAGNOSTICS v_updated_count = ROW_COUNT;

    RAISE NOTICE 'Successfully updated % employee(s) for remote work eligibility.', v_updated_count;

END $$;
```


---

### 🧠 Task 11: As a market researcher, your job is to Investigate the job market for a company that analyzes workforce data. 	Your Task is to know how many people were employed IN different types of companies AS per their size IN 2021.

```sql
SELECT 
    company_size,
    COUNT(*) AS total_employees
FROM 
    salaries
WHERE 
    work_year = 2021
GROUP BY 
    company_size
ORDER BY 
    total_employees DESC;
```


---

### 🧠 Task 12: Imagine you are a talent Acquisition specialist Working for an International recruitment agency. 	  Your Task is to identify the top 3 job titles that command the highest average salary Among part-time Positions IN the year 2023. 	  However, you are Only Interested IN Countries WHERE there are more than 50 employees, 	  Ensuring a robust sample size for your analysis.

```sql
WITH eligible_countries AS (
    SELECT 
        employee_residence
    FROM 
        salaries
    WHERE 
        work_year = 2023
        AND employment_type = 'PT'
    GROUP BY 
        employee_residence
    HAVING 
        COUNT(*) > 50
)

SELECT 
    job_title,
    ROUND(AVG(salary_in_usd), 2) AS avg_salary_usd
FROM 
    salaries
WHERE 
    work_year = 2023
    AND employment_type = 'PT'
    AND employee_residence IN (SELECT employee_residence FROM eligible_countries)
GROUP BY 
    job_title
ORDER BY 
    avg_salary_usd DESC
LIMIT 3;
```


---

### 🧠 Task 13: As a database analyst you have been assigned the task to Select Countries where average senior-level salary  	is higher than overall senior-level salary for the year 2023.

```sql
SELECT 
    employee_residence AS country,
    ROUND(AVG(salary_in_usd), 2) AS avg_senior_salary
FROM 
    salaries
WHERE 
    work_year = 2023
    AND experience_level = 'SE'
GROUP BY 
    employee_residence
HAVING 
    AVG(salary_in_usd) > (
        SELECT 
            AVG(salary_in_usd)
        FROM 
            salaries
        WHERE 
            work_year = 2023
            AND experience_level = 'SE'
    )
ORDER BY 
    avg_senior_salary DESC;
```


---

### 🧠 Task 14: As a database analyst you have been assigned the task to Identify the company locations with the highest and lowest 	average salary for senior-level (SE) employees in 2023.

```sql
-- 1st method
select company_location , rank from(
	select company_location , avg(salary_in_usd) as avg_salary,
	dense_rank() over(order by avg(salary_in_usd) desc ) as rank
		from salaries where experience_level = 'SE' and work_year = 2023
		group by company_location
		order by avg(salary_in_usd) desc 
) t
where  rank = 1 or rank= 37;

-- 2nd method
WITH location_avg AS (
    SELECT 
        company_location,
        ROUND(AVG(salary_in_usd), 2) AS avg_salary,
        RANK() OVER (ORDER BY AVG(salary_in_usd) DESC) AS high_rank,
        RANK() OVER (ORDER BY AVG(salary_in_usd) ASC) AS low_rank
    FROM 
        salaries
    WHERE 
        work_year = 2023
        AND experience_level = 'SE'
    GROUP BY 
        company_location
)

SELECT 
    company_location,
    avg_salary,
    'Highest' AS salary_type
FROM 
    location_avg
WHERE 
    high_rank = 1

UNION

SELECT 
    company_location,
    avg_salary,
    'Lowest' AS salary_type
FROM 
    location_avg
WHERE 
    low_rank = 1;
```


---

### 🧠 Task 15: You're a Financial analyst Working for a leading HR Consultancy, and your Task is to Assess the annual salary growth rate 	for various job titles. By Calculating the percentage Increase IN salary FROM previous year to this year, 	you aim to provide valuable Insights Into salary trends WITHIN different job roles.

```sql
WITH yearly_avg_salary AS (
    SELECT 
        job_title,
        work_year,
        ROUND(AVG(salary_in_usd), 2) AS avg_salary
    FROM 
        salaries
    WHERE 
        work_year IN (2022, 2023)
    GROUP BY 
        job_title, work_year
),

salary_growth AS (
    SELECT 
        a.job_title,
        a.avg_salary AS salary_2022,
        b.avg_salary AS salary_2023,
        ROUND(((b.avg_salary - a.avg_salary) / a.avg_salary) * 100, 2) AS salary_growth_percent
    FROM 
        yearly_avg_salary a
    JOIN 
        yearly_avg_salary b
    ON 
        a.job_title = b.job_title
    WHERE 
        a.work_year = 2022 AND b.work_year = 2023
)

SELECT 
    job_title,
    salary_2022,
    salary_2023,
    salary_growth_percent
FROM 
    salary_growth
ORDER BY 
    salary_growth_percent DESC;
```


---

### 🧠 Task 16: You are a researcher and you have been assigned the task to Find the 	year with the highest average salary for each job title.

```sql
select * from salaries;

select job_title, work_year , avg_salary from(
	select job_title , work_year , Round(avg(salary_in_usd),2) as avg_salary,
	row_number() over(partition by job_title order by Round(avg(salary_in_usd),2) desc ) as rank
	from salaries
	group by job_title , work_year
	order by job_title asc , work_year asc
) t
where rank = 1;
```


---

### 🧠 Task 17: You have been hired by a market research agency where you been assigned the task to show the percentage of different 	employment type (full time, part time) in Different job roles, in the format where each row will be job title, each column  	will be type of employment type and cell value for that row and column will show the % value.

```sql
SELECT
  job_title,
  ROUND(
    100.0 * SUM(CASE WHEN employment_type = 'FT' THEN 1 ELSE 0 END) / COUNT(*), 
    2
  ) AS full_time_percentage,
  ROUND(
    100.0 * SUM(CASE WHEN employment_type = 'PT' THEN 1 ELSE 0 END) / COUNT(*), 
    2
  ) AS part_time_percentage
FROM salaries
GROUP BY job_title
ORDER BY job_title;
```


---

## 📁 Files in the Repository

| File Name | Description |
|-----------|-------------|
| `salaries.csv` | Cleaned dataset of Data Science jobs |
| `task_solution.sql` | SQL queries solving business problems |
| `Data Science job SQL questions.docx` | Business problem statements |
| `README.md` | Project documentation |

---

## 🚀 How to Use

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/data-science-sql-analysis.git
   ```

2. Load `salaries.csv` into MySQL/PostgreSQL DB.

3. Run `task_solution.sql` queries to explore data and solve business problems.

---

## 📬 Contact

- [LinkedIn](https://www.linkedin.com/in/your-profile)
- [GitHub](https://github.com/yourusername)

---

⭐ Star this repo if you found it useful!
