# HR Employee Distribution Report

## Purpose Of The Project

The project aims to analyze employee data, including birthdate, hire date, termination date, age, gender, first name, last name, department, and location state. Insights will inform workforce demographics, talent management, performance analysis, strategic planning, and geographical distribution.


![Screenshot 2024-05-24 133053](https://github.com/Kanakgiri/HR-Employee-Distribution-Report/assets/171118310/5e4860fd-cd2b-494b-b9e8-6701dc6230a6)
![Screenshot 2024-05-24 133114](https://github.com/Kanakgiri/HR-Employee-Distribution-Report/assets/171118310/befed080-5a0f-44f0-80c6-0f403eedf81a)



## Data Used
- Data - HR Data with over 22000 rows from the year 2000 to 2020.

## Tools Used

- Microsoft Excel
- Power BI
- MySQL Workbench

## Approach Used

- Feature Engineering: This will help us generate some new columns from existing ones using Power Query.

The column `Age` is added using the column `birthdate`.

``` update hr
set age = timestampdiff(year, birthdate, curdate());
```

In some cases the age of the employees was shown with a Negative sign because the birthdate was 100 Years more than the actual date this was solved by substracting 100 Years from the birthdate for those particular employees.

```select * from hr
where age < 18;

update hr
set birthdate = case 
	when age < 18 then date_sub(birthdate, interval 100 year)
    else birthdate end;
```

4 age groups were created (18-25, 26-40, 41-55, 55+).

- Exploratory Data Analysis (EDA): Exploratory data analysis is done to answer the listed questions and aims of this project.

```
create database empdis;

alter table hr
change column ï»¿id Emp_id varchar(30);

describe hr;

SELECT birthdate,
	case when birthdate like '%/%'then str_to_date(birthdate,'%m/%d/%Y')
	     when birthdate like '%-%'then str_to_date(birthdate,'%m-%d-%Y')
		 else null end as newbirth		  
FROM hr;

update hr
set birthdate = case when birthdate like '%/%'then str_to_date(birthdate,'%m/%d/%Y')
				when birthdate like '%-%'then str_to_date(birthdate,'%m-%d-%Y')
				else null end;

select hire_date,
	case when hire_date like '%/%' then str_to_date(hire_date, '%m/%d/%Y')
         when hire_date like '%-%' then str_to_date(hire_date, '%m-%d-%Y')
         else null end as newhire
from hr;

update hr
set hire_date = case when hire_date like '%/%' then str_to_date(hire_date, '%m/%d/%Y')
					 when hire_date like '%-%' then str_to_date(hire_date, '%m-%d-%Y')
					 else null end;

select termdate,
			case when termdate = '' then null
            else date(str_to_date(termdate,'%Y-%m-%d %H:%i:%s UTC'))
            end as newterm
from hr;

update hr
set termdate = case when termdate = '0000-00-00' then null
            else date(str_to_date(termdate,'%Y-%m-%d %H:%i:%s UTC'))
            end;

select * from hr;
describe hr;

alter table hr
modify column birthdate date;

alter table hr
modify column hire_date date;

alter table hr
modify column termdate date;

alter table hr
add column age int;

update hr
set age = timestampdiff(year, birthdate, curdate());

select * from hr
where age < 18;

update hr
set birthdate = case 
	when age < 18 then date_sub(birthdate, interval 100 year)
    else birthdate end;

delete from hr
where termdate > curdate();

-- What is the gender breakdown of employees in the company?

select gender, count(gender)
from hr
where termdate is null
group by gender;

-- What is the race/ethnicity breakdown of employees in the company?

select race, count(race)
from hr
where termdate is null
group by race
order by count(race) desc;

-- What is the age distribution of employees in the company?

with cte as(
select age,
		case when age >= 18 and age <= 25 then '18-25'
			 when age >= 26 and age <= 40 then '26-40'
             when age >= 41 and age <= 55 then '41-55'
             else '55+' end as age_group
from hr
where termdate is null
)
select age_group, count(age_group)
from cte
group by age_group
order by count(age_group) desc;

select min(age), max(age)
from hr
where termdate is null;


-- How many employees work at headquarters versus remote locations?

select location, count(location)
from hr
where termdate is null
group by location;

-- What is the average length of employment for employees 
-- who have been terminated?

select round(avg(year(termdate) - year(hire_date)),1) as av_length
from hr
where termdate is not null;

-- How does the gender distribution vary across departments?

select department, gender, count(gender) as count
from hr
where termdate is null
group by gender, department
order by department;


-- What is the distribution of job titles across the company?

select jobtitle, count(jobtitle)
from hr
where termdate is null
group by jobtitle
order by count(jobtitle) desc;

-- Which department has the highest turnover rate?

select department, count(department) as total_count, 
	   count(termdate) as terminated_, 
	   count(termdate)/count(department) as term_rate
from hr
group by department
order by count(termdate)/count(department) desc;

-- What is the distribution of employees across locations by state?

select location_state, count(location_state)
from hr
where termdate is null
group by location_state
order by count(location_state) desc;

-- How has the company's employee count changed over time based on hire 
-- and term dates?

select year(hire_date), count(hire_date), count(termdate),
	   count(hire_date)-count(termdate) as net_change,
       (count(hire_date)-count(termdate))/count(hire_date) * 100 as net_percent_change
from hr
group by year(hire_date)
order by year(hire_date);

-- Age-Gender Distribution

with cte as(
select  gender, age,
		case when age >= 18 and age <= 25 then '18-25'
			 when age >= 26 and age <= 40 then '26-40'
             when age >= 41 and age <= 55 then '41-55'
             else '55+' end as age_group
from hr
where termdate is null
)
select age_group, gender,count(gender)
from cte
group by age_group, gender
order by age_group, gender;

```

## Conclusion:

## Business Questions Answered

1. What is the gender breakdown of employees in the company?
- There are more male employees.

2. What is the age distribution of employees in the company?
- The youngest employee is 21 years old and the oldest is 58 years old. 4 age groups were created (18-25, 26-40, 41-55, 55+). A large number of employees were between 26-40 followed by 41-55 while the smallest group was 55+.

3. How many employees work at headquarters versus remote locations?
- A large number of employees work at the headquarters versus remotely.

4. What is the average length of employment for employees who have been terminated?
- The average length of employment for terminated employees is around 7.8 years.

4. How does the gender distribution vary across departments?
- The gender distribution across departments is fairly balanced but there are generally more male than female employees.

5. Which department has the highest turnover rate?
- The Auditing department has the highest turnover rate followed by Legal. The least turn-over rate are in the Marketing, Business Development and Sales.

6. What is the distribution of employees across locations by state?
- A large number of employees come from the state of Ohio, followed by Pennsylvania and Illinois.

7. How has the company's employee count changed over time based on hire and term dates?
- The net change in employees has increased over the years.


8. What is the tenure distribution for each department?
- The average tenure for each department is about 8 years with Legal and Auditing having the highest and Services, Sales and Marketing having the lowest.

# Limitations

- Some termination dates were far into the future and were not included in the analysis(1414 records). The only term dates used were those less than or equal to the current date.
