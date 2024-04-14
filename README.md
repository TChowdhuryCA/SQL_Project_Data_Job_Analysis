# Introduction

This project is the capstone project for the "SQL for Data Analytics" course by [Luke Barousse](https://www.lukebarousse.com/sql). 

It looks into the job market for data analysts using 2023 data and explores the types of jobs available, the salary range, the skill requirements for the jobs and the employers posting the jobs.

# Background

This project was taken on after completing the Google Data Analyst certification as a dive into exploring different SQL tools and a deeper understanding of the query language.

In this project, a few key questions are answered about the 'Data Analyst' job market, creating a guideline to grow in the career path. 

The Questions are:

1. What are the top paying jobs for my role?
2. What are the skills required for these top-paying roles?
3. What are the most in-demand skills for my role?
4. What are the top skills based on salary for my role?
5. What are the most optimal skills to learn?

# Tools I used

The methodology for this data analysis project has been exclusively the SQL query language. The following tools were used:

- PostgreSQL: The database server of choice for the data.
- Visual Studio Code: The code editor I have used for executing and managing the queries.
- Github: Used to upload and share the analysis along with the queries.

# The Analysis

In order to tackle the job market as a new data analyst, the job environment was dissected using five questions.

### **What are the top paying jobs for my role?**

The following query was used to find the answer to this question:

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
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE 
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10;
```
Based on the results of the query, the following observations were made:

- The highest-paid data analyst position is offered by Mantys with an annual salary of $650,000.
- Most of the top-paying positions are full-time and offer salaries ranging from $184,000 to $650,000 per year.
- Companies offering these high salaries include well-known names like Meta, AT&T, and Pinterest Job Advertisements.
- The highest-paid positions are diverse, ranging from entry-level data analyst roles to director-level positions.
- The distribution of job postings across the year is relatively uniform, with postings spread throughout the year, indicating a consistent demand for highly paid data analyst roles.
- Remote or hybrid work arrangements are common among these high-paying positions, reflecting the increasing trend towards flexible work arrangements in the data analysis field.

The bar chart shows the salary comparison among the top 10 postings.

![salary_comparison](assets\q1_viz.jpg)

### **What are the skills required for these top-paying roles?**

For answering this question, the following query was used:
```sql
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM
        job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE 
        job_title_short = 'Data Analyst' AND
        job_location = 'Anywhere' AND
        salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)

SELECT
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY salary_year_avg DESC
```
Based on the results of the query, I have generated a graph showing the frequency of skill requirements for the top jobs.

![top_skills](assets\q2_viz.jpg)

We can observe that:
- SQL is a common requirement across all the top-paying data analyst jobs, indicating its importance in data analysis roles.
- Python is another widely sought-after skill, being required by most of the job listings.
- Other commonly requested skills include R, Excel, Tableau, AWS, and Azure, showcasing the diverse toolset expected from data analysts.
- Skills such as Git, Bitbucket, Atlassian, Jira, and Confluence are also mentioned, emphasizing the significance of version control and collaboration tools in data analysis projects.
- The variety of skills required reflects the multidisciplinary nature of data analysis roles, which often involve tasks ranging from data manipulation and visualization to statistical analysis and software development.


### **What are the most in-demand skills for my role?**

Now that we have found the skills required for the top paying jobs, it is also important to find out which skills provide the most opportunities. The following query was run to find this.

```sql
SELECT
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_work_from_home = TRUE
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 5;
```
The results here are very similar to the previous question, with SQL leading the way.

| **Skill**   | **Demand Count** |
|-------------|------------------|
| SQL         | 7291             |
| Excel       | 4611             |
| Python      | 4330             |
| Tableau     | 3745             |
| Power BI    | 2609             |

The top skills remain somewhat the same with an exception of Power BI ranking much higher.

### **What are the top skills based on salary for my role?**

When we compare the skills directly to the highest salaries, we get a different pattern compared to what we have been getting till now.

The following code was used:

```sql
SELECT
    skills,
    ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    salary_year_avg IS NOT NULL
GROUP BY
    skills
ORDER BY
    avg_salary DESC
LIMIT 25;
```

The results:
![highest_pay_skills](assets\avg_salary_vs_skills.jpg)


1. SVN (Subversion): Data analysts with expertise in SVN earn an average salary of $400,0001.
2. Solidity: Professionals skilled in Solidity, a programming language for smart contracts on the Ethereum blockchain, have an average salary of $179,0001.
3. Couchbase: Data analysts proficient in Couchbase, a NoSQL database, earn an average salary of $160,5151.
4. DataRobot: Those experienced with DataRobot, an automated machine learning platform, have an average salary of $155,4861.
5. Go (Golang): Golang experts earn an average salary of $155,0001.

As we can see in the chart, the highest paid skills are very specialized in nature and the results we have seen before are no longer visible.

### **What are the most optimal skills to learn?**

While the above skills are the highest paying, the market is too niche to specialize, therefore the following query was created to show high paying skills that also have an actionable level of demand.

```sql
SELECT
    skills_dim.skill_id,
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count,
    ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = TRUE
GROUP BY
    skills_dim.skill_id
HAVING
    COUNT(skills_job_dim.job_id) > 25
ORDER BY
    avg_salary DESC,
    demand_count DESC
LIMIT 25;
```
The geographical restrictions are lifted by showing only jobs that are remote, and the likelihood of employment has been significantly raised by displaying only skills with at least 25 job postings.

| **Skill**   | **Demand Count** | **Average Salary** |
|-------------|------------------|--------------------|
| Go          | 27               | $115,320           |
| Snowflake   | 37               | $112,948           |
| Azure       | 34               | $111,225           |
| AWS         | 32               | $108,317           |
| Oracle      | 37               | $104,534           |
| Looker      | 49               | $103,795           |
| Python      | 236              | $101,397           |
| R           | 148              | $100,499           |
| Tableau     | 230              | $99,288            |
| SAS         | 63               | $98,902            |
| SQL Server  | 35               | $97,786            |
| Power BI    | 110              | $97,431            |
| SQL         | 398              | $97,237            |
| Flow        | 28               | $97,200            |
| PowerPoint  | 58               | $88,701            |
| Excel       | 256              | $87,288            |
| Sheets      | 32               | $86,088            |
| Word        | 188              | $85,000            |

Here we can see that more familiar skills like Python, R and Tableau are on the list while the top positions are dominated by the cloud platform skills.

# What I Learned

From doing this project I have developed a robust understanding of working with multiple files using the JOIN function. This helped me analyze data sets using information that is spreadout in different csv files, thus having deeper insights.

I have also polished my ability to add layers to my analysis using CTEs.

Besides the core skills, I have also learned tools like Git, Github, PostgreSQL and VS Code. This gave me an understanding of a wider range of SQL tools and taught me how to share my analysis effectively.

# Conclusion

From all the querys we have run and the analysis we have performed, we can deduce that SQL is the most important skill in the Data Analysis space. It is the most common skill among jobs starting from entry level to director level.

While SQL provides a core skill that can be used throughout the career, more specialized skills such as cloud tools help get a higher pay down the line.