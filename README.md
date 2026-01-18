TODO: Update contents of this later

# INTRODUCTION

Purpose of this project is to gain some experiance in SQL world and also to dive into data job market and explore it!
The main focus was on data scientist area, top-paying roles and skills, in-demanded skills and also where high demand meets
high salary in data world.

Check out SQL queries: [project_sql folder](/project_sql/)

# BACKGROUND

## The question I wanted to answer through this project were:

    1. What are the top-paying data scientist jobs?
    2. What skills are required for these top-paying jobs?
    3. What skills are most in-demand for data scientists?
    4. Which skills are associated with higher salaries?
    5. What are the most optimal skills to learn?

# TOOLS I USED

- **SQL**: Allowed me to query the database and identify data-driven insights.
- **Visual Studio Code**: My go-to for database management and executing SQL queries.
- **PostgreSQL**: The chosen database management system.
- **Git & GitHub**: Essential for version control and sharing my analysis.

# THE ANALYSIS

## 1. Top-Paying Data Scientist Jobs

This analysis identifies the highest-paying Data Scientist positions that are fully remote. I filtered job postings to include only Data Scientist roles with a non-null annual salary and a job location marked as “Anywhere.” By joining the job postings with company data and ordering results by salary in descending order, I extracted the top 10 highest-paying roles, highlighting job titles, companies, locations, and salary levels.

```sql
SELECT
    job_id,
    job_title,
    job_location,
    salary_year_avg,
    company_dim.name AS company_name
FROM 
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE 
    job_title_short = 'Data Scientist' AND 
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY salary_year_avg DESC
LIMIT 10;
```
Top 10 paying data scientist roles span from $300,000 to $550,000, indicating significant salary potential in the field.

![Top-Paying Data Scientist Jobs](assets\top_paying_remote_data_scientist_roles.png)


## 2. Skills Required for Top-Paying Data Scientist Jobs

In this step, I analyzed the skill requirements for the top-paying Data Scientist positions identified previously. Using a Common Table Expression (CTE), I first isolated the top 10 highest-paying remote Data Scientist jobs. I then joined this dataset with skill mapping tables to retrieve all associated skills. This approach allowed me to understand which technical skills are most commonly required for high-salary Data Scientist roles.

```sql
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        company_dim.name AS company_name
    FROM 
        job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE 
        job_title_short = 'Data Scientist' AND 
        job_location = 'Anywhere' AND
        salary_year_avg IS NOT NULL
    ORDER BY salary_year_avg DESC
    LIMIT 10
)

SELECT 
    top_paying_jobs.job_id,
    top_paying_jobs.job_title,
    top_paying_jobs.salary_year_avg,
    top_paying_jobs.company_name,
    skills_dim.skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY top_paying_jobs.salary_year_avg DESC;
```

## 3. Most In-Demand Skills for Data Scientists

This analysis focuses on overall skill demand across all Data Scientist job postings. By joining job postings with skill data, I counted how frequently each skill appears in Data Scientist roles. The results were grouped and ranked by demand count, revealing the top five most requested skills in the Data Scientist job market, regardless of salary level.

```sql
SELECT
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE 
    job_title_short = 'Data Scientist'
GROUP BY
    skills
ORDER BY 
    demand_count DESC
LIMIT 5
```
![In-Demand Skills:](assets\top_in_demand_skills.png)

## 4. Skills Associated with Higher Salaries

Here, I examined how individual skills correlate with higher salaries. I filtered for Data Scientist roles with available salary data and calculated the average annual salary for each skill. By ordering skills by their average salary, this analysis highlights which skills are most strongly associated with higher-paying Data Scientist positions.

```sql
SELECT 
    skills,
    ROUND(AVG(job_postings_fact.salary_year_avg),0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Scientist' AND
    salary_year_avg IS NOT NULL
GROUP BY skills
ORDER BY avg_salary DESC
LIMIT 25
```
| Skill         | Average Salary |
|---------------|---------------|
| asana         | 215477        |
| airtable      | 201143        |
| redhat        | 189500        |
| watson        | 187417        |
| elixir        | 170824        |
| lua           | 170500        |
| slack         | 168219        |
| solidity      | 166980        |
| ruby on rails | 166500        |
| rshiny        | 166436        |


## 5. Most Optimal Skills to Learn 

In the final analysis, I combined skill demand and salary data to identify the most optimal skills for career growth. Using two CTEs, I calculated both the demand count and average salary for each skill. By joining these results and ranking skills by demand and salary, I identified skills that offer the best balance between market demand and earning potential, making them strong candidates for focused learning.

```sql
WITH skills_demand AS (
    SELECT
        skills_dim.skill_id, 
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE 
        job_title_short = 'Data Scientist' AND
        salary_year_avg IS NOT NULL
    GROUP BY 
        skills_dim.skill_id
), average_salary AS (
    SELECT
        skills_dim.skill_id, 
        skills,
        ROUND(AVG(salary_year_avg),0) AS avg_salary
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE 
        job_title_short = 'Data Scientist' AND
        salary_year_avg IS NOT NULL
    GROUP BY 
        skills_dim.skill_id
)

SELECT
    skills_demand.skill_id,
    skills_demand.skills,
    demand_count,
    avg_salary
FROM
    skills_demand
INNER JOIN average_salary ON skills_demand.skill_id = average_salary.skill_id
ORDER BY
    demand_count DESC,
    avg_salary DESC
LIMIT 25
```
The most optimal skills to learn (high demand and a high-paying skills) 4 out of 5 skills from top 5 in-demanded: 
    ![Optimal Skills: ](assets\optimal_skills_demand_vs_salary.png)
             
AWS emerges as a high-paying skill, suggesting that cloud expertise significantly enhances earning potential.

# WHAT I LEARNED

**SQL Skills:** How to write complex queries using JOINs, CTEs, and aggregations to extract insights from real-world job posting data.

**Skill & Salary Analysis:** How to analyze demand and salary patterns to see which skills are both popular and high-paying.

**Data Integration:** How to combine multiple datasets (jobs, companies, skills) to answer meaningful business questions.

**Data Visualization:** How to present data effectively using charts to communicate trends in skills, salaries, and job roles.

**Career Insights:** How to extract actionable insights on which skills to focus on for growth as a Data Scientist.

# CONCLUSIONS

### Insights
From the analysis, these are several insights emerged:

1. **Top-Paying Data Scientist Jobs:** The highest-paying jobs for data scientist that allow remote work offer a wide range of salaries, the highest at $550,000!
2. **Skills for Top-Paying Jobs:** High-paying data scientist jobs require advanced proficiency in SQL and Python, suggesting it’s a critical skill for earning a top salary.
3. **Most In-Demand Skills:** Python is also the most demanded skill in the data scientist job market.
4. **Skills with Higher Salaries:** Specialized skills, such as Asana and Airtable, are associated with the highest average salaries, indicating a premium on niche expertise.
5. **Optimal Skills for Job Market Value:** Python leads in demand and offers for a high average salary, positioning it as one of the most optimal skills for data scientists to learn to maximize their market value, but don't leave out SQL, R and Tableau.



