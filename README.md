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
