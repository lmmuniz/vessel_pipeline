# Contents

**Key Takeaways:**
- All Vessels generates every 15 minutes a location position along with other information, such as: Latitude, Longitude, Speed Over Ground and Heading into a Customer managed Apache Kafka topic
- The Vessel Performance is maintained into a On-Premise PostgresSQL database and every hour it needs to send data to the Data Lake in AWS S3 (Simple Storage Service), comprised of useful information, such as: Engine Power, Current Direction, Fuel and Oil Temperature (and over other 300 metrics)
- The Vessel Information is maintained into a On-Premise PostgresSQL database and Once in a Week it needs to send data to the Data Lake in AWS S3 (Simple Storage Service), comprised of useful information, such as: Vessel Name, Vessel ID, Feet Type, Lenght, Breath
- The Weather Condition of a Vessel position (Longitude/Latitude) based on a Unix Epoch timestamp of the vessel generated position, which is going to be used the OpenWeatherMap.org platform thru a One-Call API of the Historical Weather Data


# Data Architecture

![BI_Vessel_Pipeline](https://user-images.githubusercontent.com/39410838/211897634-10b06248-17e7-464d-a0b4-1b648ce170f9.jpg)


# Pipelines Services:

**AWS S3 bucket**
- Create 3 buckets for landing files:
    - vessel-information
    - vessel-performance
    - vessel-position-weather

**AWS ECR Repository**
- Create a Repository to store the Python Docker container

**AWS ECS Task**
- Task Trigger = Every 15 min, to be created in AWS Eventbridge
- Task Definition = To be created in AWS ECS
- ECR Docker Container = A Python Docker based docker container stored in AWS ECR Repository
    **with a requirements.txt libraries list:**
    - boto3
    - awswrangler
    - json
    - requests
    - confluent-kafka
    - pandas
 - Python Steps:
    - Connect to the Apache Kafka topic based on the broker bootstrap hosts
    - Use the method Consumer to read messages ofsets
    - For each message received, send an API request to the OpenWeatherMap.org URL sending the message latitude, longitude and datetime
          - Append the returned json payload to the message payload into a Pandas Dataframe
          - Append each message to a final Pandas Dataframe
    - Write the final Pandas dataframe to a AWS S3 bucket (e.g.: vessel-position-weather) in parquet format, partitioned by Vessel ID and position timestamp

**AWS DMS Task**

The Database Migration Service is a service which helps to bring On-Premise database tables to the AWS
For this purpose, the DMS we would need to setup:
- A Replication Instance
- A source Endpoint:
    - 1. Vessel Information source database (host/port/user/password)
    - 2. Vessel Performance source database (host/port/user/password)
- A target Endpoint:
    - 1. Vessel Information target S3 bucket (e.g: vessel-information)
    - 2. Vessel Performance target S3 bucket (e.g: vessel-performance)
- A migration task:
    - 1. Vessel Information from Source to Target Endpoint and mode (Full Load + CDC)
    - 2. Vessel Performance  from Source to Target Endpoint and mode (Full Load + CDC)

**AWS Glue Data Catalog**

The AWS Glue data catalog is a service which runs a crawler in the S3 bucket and map new/changes on partitions and reference the other services such as AWS Athena the right table schema
- Create 3 Glue crawlers
    - vessel-information (daily)
    - vessel-performance (hourly)
    - vessel-position-weather (every 15 minutes)

**AWS Athena**
- Creates a Database and Tables based on the Glue Data Catalog using a Presto like SQL layer of abstraction.

**AWS Quicksight**
- Creates Dashboards/Reports on top of Athena Datasets imported to it

**AWS Sagemaker**
- Creates a Connection of ML Notebooks (Jupyter) on top of AWS Athena database, to inference data from the tables
