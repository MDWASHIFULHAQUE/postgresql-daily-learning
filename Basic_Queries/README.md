# PostgreSQL Comprehensive Daily Query Practice Notebook

This repository serves as a structured daily learning log tracking my progress through PostgreSQL data retrieval, sorting mechanisms, conditional row filters, wildcard pattern matching, arithmetic updates, aggregations, and advanced table relational joins.

## Dataset Context
Operations are executed across multiple relational databases:
1. Job Postings Database (schema tables like job_postings_fact, company_dim, skills_dim, and skills_to_job)
2. Invoices Database (schema tables like invoices_fact)

---

## Sorting and Distinct Rows (ORDER BY and DISTINCT)

### Exercise 1: Unique Locations Sorted Alphabetically (ASC)
Goal: Extract unique job locations and sort them from A to Z.
```sql
SELECT DISTINCT
    job_location
FROM
    job_postings_fact
ORDER BY
    job_location ASC;
```

### Exercise 2: Sorting Columns in Descending Order (DESC)
Goal: Retrieve core job characteristics ordered by location from Z to A.
```sql
SELECT
    job_id,
    job_title_short,
    job_location,
    job_via
FROM
    job_postings_fact
ORDER BY
    job_location DESC;
```

---

## Standard Row Filtering (WHERE, NOT, <>)

### Exercise 3: Exact Match Filtering (=)
Goal: Find details strictly for Data Engineer postings.
```sql
SELECT
    job_id,
    job_title_short,
    job_location,
    job_via,
    salary_year_avg
FROM
    job_postings_fact
WHERE
    job_title_short = 'Data Engineer';
```

### Exercise 4: Excluding Items Using WHERE NOT
Goal: Use explicit logical negation to find all job postings except Data Engineer.
```sql
SELECT
    job_id,
    job_title_short,
    job_location,
    job_via,
    salary_year_avg
FROM
    job_postings_fact
WHERE NOT
    job_title_short = 'Data Engineer';
```

### Exercise 5: Excluding Items Using the Inequality Operator (<>)
Goal: Achieve the same exclusion as above using standard comparison operations.
```sql
SELECT
    job_id,
    job_title_short,
    job_location,
    job_via,
    salary_year_avg
FROM
    job_postings_fact
WHERE
    job_title_short <> 'Data Engineer';
```

### Exercise 6: Filtering Out Specific Sources
Goal: Exclude all jobs posted via LinkedIn.
```sql
SELECT 
    job_id,
    job_title_short,
    job_location,
    job_via,
    salary_year_avg
FROM 
    job_postings_fact
WHERE 
    job_via <> 'via LinkedIn';
```

### Exercise 7: Value Set Matching (IN)
Goal: Return postings originating from specific target countries.
```sql
SELECT 
    job_id,
    job_title_short,
    job_location,
    job_via,
    salary_year_avg
FROM 
    job_postings_fact
WHERE 
    job_location IN ('Belgium', 'Argentina');
```

### Exercise 8: Filtering on Exact Text Properties
Goal: Look up rows matching full-time work constraints.
```sql
SELECT 
    job_id,
    job_title_short,
    job_location,
    job_via,
    salary_year_avg
FROM 
    job_postings_fact
WHERE 
    job_schedule_type = 'Full-time';
```

### Exercise 9: Numerical Greater-Than-Or-Equal Conditions (>=)
Goal: Isolate jobs offering average annual salaries of $65,000 or more.
```sql
SELECT 
    job_id,
    job_title_short,
    job_location,
    job_via,
    salary_year_avg
FROM 
    job_postings_fact
WHERE 
    salary_year_avg >= 65000;
```

### Exercise 10: Missing Data Evaluation (IS NULL)
Goal: Identify all job postings that contain neither an annual average salary nor an hourly average salary to find postings without compensation details.
```sql
SELECT
    job_id,
    job_title,
    salary_year_avg,
    salary_hour_avg
FROM
    job_postings_fact
WHERE
    salary_year_avg IS NULL
    AND salary_hour_avg IS NULL;
```

---

## Column Renaming and Custom Formatting (AS)

### Exercise 11: Custom Column Aliasing
Goal: Clean up result headers by providing descriptive temporary names to fields.
```sql
SELECT 
    job_id,
    job_title_short,
    job_location,
    job_via AS job_posted_site,
    job_posted_date,
    salary_year_avg AS avg_yearly_salary
FROM 
    job_postings_fact;
```

---

## Wildcard Pattern Matching (LIKE)

### Exercise 12: Single Character Match Wildcard (_)
Goal: Locate companies with names including Tech followed by exactly one trailing character.
```sql
SELECT name
FROM company_dim
WHERE name LIKE '%Tech_';
```

### Exercise 13: Trailing Character String Wildcards
Goal: Match jobs where the keyword Engineer is followed by exactly one character at the end of the text string.
```sql
SELECT 
    job_id,
    job_title,
    job_posted_date
FROM 
    job_postings_fact
WHERE 
    job_title LIKE '%Engineer_';
```

### Exercise 14: Multi-Pattern Inclusive and Exclusive Filters
Goal: Look for non-senior Data or Business Analyst roles while excluding titles containing Senior.
```sql
SELECT 
    job_title,
    job_location AS location,
    salary_year_avg AS salary
FROM 
    job_postings_fact
WHERE 
    (job_title LIKE '%Data%' OR job_title LIKE '%Business%') 
    AND job_title LIKE '%Analyst%' 
    AND job_title NOT LIKE '%Senior%';
```

---

## Multi-Conditional Priority Logic (AND / OR)

### Exercise 15: Tiered Role Scoping with Bracket Precedence
Goal: Locate target positions within specified geographical limits meeting separate, unique minimum salary expectations.
```sql
SELECT 
    job_id,
    job_title_short,
    job_location,
    job_via,
    salary_year_avg
FROM 
    job_postings_fact
WHERE 
    job_location IN ('Boston, MA', 'Anywhere') 
    AND (
        (job_title_short = 'Data Analyst' AND salary_year_avg > 100000) 
        OR 
        (job_title_short = 'Business Analyst' AND salary_year_avg > 70000)
    );
```

---

## Mathematical and Arithmetic Operations

### Exercise 16: Column Modifications (Addition and Subtraction)
Goal: Calculate shifted invoice rates based on a baseline original value.
```sql
SELECT
    project_company,
    nerd_id,
    nerd_role,
    hours_rate AS rate_original,
    hours_rate - 5 AS rate_drop,
    hours_rate + 5 AS rate_hike
FROM
    invoices_fact;
```

### Exercise 17: Column Addition
Goal: Combine hourly values and billing rates to generate a unified custom priority metric.
```sql
SELECT
    hours_spent + hours_rate AS priority_value
FROM
    invoices_fact;
```

### Exercise 18: Column Division
Goal: Divide hours spent by hourly rates to calculate operational efficiency metrics per activity.
```sql
SELECT
    activity_id,
    hours_spent / hours_rate AS efficiency
FROM
    invoices_fact;
```

---

## SQL Aggregations and Group Summaries (SUM, AVG, COUNT, GROUP BY, HAVING)

### Exercise 19: Global Dataset Aggregations
Goal: Determine total summary metrics including sum, average, total rows, and total unique job titles.
```sql
SELECT
    SUM(salary_year_avg) AS salary_sum,
    AVG(salary_year_avg) AS salary_avg,
    COUNT(*) AS count_rows,
    COUNT(DISTINCT job_title_short) AS job_type_total
FROM
    job_postings_fact;
```

### Exercise 20: Filtered Counts
Goal: Quantify total job postings offering health insurance benefits.
```sql
SELECT
    COUNT(*) AS total_jobs_with_health_insurance
FROM
    job_postings_fact
WHERE
    job_health_insurance = TRUE;
```

### Exercise 21: Manual Average Logic with Logical Filters
Goal: Calculate remote job average salaries by manually dividing total salary sums by row counts.
```sql
SELECT
    SUM(salary_year_avg) / COUNT(salary_year_avg) AS average_salary
FROM
    job_postings_fact
WHERE
    job_work_from_home = TRUE
    AND salary_year_avg IS NOT NULL;
```

### Exercise 22: Group Summaries Sorted by Volume (GROUP BY)
Goal: Aggregate job counts by country and order the results by the resulting size volume.
```sql
SELECT
    job_country,
    COUNT(job_id) AS job_posting_count
FROM
    job_postings_fact
GROUP BY
    job_country
ORDER BY
    job_posting_count;
```

### Exercise 23: Group Filtering with Aggregate Criteria (GROUP BY and HAVING)
Goal: Aggregate metrics by role title, keeping groups with over 10 postings, sorted by average salary.
```sql
SELECT
    job_title_short AS jobs,
    COUNT(job_title_short) AS job_count,
    AVG(salary_year_avg) AS salary_avg,
    MIN(salary_year_avg) AS salary_min,
    MAX(salary_year_avg) AS salary_max
FROM
    job_postings_fact
GROUP BY
    job_title_short
HAVING
    COUNT(job_title_short) > 10
ORDER BY
    salary_avg;
```

### Exercise 24: Aggregate Math Groupings
Goal: Calculate the current month's total cost (hours spent multiplied by hour rate) per project alongside a projected cost forecast where the baseline hourly rate increases by $5.
```sql
SELECT
    project_id,
    SUM(hours_spent * hours_rate) AS project_original_cost,
    SUM(hours_spent * (hours_rate + 5)) AS project_projected_cost
FROM
    invoices_fact
GROUP BY
    project_id;
```

---

## Relational Database Joins (INNER JOIN and LEFT JOIN)
Goal: Connect the job postings fact table with the company dimensions table to retrieve a clean listing of job titles and their corresponding company names matching the phrase Data Scientist.

```sql
SELECT
    j.job_title,
    c.name AS company_name
FROM
    job_postings_fact AS j
INNER JOIN company_dim AS c ON j.company_id = c.company_id
WHERE
    j.job_title LIKE '%Data Scientist%';
```

### Exercise 26: Multi-Table Joins with Group Aggregations
Goal: Connect skills data, bridge references, and main job records via LEFT JOINs to count total job postings and calculate the average salary associated with each distinct skill name, ordered highest to lowest.

```sql
SELECT
    skills.skills AS skill_name,
    COUNT(skills_to_job.job_id) AS number_of_job_postings,
    AVG(job_postings_fact.salary_year_avg) AS average_salary_for_skill
FROM
    skills_dim AS skills
LEFT JOIN skills_to_job ON skills.skill_id = skills_to_job.skill_id
LEFT JOIN job_postings_fact ON skills_to_job.job_id = job_postings_fact.job_id
GROUP BY
    skills.skills
ORDER BY
    average_salary_for_skill DESC;
```


















