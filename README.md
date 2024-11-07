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
Workflow (SF_DEMO_CICD.yml)
The GitHub Actions workflow runs SchemaChange to apply migrations:
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
<h3>Conclusion:</h3>
This setup automates Snowflake schema changes using SchemaChange and GitHub Actions, ensuring smooth deployments with version control and secure credentials management.

