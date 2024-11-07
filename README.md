<h1>Snowflake CI/CD Demo with SchemaChange and GitHub Actions</h1>

This project demonstrates how to use Snowflake, SchemaChange, and GitHub Actions to implement a simple CI/CD pipeline for automating Snowflake schema changes. The setup involves creating migration scripts, pushing them to GitHub, and using GitHub Actions to deploy those changes to Snowflake.

Project Overview
SchemaChange is used to version control and apply schema changes to Snowflake. It can automatically create a history table to track changes and deploy migration scripts.
GitHub Actions is used to automate the deployment pipeline by triggering workflows whenever changes are pushed to the repository.
This project uses a folder structure where SQL migration scripts are stored in a migrations/ directory.
Requirements
Before setting up the CI/CD pipeline, make sure you have the following:

Snowflake Account: You should have access to a Snowflake account with necessary privileges to create schemas, tables, and perform schema migrations.
GitHub Account: A GitHub repository where you can store and version control your Snowflake schema changes.
SchemaChange: This is a tool that helps you manage and apply version-controlled schema changes to Snowflake.
Setup Instructions
Step 1: Create Snowflake Secrets in GitHub
In order to securely pass Snowflake credentials to the GitHub Actions workflow, you need to create secrets in your GitHub repository.

Go to your GitHub repository.

Navigate to Settings > Secrets > New repository secret.

Add the following secrets (replace the values with your Snowflake credentials):

SF_ACCOUNT: Your Snowflake account URL (e.g., xy12345.snowflakecomputing.com).
SF_USERNAME: Your Snowflake username.
SF_PASSWORD: Your Snowflake password.
SF_ROLE: The role to be used for the deployment.
SF_WAREHOUSE: The Snowflake warehouse to run the deployment.
SF_DATABASE: The Snowflake database to deploy changes to.
Step 2: Folder Structure
The project folder structure should look like this:

graphql
Copy code
.
├── migrations/
│   ├── V1.1.1__initial_objects.sql  # Initial schema creation script
│   ├── V1.1.2__updated_objects.sql  # Updated schema with new column
│
├── .github/
│   └── workflows/
│       └── SF_DEMO_CICD.yml  # GitHub Actions Workflow file
│
├── README.md  # Project description
├── requirements.txt  # Python dependencies for CI/CD
Step 3: Create Migration Scripts
Create SQL migration files in the migrations/ folder. Each file should follow the naming convention V<version>__<description>.sql:

V1.1.1__initial_objects.sql:

sql
Copy code
CREATE SCHEMA DEMO;
CREATE TABLE HELLO_WORLD
(
   FIRST_NAME VARCHAR,
   LAST_NAME VARCHAR
);
V1.1.2__updated_objects.sql (this script adds a new column to the existing table):

sql
Copy code
USE SCHEMA DEMO;
ALTER TABLE HELLO_WORLD ADD COLUMN AGE NUMBER;
Step 4: GitHub Actions Workflow
The pipeline is triggered by pushing changes to the main branch, specifically when changes occur in the migrations/ folder. Below is the content of the GitHub Actions workflow file, .github/workflows/SF_DEMO_CICD.yml.

yaml
Copy code
name: snowflake-devops-demo

on:
  push:
    branches:
      - main
    paths:
      - 'migrations/**'

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  deploy-snowflake-changes-job:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Use Python 3.8.x
        uses: actions/setup-python@v2.2.1
        with:
          python-version: 3.8.x

      - name: Run schemachange
        env:
          SF_ACCOUNT: ${{ secrets.SF_ACCOUNT }}
          SF_USERNAME: ${{ secrets.SF_USERNAME }}
          SF_ROLE: ${{ secrets.SF_ROLE }}
          SF_WAREHOUSE: ${{ secrets.SF_WAREHOUSE }}
          SF_DATABASE: ${{ secrets.SF_DATABASE }}
          SNOWFLAKE_PASSWORD: ${{ secrets.SF_PASSWORD }}
        run: |
          echo "GITHUB_WORKSPACE: $GITHUB_WORKSPACE"
          python --version
          echo "Step 1: Installing schemachange"
          pip install schemachange
          
          echo "Step 2: Running schemachange"
          schemachange -f $GITHUB_WORKSPACE/migrations -a $SF_ACCOUNT -u $SF_USERNAME -r $SF_ROLE -w $SF_WAREHOUSE -d $SF_DATABASE -c $SF_DATABASE.SCHEMACHANGE.CHANGE_HISTORY --create-change-history-table
Workflow Explanation:
Trigger: The workflow triggers on a push to the main branch or a manual trigger (workflow_dispatch).
Checkout: The repository is checked out so the workflow can access the migration files.
Python Setup: The workflow sets up Python 3.8.x for installing and running SchemaChange.
SchemaChange Execution:
Installs SchemaChange using pip.
Executes SchemaChange to apply the migration scripts located in the migrations/ folder to Snowflake.
Step 5: Commit and Push
Once you have your migration scripts and the GitHub Actions workflow in place, commit and push the changes to GitHub:

bash
Copy code
git add .
git commit -m "Add migration scripts and CI/CD workflow"
git push origin main
This will automatically trigger the GitHub Actions workflow, which will run SchemaChange and apply the changes to Snowflake.

Step 6: Monitor and Review
Once the push is made, the workflow will be triggered. You can monitor the progress and status of the workflow in the Actions tab of your GitHub repository.
If successful, the migration scripts will be applied to Snowflake.
If there are any issues, you can view the logs in the GitHub Actions UI for debugging.
Conclusion
By following the steps outlined in this README, you can successfully set up a simple CI/CD pipeline using Snowflake, SchemaChange, and GitHub Actions to automate your database schema changes. This setup allows you to version control your schema changes, apply them in a consistent manner, and manage your Snowflake environment using modern DevOps practices.
