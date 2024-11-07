<h2>Snowflake CI/CD with SchemaChange and GitHub Actions</h2>
This project demonstrates how to automate Snowflake schema changes using SchemaChange and GitHub Actions.

<h3>Overview</h3>
SchemaChange is used to manage Snowflake schema changes via version-controlled SQL migration files.
GitHub Actions is used to automatically deploy those changes to Snowflake when updates are pushed to the repository.
Features
Version-controlled SQL migration scripts in the migrations/ folder.
GitHub Actions automatically deploys changes to Snowflake.
Secure Snowflake credentials stored as GitHub secrets.

<h3>Project Structure</h3>

```bash
├── migrations/
│   ├── V1.1.1__initial_objects.sql  # Initial schema creation
│   ├── V1.1.2__updated_objects.sql  # Add column to table
├── .github/
│   └── workflows/
│       └── SF_DEMO_CICD.yml  # GitHub Actions workflow
├── README.md
```
<h3>How It Works</h3>
Migration Scripts: SQL scripts for schema changes are stored in the migrations/ folder (e.g., creating tables, altering columns).

CI/CD Workflow: The workflow is triggered on push events to the main branch, running SchemaChange to apply the migration scripts to Snowflake.
GitHub Secrets:
Add the following Snowflake credentials as secrets in your GitHub repository:
```
SF_ACCOUNT: Snowflake account URL
SF_USERNAME: Snowflake username
SF_PASSWORD: Snowflake password
SF_ROLE: Snowflake role
SF_WAREHOUSE: Snowflake warehouse
SF_DATABASE: Snowflake database
```
yaml file:
```
name: snowflake-devops-demo

on:
  push:
    branches:
      - main
    paths:
      - 'migrations/**'

jobs:
  deploy-snowflake-changes-job:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2.2.1
        with:
          python-version: 3.8.x

      - name: Run SchemaChange
        env:
          SF_ACCOUNT: ${{ secrets.SF_ACCOUNT }}
          SF_USERNAME: ${{ secrets.SF_USERNAME }}
          SF_ROLE: ${{ secrets.SF_ROLE }}
          SF_WAREHOUSE: ${{ secrets.SF_WAREHOUSE }}
          SF_DATABASE: ${{ secrets.SF_DATABASE }}
          SNOWFLAKE_PASSWORD: ${{ secrets.SF_PASSWORD }}
        run: |
          pip install schemachange
          schemachange -f $GITHUB_WORKSPACE/migrations -a $SF_ACCOUNT -u $SF_USERNAME -r $SF_ROLE -w $SF_WAREHOUSE -d $SF_DATABASE -c $SF_DATABASE.SCHEMACHANGE.CHANGE_HISTORY --create-change-history-table
```

<h3>Results:</h3>

```
-- Once the files are uploaded the changed timestamp and more details are updated in this table.

use database sf_cicd;

use schema schemachange;

select * from change_history;

VERSION	DESCRIPTION	SCRIPT	SCRIPT_TYPE	CHECKSUM	EXECUTION_TIME	STATUS	INSTALLED_BY	INSTALLED_ON
1.1.1	Initial objects	V1.1.1__initial_objects.sql	V	11a7cce779a911b7d6a9928d9c330ff7c8fbf6d5b627544adc9d95fe	4	Success	SHRE1234	2024-11-04 22:54:53.790 -0800
1.1.2	Updated objects	V1.1.2__updated_objects.sql	V	3a94dad244c63909acc40e43d1f2909e83694ff36ca836c8c8d628a7	4	Success	SHRE1234	2024-11-05 01:46:14.162 -0800
1.1.3	Updated objects	V1.1.3__updated_objects.sql	V	a0fb47bb7c6a35ba56f609287025cba845b67386807782cfb8f2a5e0	4	Success	SHRE1234	2024-11-06 20:18:02.196 -0800

-- Main Table

use schema demo;

select get_ddl('table','Hello_world');

create or replace TABLE HELLO_WORLD (
	FIRST_NAME VARCHAR(16777216),
	LAST_NAME VARCHAR(16777216),
	AGE NUMBER(38,0),
	ADDRESS VARCHAR(500)
);
```
<h3>Github Actions:</h3>

Executed the alter statement in Github and commited the changes

![image](https://github.com/shreyasmc1234/Snowflake_CI-CD_Test/blob/main/Images/Screenshot%202024-11-07%20104609.png)

Workflow got triggered 

![image](https://github.com/shreyasmc1234/Snowflake_CI-CD_Test/blob/main/Images/Screenshot%202024-11-07%20104632.png)

Job completed and made the changes to the main table

![image](https://github.com/shreyasmc1234/Snowflake_CI-CD_Test/blob/main/Images/Screenshot%202024-11-07%20104700.png)




<h3>Conclusion:</h3>
This setup automates Snowflake schema changes using SchemaChange and GitHub Actions, ensuring smooth deployments with version control and secure credentials management.

