# Contents

**Key Takeaways:**
- All Vessels generates every 15 minutes a location position along with other information, such as: Latitude, Longitude, Speed Over Ground and Heading into a Customer managed Apache Kafka topic
- The Vessel Performance is maintained into a On-Premise PostgresSQL database and every hour it needs to send data to the Data Lake in AWS S3 (Simple Storage Service), comprised of useful information, such as: Engine Power, Current Direction, Fuel and Oil Temperature (and over other 300 metrics)
- The Vessel Information is maintained into a On-Premise PostgresSQL database and Once in a Week it needs to send data to the Data Lake in AWS S3 (Simple Storage Service), comprised of useful information, such as: Vessel Name, Vessel ID, Feet Type, Lenght, Breath
- The Weather Condition of a Vessel position (Longitude/Latitude) based on a Unix Epoch timestamp of the vessel generated position, which is going to be used the OpenWeatherMap.org platform thru a One-Call API of the Historical Weather Data


# Data Architecture

![BI_Vessel_Pipeline](https://user-images.githubusercontent.com/39410838/211897634-10b06248-17e7-464d-a0b4-1b648ce170f9.jpg)


# Pipelines

1. ECS Task
  1.1 Task Trigger = Every 15 min via AWS Eventbridge event
  1.2 Task Definition = To be setup in AWS ECS service
  1.3 Container = A Python Docker based container with a requirements.txt libraries list:
      - boto3
      - awswrangler
      - json
      - requests
      - confluent-kafka
      - pandas
      
