# 📊 Data Analyst Job Market Analysis (SQL Project)

## 📌 Introduction
Driven by a quest to navigate the data analyst job market effectively, this project explores **top-paying jobs**, **in-demand skills**, and **where high demand meets high salary** in data analytics.

The analysis uses real-world tech job postings to evaluate what employers are seeking and paying for in data roles.

---

## 🎯 Questions Addressed
1. **Top-Paying Jobs:** What are the highest-paying remote Data Analyst roles?
2. **Skills for High Pay:** Which skills are required for these top-paying roles?
3. **Most Demanded Skills:** What are the most in-demand skills for Data Analysts overall?
4. **Top Skills by Salary:** Which skills command the highest average compensation?
5. **Optimal Skills to Learn:** Which skills offer the highest market demand combined with top pay?

---

## 🛠️ Tools & Technologies
- **SQL (PostgreSQL):** Primary query engine used for data extraction, aggregations, CTEs, and window logic.
- **DBeaver / VS Code:** Database management and SQL query development.
- **Git & GitHub:** Version control, project tracking, and portfolio hosting.

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

Key Findings:Top 10 salaries range from $184,000 to $650,000, showing extreme upside for specialized and senior analyst positions.High compensation is not exclusive to Big Tech; financial institutions, healthcare firms, and startups offer competitive pay for remote talent.2. Skills Required for Top-Paying JobsExamines the specific tools and languages listed by the highest-paying roles.SQLWITH top_paying_jobs AS (
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
Key Findings:SQL is required in 8 of the 10 top postings.Python (7/10) and Tableau (6/10) are the dominant companion tools for premium roles.3. Most In-Demand Skills for Data AnalystsCalculates posting frequency across all remote Data Analyst roles.SQLSELECT 
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
SkillDemand CountSQL7,291Excel4,611Python4,330Tableau3,745Power BI2,609Key Findings:Core data querying (SQL) and spreadsheet analysis (Excel) remain fundamental entry tickets.Modern analysts are expected to pair querying with a visualization layer (Tableau / Power BI) and programming (Python).4. Top Skills Based on SalaryIdentifies which technical skills command the highest average annual salaries.SQLSELECT 
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
SkillAverage Salary ($)PySpark$208,172Bitbucket$189,155Couchbase$160,515Watson$160,515DataRobot$155,486GitLab$154,500Swift$153,750Jupyter$152,777Pandas$151,821Elasticsearch$145,000Key Findings:Big Data & Distributed Computing: PySpark tops the list above $200k/year.Engineering Crossover: Version control and CI/CD tools (GitLab, Bitbucket) command high wages, reflecting demand for analysts who follow software development practices.5. Most Optimal Skills to Learn (High Demand + High Pay)Pinpoints the sweet spot by filtering for skills with high posting volume (>10 mentions) and ordering by average salary.SQLWITH skills_demand AS (
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
📈 Strategic TakeawaysThe Core Stack: Prioritize SQL and Excel first—they appear in the overwhelming majority of job openings.The Growth Lever: Add Python and Tableau to cross into six-figure roles and expand into advanced reporting.The Salary Multiplier: Learn big data and cloud tools (PySpark, Snowflake, Databricks) to stand out in senior, high-paying enterprise analytics teams
