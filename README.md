
# 💼 Data Science Job Trends Analysis using SQL

## Project Overview

This project presents a detailed SQL-based analysis of global **Data Science job trends**. It answers real-world business problems such as salary distribution, remote work patterns, domain-switch guidance, and more, using structured queries on a real dataset.

![Data Science](https://cdn-icons-png.flaticon.com/512/1055/1055687.png)

---

## Objectives

1. Import and analyze structured job data from the Data Science industry.
2. Solve real-world HR, compensation, and recruitment problems using SQL.
3. Provide salary-driven guidance for career shifts.
4. Identify salary trends and remote work evolution (2020–2024).
5. Derive actionable insights for consultants, data scientists, and analysts.
6. Ensure data security and implement access control based on experience levels.

---

## 📥 Dataset Import

To load the dataset into your SQL database, you can use the following command (MySQL example):

```sql
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/salaries.csv'
INTO TABLE your_database.salaries
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

---

## Business Tasks & SQL Solutions

---

### 🌍 Task 1: Remote Managers Earning Above $90,000

**Objective:** Identify countries offering fully remote jobs for "Managers" with salaries exceeding $90,000 USD.

```sql
SELECT company_location
FROM salaries
WHERE job_title LIKE '%Manager%'
  AND remote_ratio = 100
  AND salary_in_usd > 90000;
```

---

### 🏢 Task 2: Top 5 Countries with Most Large Companies

**Objective:** Identify countries with the highest count of large-sized companies.

```sql
SELECT company_location, COUNT(*) AS company_count
FROM salaries
WHERE company_size = 'L'
GROUP BY company_location
ORDER BY company_count DESC
LIMIT 5;
```

---

### 💸 Task 3: Percentage of High-Paying Fully Remote Roles

**Objective:** Calculate % of employees earning > $100K in fully remote roles.

```sql
SELECT 
  ROUND(
    (SELECT COUNT(*) 
     FROM salaries 
     WHERE remote_ratio = 100 AND salary_in_usd > 100000) * 100.0 / COUNT(*), 2
  ) AS high_paid_remote_percentage
FROM salaries;
```

---

### 🌎 Task 4: Entry-Level Locations with Higher Than Market Avg

**Objective:** Compare entry-level salary by location to global entry-level average.

```sql
WITH global_avg AS (
  SELECT AVG(salary_in_usd) AS avg_salary
  FROM salaries
  WHERE experience_level = 'EN'
)
SELECT company_location, AVG(salary_in_usd) AS avg_entry_salary
FROM salaries, global_avg
WHERE experience_level = 'EN'
GROUP BY company_location
HAVING avg_entry_salary > global_avg.avg_salary;
```

---

### 🏆 Task 5: Highest Paying Country by Job Title

**Objective:** For each job title, find the country with the highest average salary.

```sql
SELECT job_title, company_location, MAX(avg_salary) AS max_avg_salary
FROM (
  SELECT job_title, company_location, AVG(salary_in_usd) AS avg_salary
  FROM salaries
  GROUP BY job_title, company_location
) t
GROUP BY job_title;
```

---

## 📁 Files in the Repository

| File Name                      | Description                             |
|-------------------------------|-----------------------------------------|
| `salaries.csv`                | Dataset with job-level salary records   |
| `task_solution.sql`           | SQL queries solving business problems   |
| `Data Science job SQL questions.docx` | Business problem descriptions     |
| `README.md`                   | Project documentation (this file)       |

---

## 🚀 How to Use

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/data-science-job-sql-analysis.git
   ```

2. Import `salaries.csv` into MySQL/PostgreSQL database.

3. Run SQL queries from `task_solution.sql` to explore insights and solve tasks.

---

## 📬 Contact

- [LinkedIn](https://linkedin.com/in/your-profile)
- [GitHub](https://github.com/yourusername)

---

⭐ Star this repo if you found it insightful!


---

### 📊 Task 6: Countries with Sustained Salary Growth (3 Years)

**Objective:** Identify countries where average salary has increased over three consecutive years.

```sql
WITH yearly_avg AS (
  SELECT company_location, work_year, AVG(salary_in_usd) AS avg_salary
  FROM salaries
  GROUP BY company_location, work_year
),
filtered AS (
  SELECT * FROM yearly_avg
  WHERE work_year IN (2022, 2023, 2024)
)
SELECT company_location
FROM filtered
GROUP BY company_location
HAVING COUNT(*) = 3
   AND MIN(avg_salary) < MAX(avg_salary)
   AND MAX(avg_salary) > (
        SELECT avg_salary FROM filtered WHERE work_year = 2023 AND company_location = filtered.company_location
   );
```

---

### 🔁 Task 7: Remote Work Trend Comparison (2021 vs 2024)

**Objective:** Compare remote work % across experience levels in 2021 and 2024.

```sql
SELECT experience_level, 
       ROUND(SUM(CASE WHEN work_year = 2021 AND remote_ratio = 100 THEN 1 ELSE 0 END) * 100.0 / COUNT(CASE WHEN work_year = 2021 THEN 1 END), 2) AS pct_2021,
       ROUND(SUM(CASE WHEN work_year = 2024 AND remote_ratio = 100 THEN 1 ELSE 0 END) * 100.0 / COUNT(CASE WHEN work_year = 2024 THEN 1 END), 2) AS pct_2024
FROM salaries
WHERE work_year IN (2021, 2024)
GROUP BY experience_level;
```

---

### 📈 Task 8: Salary Growth % (2023 to 2024)

**Objective:** Calculate average salary increase % by job title and experience level.

```sql
WITH t23 AS (
  SELECT job_title, experience_level, AVG(salary_in_usd) AS avg_2023
  FROM salaries
  WHERE work_year = 2023
  GROUP BY job_title, experience_level
),
t24 AS (
  SELECT job_title, experience_level, AVG(salary_in_usd) AS avg_2024
  FROM salaries
  WHERE work_year = 2024
  GROUP BY job_title, experience_level
)
SELECT t24.job_title, t24.experience_level,
       ROUND(((t24.avg_2024 - t23.avg_2023) / t23.avg_2023) * 100, 2) AS salary_growth_pct
FROM t24
JOIN t23 ON t24.job_title = t23.job_title AND t24.experience_level = t23.experience_level;
```

---

### 🔒 Task 9: Role-Based Access by Experience Level

**Objective:** Simulate access control for data visibility by experience level.

```sql
-- Example: Only allow access to records matching a user's experience level
SELECT *
FROM salaries
WHERE experience_level = 'EN';  -- Replace 'EN' with dynamic session value for user level
```

---

### 🔄 Task 10: Recommend Domain Switch Based on Salary

**Objective:** Suggest domain switch based on user's preferences and salary data.

```sql
SELECT job_title, AVG(salary_in_usd) AS avg_salary
FROM salaries
WHERE experience_level = 'SE'
  AND employment_type = 'FT'
  AND company_location = 'US'
  AND company_size = 'L'
GROUP BY job_title
ORDER BY avg_salary DESC;
```

---

### 🧮 Task 11: Employee Count by Company Size in 2021

**Objective:** Show distribution of employees by company size.

```sql
SELECT company_size, COUNT(*) AS employee_count
FROM salaries
WHERE work_year = 2021
GROUP BY company_size;
```

---

### 💼 Task 12: Top 3 High-Salary Part-Time Jobs (2023)

**Objective:** Find top 3 paid part-time job titles in countries with >50 employees.

```sql
WITH country_counts AS (
  SELECT company_location
  FROM salaries
  WHERE work_year = 2023
  GROUP BY company_location
  HAVING COUNT(*) > 50
)
SELECT job_title, AVG(salary_in_usd) AS avg_salary
FROM salaries
WHERE employment_type = 'PT'
  AND work_year = 2023
  AND company_location IN (SELECT company_location FROM country_counts)
GROUP BY job_title
ORDER BY avg_salary DESC
LIMIT 3;
```

---

### ⚖️ Task 13: Countries with Above-Average Mid-Level Salary (2023)

**Objective:** Compare countries to the global average.

```sql
WITH overall_avg AS (
  SELECT AVG(salary_in_usd) AS avg_salary
  FROM salaries
  WHERE experience_level = 'MI' AND work_year = 2023
)
SELECT company_location, AVG(salary_in_usd) AS avg_salary
FROM salaries, overall_avg
WHERE experience_level = 'MI' AND work_year = 2023
GROUP BY company_location
HAVING AVG(salary_in_usd) > overall_avg.avg_salary;
```

---

### 📍 Task 14: Highest & Lowest Senior-Level Salary (2023)

**Objective:** Find company locations with the max/min average salary for senior employees.

```sql
SELECT company_location, AVG(salary_in_usd) AS avg_salary
FROM salaries
WHERE experience_level = 'SE' AND work_year = 2023
GROUP BY company_location
ORDER BY avg_salary DESC;
```

---

### 📊 Task 15: Salary Growth by Job Title

**Objective:** Calculate salary growth rate by job title between years.

```sql
WITH t1 AS (
  SELECT job_title, AVG(salary_in_usd) AS avg_prev
  FROM salaries
  WHERE work_year = 2023
  GROUP BY job_title
),
t2 AS (
  SELECT job_title, AVG(salary_in_usd) AS avg_curr
  FROM salaries
  WHERE work_year = 2024
  GROUP BY job_title
)
SELECT t2.job_title, ROUND(((t2.avg_curr - t1.avg_prev) / t1.avg_prev) * 100, 2) AS salary_growth_pct
FROM t2
JOIN t1 ON t1.job_title = t2.job_title;
```

---

### 🌍 Task 16: Top 3 Countries for Entry-Level Salary Growth (2020–2023)

**Objective:** Identify growth across years for entry roles in countries with > 50 employees.

```sql
WITH avg_salary AS (
  SELECT company_location, work_year, AVG(salary_in_usd) AS avg_salary
  FROM salaries
  WHERE experience_level = 'EN'
  GROUP BY company_location, work_year
),
pivoted AS (
  SELECT company_location,
         MAX(CASE WHEN work_year = 2020 THEN avg_salary END) AS salary_2020,
         MAX(CASE WHEN work_year = 2023 THEN avg_salary END) AS salary_2023
  FROM avg_salary
  GROUP BY company_location
),
filtered AS (
  SELECT company_location, salary_2020, salary_2023,
         ROUND(((salary_2023 - salary_2020) / salary_2020) * 100, 2) AS growth_pct
  FROM pivoted
  WHERE salary_2020 IS NOT NULL AND salary_2023 IS NOT NULL
)
SELECT * FROM filtered
ORDER BY growth_pct DESC
LIMIT 3;
```

---

### 🧑‍💻 Task 17: Update Remote Ratio for $90K+ Earners (US/AU)

**Objective:** Update remote work status for eligible employees.

```sql
UPDATE salaries
SET remote_ratio = 100
WHERE salary_in_usd > 90000
  AND company_location IN ('US', 'AU');
```

---

### 📅 Task 18: Year with Highest Avg Salary by Job Title

**Objective:** Find peak salary year for each role.

```sql
WITH yearly_avg AS (
  SELECT job_title, work_year, AVG(salary_in_usd) AS avg_salary
  FROM salaries
  GROUP BY job_title, work_year
)
SELECT job_title, work_year, avg_salary
FROM (
  SELECT *, RANK() OVER(PARTITION BY job_title ORDER BY avg_salary DESC) AS rnk
  FROM yearly_avg
) t
WHERE rnk = 1;
```

---

### 📊 Task 19: Employment Type % by Job Title

**Objective:** Show employment type distribution per role.

```sql
SELECT job_title,
       ROUND(SUM(CASE WHEN employment_type = 'FT' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS Full_Time,
       ROUND(SUM(CASE WHEN employment_type = 'PT' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS Part_Time,
       ROUND(SUM(CASE WHEN employment_type = 'CT' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS Contract,
       ROUND(SUM(CASE WHEN employment_type = 'FL' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS Freelance
FROM salaries
GROUP BY job_title;
```



---

## 🛠 Original vs Refactored SQL Queries

For transparency and learning purposes, here’s a side-by-side comparison of the original SQL queries (as written in `task_solution.sql`) and the refactored versions included above in the README.

---

### 🌍 Task 1: Remote Managers Earning Above $90,000

**Original:**
```sql
SELECT company_location FROM salaries
WHERE job_title LIKE '%Manager%' AND remote_ratio = 100 AND salary_in_usd > 90000;
```

**Refactored:**
```sql
SELECT company_location
FROM salaries
WHERE job_title LIKE '%Manager%'
  AND remote_ratio = 100
  AND salary_in_usd > 90000;
```

---

### 🏢 Task 2: Top 5 Countries with Most Large Companies

**Original:**
```sql
SELECT company_location , count(*) FROM salaries
WHERE company_size='L'
GROUP BY company_location
ORDER BY count(*) DESC LIMIT 5;
```

**Refactored:**
```sql
SELECT company_location, COUNT(*) AS company_count
FROM salaries
WHERE company_size = 'L'
GROUP BY company_location
ORDER BY company_count DESC
LIMIT 5;
```

---

### 💸 Task 3: Percentage of High-Paying Fully Remote Roles

**Original:**
```sql
SELECT ROUND((SELECT COUNT(*) FROM salaries WHERE remote_ratio=100 AND salary_in_usd>100000)*100/COUNT(*),2)
FROM salaries;
```

**Refactored:**
```sql
SELECT 
  ROUND(
    (SELECT COUNT(*) 
     FROM salaries 
     WHERE remote_ratio = 100 AND salary_in_usd > 100000) * 100.0 / COUNT(*), 2
  ) AS high_paid_remote_percentage
FROM salaries;
```

---

### 🌎 Task 4: Entry-Level Locations with Higher Than Market Avg

**Original:**
```sql
WITH t AS (
	SELECT AVG(salary_in_usd) AS avg_sal FROM salaries WHERE experience_level='EN'
)
SELECT company_location, AVG(salary_in_usd) AS avg_salary
FROM salaries, t
WHERE experience_level='EN'
GROUP BY company_location
HAVING AVG(salary_in_usd) > t.avg_sal;
```

**Refactored:**
```sql
WITH global_avg AS (
  SELECT AVG(salary_in_usd) AS avg_salary
  FROM salaries
  WHERE experience_level = 'EN'
)
SELECT company_location, AVG(salary_in_usd) AS avg_entry_salary
FROM salaries, global_avg
WHERE experience_level = 'EN'
GROUP BY company_location
HAVING avg_entry_salary > global_avg.avg_salary;
```

---

### 🏆 Task 5: Highest Paying Country by Job Title

**Original:**
*Not present in original SQL file — Added based on business prompt.*

**Refactored:**
```sql
SELECT job_title, company_location, MAX(avg_salary) AS max_avg_salary
FROM (
  SELECT job_title, company_location, AVG(salary_in_usd) AS avg_salary
  FROM salaries
  GROUP BY job_title, company_location
) t
GROUP BY job_title;
```

---

### 📊 Task 6 to 19:

*Many of these were not included in the original SQL file. For these, queries were designed based on the business problem provided in the `.docx` file.*

If you'd like, I can help extract **exact queries from your `task_solution.sql`** file (wherever applicable) and match them line-by-line with the refactored ones above in the README.

---

📌 This dual-format helps you:
- Understand cleaner ways to write SQL (refactored)
- Preserve your original thought process (original)

Let me know if you'd like this exported as a PDF version for documentation/portfolio!
