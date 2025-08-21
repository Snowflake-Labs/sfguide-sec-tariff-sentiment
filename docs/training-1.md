# AWS S3 ingestion guided steps

## Overview

In this guide we will be:

1. Adding an AWS S3 ingestion to this solution template


## Stage 0 - Design

### Deploy vs Demo

!!! success "Important"

    Before you begin making changes to your template, you will need to decide which
    parts of your solution are to be deployed and built (automated) before you
    present your demo to the customers. 
    
    Everything you want to demo to customers will become part of this homepage to 
    help others with the talk track needed for a successful demo.


We recommend splitting your design into 2 parts, the "deploy" i.e. things that need
to be built before the demo, and the "demo" itself which can be combinations of
using Snowflake features in Snowsight UI or the DataOps.live DDE. The narrative of
your solution would be captured in this solution homepage.

### Snowflake objects

All our demo objects required for this training can be deployed ahead of the demo
delivery so we will add their management to our SOLE and MATE configurations.

### Talk track

We will need to explain the addition of automatic ingestion to our customer, this
means we will update the solution homepage with this information.

!!! tip

    As a simple rule of thumb, if it's something you want shown in front of a 
    customer, it goes in the Solution Homepage. If it's something that your 
    narrative depends on, but isn't part of the narrative itself then it goes 
    in the build pipeline.

## Stage 1 - Updating SOLE

!!! note

    Some configuration is omitted in the examples in these steps. See the 
    fully completed file at the end of each stage.

!!! tip "Tip about indentation"

    When editing in the DDE, you may want to detect the 2 space indentation so 
    you can use the TAB key for easier indentation management. 
    
    Use the command pallet with `command + shift + p` and search for "Detect 
    indentation", hit enter and it should detect the 2 space indentation. 
    
    Check your current indentation from the status bar in the bottom right.

!!! abstract

    This stage is part of the **Deploy** design.

### Step 1: Add schema

In `dataops/snowflake/databases.template.yml`:

1. Add a new `STAGING` schema with placeholder object sections:
    ```yaml
    {% raw %}
    databases:
      "{{ env.DATAOPS_DATABASE }}":
        <!-- other content omitted -->
        schemas:
          STAGING:
            grants: *default_schema_grants
            stages:
            file_formats:
            tables:
    {% endraw %}
    ```

### Step 2: Add stage

1. Add a new `S3_STAGE` in the stages section:
    ```yaml hl_lines="8-18"
    {% raw %}
    databases:
      "{{ env.DATAOPS_DATABASE }}":
        <!-- other content omitted -->
        schemas:
          STAGING:
            grants: *default_schema_grants
            stages:
              S3_STAGE:
                comment: Created to load the Adventureworks data
                url: s3://dataopsliveadvancedworkshop/
                file_format: ADVW_UTF8_TAB              
                grants:
                  ALL PRIVILEGES:
                    - ADMIN
                    - DATA_ENGINEER
                    - DATA_SCIENTIST
                    - BI
                    - DATA_APP
            file_formats:
            tables:
    {% endraw %}
    ```

### Step 3: Add file format

1. Add a new `ADVW_UTF8_TAB` file format in the file formats section:
    
    /// html | div[class="collapse-code"]

    ```yaml hl_lines="11-42"
    {% raw %}
    databases:
      "{{ env.DATAOPS_DATABASE }}":
        <!-- other content omitted -->
        schemas:
          STAGING:
            grants: *default_schema_grants
            stages:
              S3_STAGE:
                ...
            file_formats:
              ADVW_UTF8_TAB:
                comment: "File format for type csv"
                format_type: CSV
                compression: AUTO
                record_delimiter: "\n"
                field_delimiter: "\t"
                binary_format: HEX
                encoding: "UTF8"
                null_if: 
                - "\\\\N"              
                escape_unenclosed_field: "\\\\"
                skip_header: 0
                escape: none
                field_optionally_enclosed_by: "NONE"
                date_format: "AUTO"
                time_format: "AUTO"
                timestamp_format: "AUTO"
                validate_utf8: true
                error_on_column_count_mismatch: true
                empty_field_as_null: true
                skip_byte_order_mark: true
                lifecycle:
                    ignore_changes:
                      - null_if
                      - escape_unenclosed_field              
                grants:
                  USAGE:
                    - ADMIN
                    - DATA_ENGINEER
                    - DATA_SCIENTIST
                    - BI
                    - DATA_APP
            tables:
    {% endraw %}
    ```
    ///

### Step 4: Add tables

1. Remove the unused `SNOWFLAKE` database.
1. Add a new tables in the tables section:

/// html | div[class="collapse-code"]
```yaml
{{ include_raw("docs/training/stage1-step5-tables.yml") }}

```
///
### Step 5: Run SOLE Validate

#### Option 1: CLI
Using the terminal, you can run the command to validate the syntax of the files found in the dataops/snowflake directory

!!! hint

    You do NOT need to be in the dataops/snowflake directory to run this command.

1. In a terminal run the command:
    ```bash
    sole-validate
    ```

1. The output will be shown in the terminal and you will see "SOLE configuration: LOOKS GOOD" if your syntax is correct, or an error message with the location of the error.

#### Option 2: DDE

Using the DDE UI, you can run SOLE Validate to ensure the syntax in your dataops/snowflake files are correct.

1. In the git menu, in the DataOps.live extension, click "SOLE validate" under the SOLE section.

    ![Alt text](img/t1-stage1-sole-validate.png)

1. The output will be shown in the terminal and you will see "SOLE configuration: LOOKS GOOD" if your syntax is correct, or an error message with the location of the error.


### Summary

Completed file with the changes:

/// html | div[class="collapse-code"]
```yaml title="dataops/snowflake/databases.template.yml"
{{ include_raw("docs/training/stage1-databases-template.yml") }}
```
///

## Stage 2 - Create ingestion job

!!! abstract

    This stage is part of the **Deploy** design.

In this stage we will be creating a new job for your pipeline.

![Example ingestion job pill](img/t1-stage2-pipeline-job.png)

### Step 1: Add job file

1. Create a new directory `dataops/pipelines/includes/local_includes/snowflake_direct_ingestion/`
    ```bash
    mkdir -p dataops/pipelines/includes/local_includes/snowflake_direct_ingestion/
    ```
1. In the new directory, create a new file called `direct_ingestion_orchestrator.yml`
    ```bash
    touch dataops/pipelines/includes/local_includes/snowflake_direct_ingestion/direct_ingestion_orchestrator.yml
    ```

1. Add the job definition:

    ```yaml linenums="1" title="dataops/pipelines/includes/local_includes/snowflake_direct_ingestion/direct_ingestion_orchestrator.yml"
    Direct Ingestion:
      extends: .agent_tag 
      stage: Data Ingestion 
      image: $DATAOPS_STAGEINGESTION_RUNNER_IMAGE 
      variables: 
        DATAOPS_SOLE_DEBUG: 1                        # Only for debugging – shows the queries 
        DATAOPS_STAGE_INGESTION_ACTION: START 
        CONFIGURATION_DIR: $CI_PROJECT_DIR/dataops/snowflake 
            # Adjust the values below if required, to ingest as the appropriate user/role 
        DATAOPS_SOLE_ACCOUNT: DATAOPS_VAULT(SNOWFLAKE.ACCOUNT) 
        DATAOPS_SOLE_USERNAME: DATAOPS_VAULT(SNOWFLAKE.TRANSFORM.USERNAME) 
        DATAOPS_SOLE_PASSWORD: DATAOPS_VAULT(SNOWFLAKE.TRANSFORM.PASSWORD) 
        DATAOPS_SOLE_ROLE: DATAOPS_VAULT(SNOWFLAKE.TRANSFORM.ROLE) 
        DATAOPS_SOLE_WAREHOUSE: DATAOPS_VAULT(SNOWFLAKE.TRANSFORM.WAREHOUSE) 
      resource_group: $CI_JOB_NAME 
      script: 
        - /dataops 
      icon: ${SNOWFLAKEOBJECTLIFECYCLE_ICON}
    ```

!!! hint

    Make a note of the “stage” listed and either see that it matches up to a stage already
    listed in our `dataops/pipelines/includes/config/stages.yml` file, or add it to that file.

### Step 2: Add to pipeline

1.  In your `full-ci.yml` file, add the direct ingestion orchestrator job to the direct ingestion job:
    ```yaml linenums="1" title="full-ci.yml" hl_lines="14 15"
    include:
      - /dataops/pipelines/includes/bootstrap.yml

      ## Load Secrets job
      - project: reference-template-projects/dataops-template/dataops-reference
        ref: 5-stable
        file: /pipelines/includes/default/load_secrets.yml

      ## Snowflake Object Lifecycle jobs
      - project: reference-template-projects/dataops-template/dataops-reference
        ref: 5-stable
        file: /pipelines/includes/default/snowflake_lifecycle.yml

      ## Data ingestion
      - /dataops/pipelines/includes/local_includes/snowflake_direct_ingestion/direct_ingestion_orchestrator.yml

      ## Modelling and transformation jobs
      - /dataops/pipelines/includes/local_includes/modelling_and_transformation/build_all_models.yml

      ## Testing
      - /dataops/pipelines/includes/local_includes/modelling_and_transformation/test_all_models.yml

      ## Generate modelling and transformation documentation
      - project: "reference-template-projects/dataops-template/dataops-reference"
        ref: 5-stable
        file: "/pipelines/includes/default/generate_modelling_and_transformation_documentation.yml"

      ## Build Solution Homepage
      - /dataops/pipelines/includes/local_includes/homepage/build_solution_homepage.yml
      - /dataops/pipelines/includes/local_includes/metrics/record_pipeline_start.yml
    ```

## Stage 3 - Committing changes

!!! abstract

    This stage is part of the software development lifecycle. You will want to commit and push changes regularly.

Commit your changes and run the pipeline to see the new objects created, and the data loaded.

### Option 1: CLI

Using the terminal, you can stage the "git" changes, commit them with a message and then push to the repository.

1. In a terminal you can see the current git status:
    ```bash
    git status
    ```
    Use this to see the file changes as differential.

1. In a terminal:
    ```bash
    git add --all
    git commit -m "Add new data ingestion SOLE and job"
    git push
    ```

### Option 2: DDE

Using the DDE UI, you can stage the "git" changes, commit them with a message and then push to the repository.

1. In the git menu, in the "Changes" section, click the `+` plus button, this stages the changes.

    ![Alt text](img/t1-stage3-dde-stage-all.png)

1. Add a message `Add new data ingestion SOLE and job`.

    ![Alt text](img/t1-stage3-commit-msg.png)

1. Click the `Commit` button

1. Click the `Sync changes` button

    ![Alt text](img/t1-stage3-sync-changes.png)

## Stage 4 - Testing Source Data

!!! abstract

    This stage is part of the **Deploy** design.

We are now going to use the data we just ingested as sources for MATE models. We will also be able to define tests on these columns that will be run each time the pipeline runs to ensure data quality is met.

For the sake of this exercise, we will only be using the DEPARTMENT table as a source and in our models. You can further extend the exercise by creating similar sources files and models with the other tables in our STAGING schema. 

### Step 1: Create Source File

1. In `dataops/modelling/sources` create a new folder `humanresources`.

2. Within that folder, create a file called `department.yml`

### Step 2: Define `DEPARTMENT` columns and tests.

1. In the file you just created, define the columns as well as the tests by putting in the below code:

    ```yaml 
    version: 2
    sources:
    - name: humanresources
      description: 'Human Resources Source Schema'
      schema: STAGING
      tables:
      - name: DEPARTMENT
        description: 'Department table'
        tests: 
          - dataops.minimum_rows: 
              rows: 5
        columns:
        - name: DEPARTMENTID
          description: 'Department Primary Key'
          tests:
            - not_null
            - dbt_utils.not_constant
        - name: NAME
          description: 'The name of the department'
          tests:
            - not_null
            - dbt_utils.not_constant
        - name: GROUPNAME
          description: 'The name of the group'
          tests:
            - not_null
            - dbt_utils.not_constant
        - name: MODIFIEDDATE
          description: 'Timestamp when this record was modified'
          tests:
            - not_null
    ```

### Step 3: Add Source Testing Job to Pipeline

1. If you navigate to `dataops/pipelines/includes/local_includes/modelling_and_transformation`, you can see there is a job file `test_all_sources.yml`. We need to copy the path of this job and add it to the `full-ci.yml` pipeline file for it to run with the pipeline. 

    ```yaml linenums="1" title="full-ci.yml" hl_lines="7-8"
    include:
     <!-- other content omitted --!>

      ## Data ingestion
      - /dataops/pipelines/includes/local_includes/snowflake_direct_ingestion/direct_ingestion_orchestrator.yml

      ## Source Testing
      - /dataops/pipelines/includes/local_includes/modelling_and_transformation/test_all_sources.yml

      ## Modelling and transformation jobs
      - /dataops/pipelines/includes/local_includes/modelling_and_transformation/build_all_models.yml

    <!-- other content omitted --!>
    ```

### Step 4: Commit Changes

1. Following the directions in [Stage 3 - Committing changes](#stage-3-committing-changes) to commit these changes to your project.

## Stage 5 - Using MATE

!!! abstract

    This stage is part of the **Deploy** design.

Now that we have ingested data and tested it to ensure the quality, we can add use the MATE (Modelling and Transformation) Orchestrator to transform the data. 

### Step 1: Add a new directory and model

1. In the `dataops/modelling/models` directory, add a new subdirectory `humanresources`

1. Create a file in that subdirectory called `humanresources_department.sql`

1. The body of that file should read:

    ```sql
    {% raw %}
    {{ config(alias='DEPARTMENT') }}
    SELECT
      DEPARTMENTID
      , NAME as DEPARTMENTNAME
      , GROUPNAME
      , MODIFIEDDATE
    FROM {{ source('humanresources', 'DEPARTMENT')}}
    {% endraw %}
    ```

### Step 2: Testing your new model

1. In the `dataops/modelling/models/humanresources` directory, create a file called `humanresources_department.yml`

1. The body of that file should read:

    ```yaml
    version: 2
    models:
      - name: humanresources_department
        description: 'Department table'
        columns:
        - name: DEPARTMENTID
          description: 'Department Primary Key'
          tests:
            - not_null
            - dbt_utils.not_constant
        - name: NAME
          description: 'The name of the department'
          tests:
            - not_null
            - dbt_utils.not_constant
        - name: GROUPNAME
          description: 'The name of the group'
          tests:
            - not_null
            - dbt_utils.not_constant
            - column_width
        - name: MODIFIEDDATE
          description: 'Timestamp when this record was modified'
          tests:
            - not_null
            - column_width
            - dbt_utils.expression_is_true:
                  expression: "= to_timestamp_tz(MODIFIEDDATE)"
    ```

### Step 3: Creating Custom Test

Notice in the previous step, we included a test called "column_width". This test does not exist in the included tests, but can be created using a macro. 

1. Create a file called `column_width.sql` in the `dataops/modelling/macros` directory.

1.  In the body of the file, create jinja to identify that it is a test, as well as the arguments it will accept. Following that, use a series of SELECT statements to define the qualities you wish to exclude. In our example, it is selecting columns with values less then 10 characters. 

    ``` sql
    {% raw %}
    {% test column_width(model, column_name) %}

    with validation as (
      select
        {{ column_name }} as column_width_to_test
      from {{ model }}
    ),

    validation_errors as (
      select
        column_width_to_test
      from validation
      where len(column_width_to_test)<10
    )
    select * 
    from validation_errors

    {% endtest %}
    {% endraw %}
    ```

Now that this test has been defined, it will be used when running the pipeline.

### Step 4: Modify dbt_project.yml

1. In `dataops/modelling/dbt_project.yml`, add the following to the very bottom of the file:
    ```yaml linenums="1" title="full-ci.yml" 
    humanresources:
      +schema: HUMANRESOURCES
      +meta:
        grants: *schema_grants
    ```
### Step 5: Commit Changes

1. Following the directions in [Stage 3 - Committing changes](#stage-3-committing-changes) to commit these changes to your project. 

### Summary

We have now completed the creation of our solution by adding objects with SOLE, testing and transforming them with MATE. Now we will modify the Solution Homepage so others will be able to take our solution and use it in presentations.

## Stage 6 - Solution Homepage changes

!!! abstract

    This stage is part of the **Demo** design.

In this stage we will update the solution homepage to include information on the new data ingestion.

The solution homepage is built using a tool called `mkdocs`. This tool allows you to create content
in the common "markdown" format and then it be compiled into HTML for websites.

!!! info

    You will need to be editing the files in the DataOps.live DDE.

### Step 1: mkdocs serve

To begin editing the solution homepage, you will benefit from being able to preview
the changes you are making. To achieve this, we will use the `mkdocs serve` command.

1. In a terminal, change directory to the `solution/homepage` directory

    ```bash
    cd solution/homepage
    ```

1. Start the development server, that will provide a development URL for viewing the 
    live edits to the markdown pages, with the `mkdocs serve` command.

    ```bash
    mkdocs serve
    ```
    You may notice:
    
    1. the terminal showing the log output of the web server as it builds
    and serves your compiled markdown as HTML web pages.
    1. a new Simple Browser preview opens

### Step 2: Add new page

1. Create a new file in `solution/homepage/docs` called `example-page-1.md`

1. In the `solution/homepage/mkdocs.yml` file, add the file name as below:

    ```yaml linenums="1" title="mkdocs.yml" hl_lines="7"
    site_name: Solution Template Template
    nav:
      - index.md
      - mkdocs-reference.md
      - example.md
      - training-1.md
      - example-page-1.md

    site_url: ""
    use_directory_urls: false
    <!-- other content omitted --!>
    ```

### Step 3: Add text to the file
    
    # Vignette 2 - Ingestion Workshop
    
    ### Ingestion from S3 Bucket
      - **Stage Ingestion Orchestrator:** Use this orchestrator to ingest staged data from an S3 bucket or Azure Blob Storage. 
    
Feel free to add other text and formatting. You should see it updating in the preview browser.

### Step 4: Commit changes

1. Following the directions in [Stage 3 - Committing changes](#stage-3-committing-changes) to commit these changes to your project. 

## Stage 7 - Merge changes to template