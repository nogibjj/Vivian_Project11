# README [![CI](https://github.com/nogibjj/Project4_Vivian/actions/workflows/ci.yml/badge.svg)](https://github.com/nogibjj/Project4_Vivian/actions/workflows/ci.yml)


### Introduction 

This project develops a Databricks ETL(Extract, Transform, Load) Pipeline for retrieving and processing airline safety datasets, featuring a well-documented notebook for ETL operations, Delta Lake for storage, and Spark SQL for transformations. It ensures data integrity through robust error handling and data validation and includes data visualizations for insights. An automated Databricks API trigger highlights the focus on automation and continuous processing.

The workflow includes running a Makefile to perform tasks such as installation (`make install`), testing (`make test`), code formatting (`make format`) with Python Black, linting (`make lint`) with Ruff, and an all-inclusive task (`make all`) using `Github Actions`. This automation streamlines the data analysis process and enhances code quality.

### Preparation

+ I used the nogibjj/python-ruff-template.

+ I chose the Airline safety dataset `airline-safety.csv` from Github.

### Dataset Description

The dataset `airline-safety.csv` originates from the Aviation Safety Network and consolidates safety-related information for 56 airlines in a CSV file. It encompasses data on available seat kilometers flown every week and provides a detailed record of incidents, fatal accidents, and fatalities, each segregated into two time frames: 1985–1999 and 2000–2014.

#### [Resources](https://github.com/fivethirtyeight/data/tree/master/airline-safety) 

### Databricks Installations and Environment Setup 

+ Create a Databricks workspace on Azure

+ Connect GitHub account to Databricks Workspace

+ Create global init script for cluster start to store environment variables

  + Establishes a connection to the Databricks environment using environment variables for authentication (SERVER_HOSTNAME and ACCESS_TOKEN).

+ Create a Databricks cluster that supports Pyspark

+ Clone Github repo into Databricks workspace

+ Create a job on Databricks to build an ETL pipeline

+ Create dependencies in `requirements.txt`


### Usage and Functionality 

`mian.py`: imports functions from three different modules: `extract` from the `mylib.extract` module, `load` from the `mylib.transform_load` module, and both `query_transform` and `viz` from the `mylib.query_viz` module. 

(1) Data Extraction `mylib/extract.py`:

  + Environment Setup: loads environment variables using `dotenv`, includes the server hostname and access token necessary for API authentication.

  + Utilizes the `requests` library to fetch airline safety data from specified URLs.

    + API Communication Functions: defines several functions to interact with the Databricks REST API:

        + `perform_query`: Makes API requests and returns the response in JSON format.

        + `mkdirs`: Creates directories in Databricks FileStore.

        + `create`: Initializes file creation in the FileStore.

        + `add_block`: Adds a block of data to a file in the FileStore.

        + `close`: Closes the file operation.

    + File Upload Process: The` put_file_from_url` function downloads a file from a given URL and uploads it to the Databricks FileStore, handling the data in blocks and encoding it in base64 format. It also ensures that files are overwritten if specified.

  + `extract`: First ensures the target directory exists, then downloads and stores the data in the Databricks FileStore.

(2) Data Transformation and Load `mylib/transform_load.py`:

  + `load`: Transform the csv file into a Spark dataframe which is then converted into two Delta Lake Tables `airline_safety1_delta` and `airline_safety2_delta` and stored in the Databricks environment

        + Creating SparkSession: create `sparksession` in the application name "Read CSV". 

        + Reading CSV Data: loads the CSV data from the specified dataset path. 

        + Splitting Columns: splits the columns of the DataFrame into two halves. 

        + Creating Two New DataFrames: Two new DataFrames are created by selecting the respective halves of the columns from the original DataFrame.

        + Adding Unique IDs: Both new DataFrames (`airline_safety_df1` and `airline_safety_df2`) are augmented with a new column named "id", populated with unique IDs generated by `monotonically_increasing_id()`.

        + Writing to Delta Lake Tables: Each DataFrame is then written to separate Delta Lake tables (a storage format for big data) named `airline_safety1_delta` and `airline_safety2_delta`. The `overwrite` mode is used, meaning any existing data in the tables will be replaced.

(3) Query Transformation and Visualization `mylib/query_viz.py`:

  + Defines a Spark SQL query to perform a predefined transformation on the retrieved data.

    + `query_transform`:  creates a Spark session with the application name "Query" and runs a SQL query on two joined Delta Lake tables (`airline_safety1_delta` and `airline_safety2_delta`). The query selects and aggregates data for airline incidents, fatal accidents, and fatalities across two time periods (1985-1999 and 2000-2014). It returns the result as a Spark DataFrame.
        
  + Uses the predifined transformation Spark dataframe to create vizualizations

      + `viz`: first calls `query_transform` to get the query result. It performs data validation by checking if the DataFrame has rows. If not, it indicates that no data is available.

      + Data Conversion and Visualization: converts the Spark DataFrame to a Pandas DataFrame for visualization. 

          + Creates a bar plot to show the total number of incidents vs.  total number of fatal accidents vs. total fatalities for the top 10 airlines (sorted by total incidents) across the two time periods (1985-1999 vs. 2000-2014).

          + Creates a line plot to visualize total fatalities change for each of the top 10 airlines (sorted by total incidents) across the two time periods (1985-1999 vs. 2000-2014).

### Error Handling and Data Validation

`test_main.py` File Path Checking for `make test`:

+ Utilizes the Databricks API and the requests library

+ Implements a function to check if a specified file path exists in the Databricks FileStore.

### Automated Trigger to Initiate Pipeline via Github Push:*

`run_job.py`: utilize the Databricks API to run a job on the Databricks workspace such that when a user pushes to this repo it will initiate a job run

### Automation and GitHub Actions

`Makefile` & `cicd.yml` : `make install`, `make test`,`make format`, `make lint`, `make all`, `make job`

### Data Source, Data Sink (Delta Lake) and ETL pipeline:

+ Extract task (Data Source): `mylib/extract.py`

+ Transform and Load Task (Data Sink): `mylib/transform_load.py`

+ Query and Viz Task: `mylib/query_viz.py`



### References 

https://github.com/nogibjj/python-ruff-template

https://hypercodelab.com/docs/spark/databricks-platform/global-env-variables

https://docs.databricks.com/en/dbfs/filestore.html

https://learn.microsoft.com/en-us/azure/databricks/delta/

https://learn.microsoft.com/en-us/training/paths/data-engineer-azure-databricks/

https://docs.databricks.com/en/getting-started/data-pipeline-get-started.html



![1 1-function-essence-of-programming](https://github.com/nogibjj/python-ruff-template/assets/58792/f7f33cd3-cff5-4014-98ea-09b6a29c7557)



