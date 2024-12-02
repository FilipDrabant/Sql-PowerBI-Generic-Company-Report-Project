# Sql-PowerBI-Generic-Company-Report-Project
A simple project transforming small dataset using SSMS and PowerBI into a report.

Created to practice working with SQL and PowerBI.
## Goals
1. Prepare provided dataset structure using SQL and SQL Server Management Studio.
2. Create a 2025 Project report visualization with powerBI. Goal is to build a dashboard that gives insights into various projects and their details like which projects are over budgetet and which are under.

## Preparing dataset
Provided dataset was in multiple files in csv format each containg 1 table.
I created a new database on my local sql server using SSMS and imported the providede datasets using Import flat files function.

![image](https://github.com/user-attachments/assets/16dcb79a-2741-4e15-8e0b-ba93a7fd823b)

### Normalizing Tables

The provided dataset's structure was not Ideal to begin working with. The three tables projects, completed_projects and upcoming projects needed to be normalized.

Firstly table'upcoming projects' had a space in It's name which causes unnecessary bloat when writing sql queries as such name with spaces requires square brackets to work with.
In ssms this is simply done by right clicking the table name in explorer and choosing rename. After renaming table I refreshed cache by going to edit->InteliSense-> Refresh Local Cache, so that feauters like auto-complete and warning work properly.

Secondly table 'projects' had multiple problems such as completly wrong data with duplicite values as in other 2 tables.
I began with renaming the table to current_projects to better convey It's purpose. Second step was to delete all data from the table itself executing following query.

```
DELETE FROM current_projects;
```
After that I started altering the current_projects table structure.

![image](https://github.com/user-attachments/assets/82d57922-2c4a-459f-8034-fa363604858a)

I removed redundant column 'column1'.

```
ALTER TABLE table_name
DROP COLUMN column_name;
```
Next I needed to rename column names 'start_date' and 'end_date' to be inline with how they are named in other tables 'project_start_date' and 'project_end_date'

```
EXEC sp_rename 'current_projects.end_date', 'project_end_date', 'COLUMN';
EXEC sp_rename 'current_projects.start_date', 'project_start_date', 'COLUMN';
```
![image](https://github.com/user-attachments/assets/a9d47b75-c40d-4217-84f6-97b9eb5a18e9)

After that all that was left was to insert new data into the table.
```
insert into current_projects(project_id,project_name,project_budget,project_start_date,project_end_date,department_id)
values(
	216,
	'Optimizing data flow and storage systems',
	30000,
	'2024-11-12',
	'2025-05-14',
	5
)

select * from current_projects
```

If I inserted wrong value by mistake I corrected them like this:
```
UPDATE current_projects
SET project_budget = 34000
WHERE project_id = 213;
```

### Writing Queries for PowerBI
I decided to create a 'Main_Table' query from 'employees' table that would bring together all availible information about each employee.
I created a union table from 'current_projects' and 'upcoming_projects' as 'project_status' as employees can be assigned to one of these.
It was also Important to select for correct year.

```
-- Project Status Table

;with project_status as(
select 
	project_id,
	project_name,
	project_budget,
	project_start_date,
	project_end_date,
	department_id,
	'current' as status
from current_projects
where project_end_date >= '2025-01-01'
union all
select 
	project_id,
	project_name,
	project_budget,
	project_start_date,
	project_end_date,
	department_id,
	'upcoming' as status
from upcoming_projects
where project_end_date <= '2026-01-01'),

-- MAIN TABLE
select
	e.department_id,
	e.employee_id,
	e.first_name,
	e.last_name,
	e.email,
	e.job_title,
	e.salary,
	e.hire_date,
	d.Department_Name,
	d.Department_Goals,
	d.Department_Budget,
	pa.project_id,
	ps.project_name,
	ps.project_budget,
	ps.status
from employees e
join departments d
	on e.department_id = d.Department_ID
join project_assignments pa
	on e.employee_id = pa.employee_id
join project_status ps
	on pa.project_id = ps.project_id
```
Last query was to create a new table from departments showcasing total costs for each department
```
select 
	d.Department_ID,
	d.Department_Name,
	d.Department_Goals,
	d.Department_Budget,
	SUM(mt.salary * d.Number_of_Employees) as salary_cost,
	SUM(mt.project_budget) AS total_project_budget,
	d.Department_Budget - (SUM(mt.salary * d.Number_of_Employees) + SUM(mt.project_budget)) AS surplus_budget
from  
	departments d
join
	main_table mt
on 
	mt.department_id = d.Department_ID
group by
	d.Department_ID,
  d.Department_Name,
  d.Department_Goals,
	d.Department_Budget;
```

## Importing Data to PowerBI and building Interactive dashboard
In Desktop PowerBI I imported both tables by selecting Get Data -> SQL server ->
![image](https://github.com/user-attachments/assets/0b8fd136-8535-4e47-959d-367617703a88)

Using this data I've built an interactive dashboard using Cards, Tables, Donut charts, Transform Data, Slicers and more.
Unfortunatly because of requiring a paid version I can only share static screenshot of dashboard here:

![image](https://github.com/user-attachments/assets/8a5c0205-7d4b-4a4c-81fb-e47b52a20726)

![image](https://github.com/user-attachments/assets/fed44121-2ee6-4568-8b85-01cd3072a996)






