# Housing-Data-Analysis-with-AWS

In this project, we will delve into the realm of data engineering by building and automating an ETL process using Python. Our primary data source will be real estate property data from the Zillow Rapid API. This data will be extracted and stored in an Amazon S3 bucket, triggering a series of AWS Lambda functions. These functions will transform the data and convert it into a CSV format, which will then be loaded into another S3 bucket using Apache Airflow. We will use an S3KeySensor operator in Apache Airflow to check if the transformed data is available in the s3 bucket. Once the data is available, it will be loaded into Amazon Redshift for further analysis. Finally, we will use Tableau to visualize the data, enabling us to gain insights from the Zillow dataset

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
Monitor and Load: Utilize Apache Airflow's S3KeySensor operator to monitor the transformed data in the S3 bucket and subsequently load it into Amazon Redshift.
Visualize Data: Connect Tableau to the Redshift cluster to visualize the Zillow data.

 <img width="464" alt="image" src="https://github.com/user-attachments/assets/040a6a2e-546e-43c1-ac80-699b466a8720">
