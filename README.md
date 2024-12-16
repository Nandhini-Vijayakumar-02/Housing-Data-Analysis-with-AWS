 Housing-Data-Analysis-with-AWS

In this project, we will delve into the realm of data engineering by building and automating an ETL process using Python. Our primary data source will be real estate property data from the Zillow Rapid API. This data will be extracted and stored in an Amazon S3 bucket, triggering a series of AWS Lambda functions. These functions will transform the data and convert it into a CSV format, which will then be loaded into another S3 bucket using AWS Lambda. We will use an S3KeySensor operator in Apache Airflow to check if the transformed data is available in the s3 bucket. Once the data is available, it will be loaded into Amazon Redshift for further analysis. Finally, we will use Tableau to visualize the data, enabling us to gain insights from the Zillow dataset

Tools & Technologies:
Rapid API: A website for extracting Zillow Housing data using API
Amazon S3: For data storage and triggering the ETL process.
AWS Lambda: For data transformation and conversion to CSV format.
Apache Airflow: For orchestrating and scheduling the ETL workflow.
Amazon Redshift: For storing the processed data in a scalable data warehouse.
Amazon Redshift Query Editor: Used as the query editor for the data in redshift 
Tableau: For creating interactive dashboards and visualizations of the data.

Pipeline:

Extract Data: We Start by extracting real estate properties data from the Zillow Rapid API. We will be using apache airflow to trigger all the tasks
Load Data to S3: Load this data into an Amazon S3 bucket, which will trigger a series of AWS Lambda functions.
Transform Data: These Lambda functions will transform the data, convert it into CSV format, and load it into another S3 bucket using Apache Airflow.
Monitor and Load: Utilize Apache Airflows S3KeySensor operator to monitor the transformed data in the S3 bucket and subsequently load it into Amazon Redshift.
Visualize Data: Connect Tableau to the Redshift cluster to visualize the Zillow data.

 <img width="464" alt="image" src="https://github.com/user-attachments/assets/040a6a2e-546e-43c1-ac80-699b466a8720">


 Airflow DAG : zillow_analytics_dag
 1. Imports and Configuration
- Libraries:
  - airflow and related modules for defining the DAG and tasks.
  - datetime and timedelta for managing task schedules and retry delays.
  - json and requests for working with API data.
  - Operators such as PythonOperator, BashOperator, S3KeySensor, and S3ToRedshiftOperator.

- API Configuration:
  - The file /home/ubuntu/airflow/config_api.json is loaded to retrieve API authentication details (api_host_key). This file contains the necessary headers for accessing the Zillow API.

 2. Helper Function: extract_zillow_data
This function performs the following steps:
1. API Request:
   - Sends a GET request to the Zillow API using the provided URL, headers, and query parameters (e.g., location="ma" for Massachusetts).

2. Save Response:
   - Saves the JSON response from the API to a file named response_data_<timestamp>.json, where <timestamp> is dynamically generated.

3. Output:
   - Returns a list containing the local path of the JSON file and a placeholder for the CSV file name. These outputs are passed to downstream tasks via XCom.

 3. Default DAG Arguments
- Shared Parameters:
  - owner: Owner of the DAG.
  - start_date: Specifies the earliest date for DAG runs (e.g., 2023-08-01).
  - retries and retry_delay: Configures automatic retries for failed tasks (up to 2 retries with a 15-second delay).
  - email_on_failure and email_on_retry: Disables email notifications for retries and failures.

 4. DAG Definition: zillow_analytics_dag
- Purpose:
  - Automates the workflow of extracting Zillow data and transferring it to Redshift.

- Schedule:
  - Runs daily (schedule_interval=@daily).
  - Catchup Disabled: Ensures that missed runs are not backfilled.

---

 Tasks in the DAG

 Task 1: extract_zillow_data_var
- Type: PythonOperator
- Purpose: Calls the extract_zillow_data function to fetch data from the Zillow API.
- Parameters:
  - url: API endpoint (https://zillow56.p.rapidapi.com/search).
  - querystring: Filters results for "ma" (Massachusetts).
  - headers: Authentication details loaded from config_api.json.
  - date_string: A timestamp for file naming.
- Output: Returns file paths (JSON and CSV) for downstream tasks.

---

 Task 2: task_load_to_s3
- Type: BashOperator
- Purpose: Moves the extracted JSON file to an S3 bucket.
- Command: Uses AWS CLI to upload the file:
  bash
  aws s3 mv {{ ti.xcom_pull("task_extract_zillow_data_var")[0] }} s3://adprojectdemo1/
  
  - The file path is retrieved from extract_zillow_data_var using XCom.
  - The destination bucket is adprojectdemo1.

---

 Task 3: task_is_file_in_s3_available
- Type: S3KeySensor
- Purpose: Validates that the file is successfully uploaded to the S3 bucket.
- Parameters:
  - bucket_name: Specifies the S3 bucket (cleaned-csvdata-bucket).
  - bucket_key: File name retrieved from extract_zillow_data_var via XCom.
  - poke_interval: Checks for the files existence every 5 seconds, up to a maximum of 360 seconds.

---

 Task 4: task_transfer_s3_to_redshift
- Type: S3ToRedshiftOperator
- Purpose: Loads the data from S3 into an Amazon Redshift table.
- Parameters:
  - s3_bucket: Source bucket (cleaned-csvdata-bucket).
  - s3_key: File name retrieved via XCom.
  - schema and table: Target Redshift schema (PUBLIC) and table (zillowdata).
  - copy_options: Specifies CSV format and skips the header row (csv IGNOREHEADER 1).

---

 Task Dependencies
The tasks are arranged sequentially using the >> operator:
1. extract_zillow_data_var: Extracts Zillow data from the API.
2. task_load_to_s3: Uploads the extracted data to the S3 bucket.
3. task_is_file_in_s3_available: Verifies the files presence in S3.
4. task_transfer_s3_to_redshift: Transfers the data from S3 to Redshift.

---

This DAG provides an automated, scalable solution for processing Zillow data and making it readily 
available in a data warehouse for further analysis.

