# Introduction
This project showcases the use of SQL in diving into job postings. The focus is to explore open Data Analyst roles by reviewing top salaries, in-demand skills, and where high-demand meets higher salary in Data Analytics.

The SQL queries used in this project can be found here: [project_sql](/project_sql/).
# Background
As an aspiring Data Analyst, I wanted to create a project that showcases my knowledge of SQL. For this project in particular, I participated in an online video tutorial that was made by Luke Barousse (https://lukebarousse.com/sql), an extremely knowledgable Data Analyst who shares his experience with subscribers. 

The goal of the project was initiated by a desire to navigate the data analyst job market more effectively by targeting top-paid and in-demand skills for Data Analysts.

The datasets that are utilized in the project were provided by Luke and contain insights on job titles, salaries, locations, and essential skills.

## Questions answered through the use of SQL queries:

    1. What are the top-paying data analyst jobs?
    2. What skills are required for these top-paying jobs?
    3. What skills are most in-demand for data analysts?
    4. Which skills are associated with higher salaries?
    5. What are the most optimal skills to learn?
# Tools I Used
To fully investigate insights of the data analyst job market, I utilized the power of the following key tools:

- **SQL**: The foundation of the analysis that allowed me to query the database to uncover multiple critical insights.
- **PostgreSQL**: The recommended database management system that was ideal for handling the job posting data.
- **Visual Studio Code**: A terrific tool for database management and execution of the SQL queries.
- **Git & GitHub**: As I learned, this was essential for version control as well as sharing of the SQL scripts and analysis, and ultimately the project.
# The Analysis
Each query for this project aimed at investigating specific aspects of the data analyst job market. Below is how I approached each question:

## 1. Top Paying Data Analyst Jobs
In order to identify the highest-paying roles, I filtered data analyst positions by average yearly salary and location, focusing on remote jobs. This query highlights the high paying opportunities in the field.
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
JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY salary_year_avg DESC
LIMIT 10;
```
### Key Takeaways
- **Wide Salary Range:** There is a significant range of salaries among the top 10 most highly paid Data Analyst roles, ranging from $184,000 to $650,000. This shows notable salary potential in the field.
- **Diverse Employers:** Companies such as SmartAsset, Meta, and AT&T are among those offering high salaries, showing a broad interest across different industries.
- **Job Title Variety:** There's a high diveristy in job titles, from Data Analyst to Director of Analytics, reflecting varied roles and specializations within data analytics.

## 2. Top Paying Data Analyst Job Skills
To identify the skills associated with the top paying jobs, I used the top paying jobs query from #1 in combination with the skills dimension tables. 

```sql
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM  
        job_postings_fact
    JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_title_short = 'Data Analyst' AND
        job_location = 'Anywhere' AND
        salary_year_avg IS NOT NULL
    ORDER BY salary_year_avg DESC
    LIMIT 10
)

SELECT 
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
```
### Key Takeaways
- **Range of Skills:** The top paying jobs listed multiple skills and programming languages, suggesting applicants with well-rounded knowledge would benefit from seeking these positions.
- **Prevalance of SQL:** SQL was listed as a desired skill in all of the top paying jobs. 
- **Other Data-related Skills:** In addition to SQL, skills such as Python, Excel, and Tableau were frequently listed as desired skills to know.

## 3. Top Demanded Skills
Since the focus of the project was on remote Data Analyst positions, it was important to determine which skills were listed most frequently for these positions. In order to do this, I joined the fact table with the skills dimension tables to produce a count of the top skills listed for remote data analyst roles.

```sql
SELECT
    skills,
    COUNT(job_postings_fact.job_id) AS skills_count
FROM job_postings_fact
INNER JOIN 
    skills_job_dim ON skills_job_dim.job_id = job_postings_fact.job_id
INNER JOIN
    skills_dim ON skills_dim.skill_id = skills_job_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere'
GROUP BY
    skills
    ORDER BY skills_count DESC
LIMIT 5
```

### Key Takeaways
- **SQL Tops the List:** SQL was by far the most in-demand skill for remote Data Analyst positions, followed by Python.
- **Visualization is Key:** Also among the top 5 in-demand skills were tableau and Power BI, suggesting companies that are hiring for remote Data Analysts would like to see an applicant that is able to produce visualizations and reports based on their analysis.

## 4. Top Paying Skills
As part of this project, it was also important to analyze which skills were associated with the highest average salaries for remote Data Analyst roles.

```sql
SELECT
    skills,
    ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN 
    skills_job_dim ON skills_job_dim.job_id = job_postings_fact.job_id
INNER JOIN
    skills_dim ON skills_dim.skill_id = skills_job_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    salary_year_avg IS NOT NULL AND
    job_work_from_home = True
GROUP BY
    skills
ORDER BY 
    avg_salary DESC
LIMIT 25
```

### Key Takeaways
- **Specialized Skills:** The largest average salaries were associated with niche skills such as pyspark and bitbucket. 
- **Need for Numbers:** Although this query showcased the highest-paying skills, there was a flaw in that it did not show how many job postings listed these skills. This was rectified by the next query...

## 5. Optimal Skills
This query used a combination of #3 and #4 as temporary tables in order to find the optimal skills for remote data analyst roles. It allowed me to see the average yearly salary as well as the demand count for each skill.

```sql
WITH skills_demand AS (
    SELECT
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(job_postings_fact.job_id) AS demand_count
    FROM job_postings_fact
    INNER JOIN 
        skills_job_dim ON skills_job_dim.job_id = job_postings_fact.job_id
    INNER JOIN
        skills_dim ON skills_dim.skill_id = skills_job_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst' 
        AND salary_year_avg IS NOt NULL 
        AND job_work_from_home = True
    GROUP BY
        skills_dim.skill_id
), average_salary AS (
    SELECT
        skills_job_dim.skill_id,
        ROUND(AVG(salary_year_avg), 0) AS avg_salary
    FROM job_postings_fact
    INNER JOIN 
        skills_job_dim ON skills_job_dim.job_id = job_postings_fact.job_id
    INNER JOIN
        skills_dim ON skills_dim.skill_id = skills_job_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst' AND
        salary_year_avg IS NOT NULL AND
        job_work_from_home = True
    GROUP BY
        skills_job_dim.skill_id
)
SELECT 
    skills_demand.skill_id,
    skills_demand.skills,
    demand_count,
    avg_salary
FROM skills_demand
INNER JOIN  
    average_salary ON skills_demand.skill_id = average_salary.skill_id
WHERE 
    demand_count > 10
ORDER BY    
    avg_salary DESC,
    demand_count DESC 
LIMIT 25
```
### Key Takeaways
- **High Demand Programming Languages:** Python and R stand out for their high demand, with respective job postings of 236 and 148. The salaries for these languages also indicate that they are highly valued with average yearly salaries of $101,397 and $100,499, respectively.
- **BI and Visualization Tools:** Both Tableau and Power BI, with respective demand counts of 230 and 110, highlight the importance of data visualization and business intelligence in extracting insights from data.



# What I Learned
**Advanced SQL Queries:** Throughout this project, I was able to refresh my knowledge of SQL while also learning to incorporate JOINs and WITH clauses that take my advanced SQL queries to the next level.

**Storytelling:** I was able to gain insights from raw data and turn them into a compelling story that can in turn be used to make thoughtful decisions.

**New Tools:** Having never used VS Code prior to this project, it has now become one of my favorite tools in analyzing data.

# Conclusions
### Insights
Several insights emerged from the analysis of this data:

**Top-Paying Data Analyst Jobs:** There are plenty of high-paying remote opportunities in Data Analytics, with the highest salary at $650,000.

**Top-Paying Job Skills:** SQL stands out as a required skill for potential Data Analysts looking to land a high-paying analyst role.

**Top Demanded Skills:** Among the most in-demand skills are SQL, Excel, and Python, indicating a solid foundation of basic data driven programming knowledge is key in landing a role as a Data Analyst.

**Top Paying Skills:** Specialized skills are required for the roles with the highest average salaries, suggesting value in knowledge of niche expertise.

**Optimal Skills:** SQL once again shows its value as one of the most in-demand skills for roles paying higher average salaries. This emphasizes how important knowing SQL can be for aspiring Data Analysts looking to maximize their market vlue.

## Closing Thoughts
Throughout this project, I was able to employ my knowledge of basic SQL to filter and interpret the job posting data, while also utilizing new techniques and queries to dive even deeper into the data. The results of this project can serve as a template for which skills to focus on going forward in my quest to land a Data Analyst position. 
