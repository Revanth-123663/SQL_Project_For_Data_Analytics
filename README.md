# 📊 Data Analyst Job Market Analysis (SQL Project)

## 📌 Introduction
Driven by a quest to navigate the data analyst job market effectively, this project explores **top-paying jobs**, **in-demand skills**, and **where high demand meets high salary** in data analytics.

The analysis is based on real-world tech job postings to evaluate what employers are seeking and paying for in data roles.

---

## 🎯 Questions Addressed
1. **Top-Paying Jobs:** What are the highest-paying remote Data Analyst roles?
2. **Skills for High Pay:** Which skills are required for these top-paying roles?
3. **Most Demanded Skills:** What are the most in-demand skills for Data Analysts overall?
4. **Top Skills by Salary:** Which skills command the highest average compensation?
5. **Optimal Skills to Learn:** Which skills offer the highest market demand combined with top pay?

---

## 🛠️ Tools Used
- **SQL (PostgreSQL):** Primary query engine used for data extraction, aggregations, CTEs, and filtering.
- **VS Code / DBeaver:** Database management and query execution.
- **Git & GitHub:** Version control, documentation, and portfolio hosting.

---

## 🔍 The Analysis

### 1. Top-Paying Data Analyst Jobs
Identifies the top 10 highest-paying remote Data Analyst positions with documented salaries.

```sql
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim 
    ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' 
    AND job_location = 'Anywhere' 
    AND salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10;
```

**Key Findings:**
- Top 10 remote salaries range from **$184,000 to $650,000**, showing extreme upside for specialized and senior analyst roles.
- High compensation is not exclusive to Big Tech; startups, financial institutions, and healthcare firms offer competitive remote salaries.

---

### 2. Skills Required for Top-Paying Jobs
Examines the specific tools and languages listed across the top 10 highest-paying roles.

```sql
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM
        job_postings_fact
    LEFT JOIN company_dim 
        ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_title_short = 'Data Analyst' 
        AND job_location = 'Anywhere' 
        AND salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)
SELECT 
    top_paying_jobs.*,
    skills_dim.skills
FROM 
    top_paying_jobs
INNER JOIN skills_job_dim 
    ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim 
    ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC;
```

**Key Findings:**
- **SQL** is required in 8 of the 10 top postings.
- **Python** (7/10) and **Tableau** (6/10) are the dominant companion tools for premium roles.

---

### 3. Most In-Demand Skills for Data Analysts
Calculates posting frequency across all remote Data Analyst roles.

```sql
SELECT 
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM 
    job_postings_fact
INNER JOIN skills_job_dim 
    ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim 
    ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_postings_fact.job_title_short = 'Data Analyst'
    AND job_postings_fact.job_location = 'Anywhere'
GROUP BY
    skills_dim.skills
ORDER BY
    demand_count DESC
LIMIT 5;
```

| Skill | Demand Count |
|---|---|
| **SQL** | 7,291 |
| **Excel** | 4,611 |
| **Python** | 4,330 |
| **Tableau** | 3,745 |
| **Power BI** | 2,609 |

**Key Findings:**
- **SQL** and **Excel** remain fundamental baseline requirements for analyst positions.
- Pairing data manipulation (**Python**) with visual storytelling (**Tableau / Power BI**) covers the vast majority of market demand.

---

### 4. Top Skills Based on Salary
Identifies which technical skills command the highest average annual salaries.

```sql
SELECT 
    skills_dim.skills,
    ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
FROM 
    job_postings_fact
INNER JOIN skills_job_dim 
    ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim 
    ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_postings_fact.job_title_short = 'Data Analyst'
    AND job_postings_fact.salary_year_avg IS NOT NULL
    AND job_postings_fact.job_location = 'Anywhere'
GROUP BY
    skills_dim.skills
ORDER BY
    avg_salary DESC
LIMIT 10;
```

| Skill | Average Salary ($) |
|---|---|
| **PySpark** | $208,172 |
| **Bitbucket** | $189,155 |
| **Couchbase** | $160,515 |
| **Watson** | $160,515 |
| **DataRobot** | $155,486 |
| **GitLab** | $154,500 |
| **Swift** | $153,750 |
| **Jupyter** | $152,777 |
| **Pandas** | $151,821 |
| **Elasticsearch** | $145,000 |

**Key Findings:**
- **Big Data & Distributed Computing:** PySpark tops the list above $200,000/year.
- **Engineering Crossover:** Version control and CI/CD tools (GitLab, Bitbucket) command high wages, reflecting demand for analysts who follow software engineering practices.

---

### 5. Most Optimal Skills to Learn (High Demand + High Pay)
Pinpoints the intersection of high demand (>10 postings) and above-average compensation.

```sql
WITH skills_demand AS (
    SELECT 
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM 
        job_postings_fact
    INNER JOIN skills_job_dim 
        ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim 
        ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        AND job_location = 'Anywhere'
    GROUP BY
        skills_dim.skill_id,
        skills_dim.skills
), 
average_salary AS (
    SELECT 
        skills_job_dim.skill_id,
        ROUND(AVG(salary_year_avg), 0) AS avg_salary
    FROM 
        job_postings_fact
    INNER JOIN skills_job_dim 
        ON job_postings_fact.job_id = skills_job_dim.job_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        AND job_location = 'Anywhere'
    GROUP BY
        skills_job_dim.skill_id
)
SELECT 
    skills_demand.skill_id,
    skills_demand.skills,
    skills_demand.demand_count,
    average_salary.avg_salary
FROM 
    skills_demand
INNER JOIN average_salary 
    ON skills_demand.skill_id = average_salary.skill_id
WHERE
    skills_demand.demand_count > 10
ORDER BY
    average_salary.avg_salary DESC,
    skills_demand.demand_count DESC
LIMIT 10;
```

---

## 📈 Strategic Takeaways
1. **The Core Stack:** Prioritize **SQL** and **Excel** first—they represent the largest volume of hiring needs.
2. **The Growth Lever:** Learn **Python** and **Tableau** to access senior-level opportunities and automated workflows.
3. **The Salary Multiplier:** Specialize in big data and cloud platforms (**PySpark, Snowflake, Databricks**) to capture top-tier enterprise compensation.

---

## 🚀 How to Run
1. Clone this repository to your local machine:
   ```bash
   git clone [https://github.com/](https://github.com/)<YOUR_USERNAME>/sql_project_data_job_analysis.git
   ```
2. Initialize database schema:
   - Run `sql_load/1_create_database.sql`
   - Run `sql_load/2_create_tables.sql`
3. Load the dataset CSV files into PostgreSQL.
4. Execute individual query scripts inside the `project_sql/` directory.
