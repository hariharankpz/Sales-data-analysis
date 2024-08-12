# Real-Time Sales Data Analysis

## Project Overview

This project aims to develop a real-time sales data analysis system to monitor and analyze sales transactions as they occur. The system enables immediate insights into sales performance, customer behavior, and inventory status. It provides real-time dashboards and supports ad-hoc queries to facilitate quick decision-making and operational efficiency.

## Architecture Diagram

![image](https://github.com/user-attachments/assets/c432c7a6-dd35-4dd9-bc3c-ce776c573d84)



## Tech Stack

- **AWS Lambda**: Used for generating mock order data and transforming data in real-time.
- **Amazon DynamoDB**: Serves as the primary data store for order data and captures Change Data Capture (CDC) events using DynamoDB Streams.
- **Amazon Kinesis Stream**: Collects and processes streaming data in real-time.
- **Amazon Kinesis Firehose**: Transforms and stores the streaming data in an S3 bucket.
- **Amazon S3**: Stores transformed data.
- **AWS Glue Crawler**: Scans the S3 bucket to determine the structure and schema of the stored data and updates the Glue Catalog.
- **AWS Glue Catalog**: Acts as a metadata repository that stores information about the structure and schema of the data in S3.
- **Amazon Athena**: Used for running ad-hoc queries on the data stored in the S3 bucket.

## Objective

Develop a real-time sales data analysis system to:
- Monitor and analyze sales transactions as they occur.
- Provide immediate insights into sales performance, customer behavior, and inventory status.
- Support ad-hoc queries for quick decision-making and operational efficiency.

## Components

### 1. Stream Mock Data Generator (Lambda)
- Generates order data and stores it in a DynamoDB table.

### 2. DynamoDB
- Acts as the primary data store for order data.
- DynamoDB Streams capture Change Data Capture (CDC) events.

### 3. EventBridge Pipe
- Channels the data from DynamoDB Streams into an Amazon Kinesis Stream for further processing.

### 4. Kinesis Stream
- Collects and processes the streaming data in real-time.
- Data is consumed in batches based on either the buffer size or buffer interval.

### 5. Kinesis Firehose
- Transforms the streaming data and returns it to an S3 bucket for storage.
- Invokes a Lambda function to perform transformations on the data before storage.

### 6. S3 Bucket
- Stores the transformed data.

### 7. Glue Crawler
- Scans the S3 bucket to determine the structure and schema of the stored data.
- Updates the Glue Catalog with the metadata.

### 8. Glue Catalog
- Acts as a metadata repository storing information about the structure and schema of the data in S3.

### 9. Athena
- Provides support for running ad-hoc queries on the data stored in the S3 bucket.
- Uses the Glue Catalog to understand the schema of the data.

## Why Not Use AWS Glue for Transformation?
- Glue is resource-intensive and better suited for processing large datasets.
- For small batch processing (e.g., 2-4 records), Lambda is sufficient as it can process and return the data within microseconds.
- If large-scale processing is needed, an alternative could be using Apache Spark on EMR or Apache Flink for stream processing.

## End User Requirements
- The end user needs to run ad-hoc queries on the data, which is why AWS Athena is used in this setup.

## DynamoDB CDC
- DynamoDB is a NoSQL database that allows capturing CDC (Change Data Capture) events via DynamoDB Streams.
- Other alternatives like Debezium can be used for CDC with databases like MySQL, PostgreSQL, MongoDB, etc.

---

# Real-Time Sales Data Analysis Project Implementation

This project demonstrates the implementation of a real-time data pipeline using AWS services. The pipeline captures changes in a DynamoDB table and streams them into a Kinesis Data Stream for further processing.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Mock Data Generator for Input](#step-1-mock-data-generator-for-input)
- [Step 2: DynamoDB Table Creation](#step-2-dynamodb-table-creation)
- [Step 3: Enable DynamoDB Streams](#step-3-enable-dynamodb-streams)
- [Step 4: Create a Kinesis Data Stream](#step-4-create-a-kinesis-data-stream)
- [Step 5: Create an EventBridge Pipe](#step-5-create-an-eventbridge-pipe)
- [Step 6: IAM Role Configuration](#step-6-iam-role-configuration)
- [Step 7: Kinesis Firehose Setup](#step-7-kinesis-firehose-setup)
- [Step 8: Setup Athena for Querying S3 Data](#step-8-setup-athena-for-querying-s3-data)

## Prerequisites

- AWS Account
- IAM User with necessary permissions
- AWS CLI configured

## Step 1: Mock Data Generator for Input

This step involves creating a mock data generator that will create schema and input records. The generated data will be inserted into a DynamoDB table. The data is inserting into dynamoDB via python file (mock_data_generator_for_dynamodb.py)

## Step 2: DynamoDB Table Creation

1. Create a DynamoDB table named `OrdersRawTable`.
2. Define `orderid` as the partition key.
3. Keep the settings as default.
4. Add a tag: `dev`.
5. Create the table.
   

## Step 3: Enable DynamoDB Streams

- Enable DynamoDB Streams on the `OrdersRawTable`.
- Select `New image` and turn on the stream.
- This will capture any changes (inserts, updates, deletes) to items in the table.

  ![image](https://github.com/user-attachments/assets/c21aa8e5-12f5-4463-9a31-67a620111df1)

  
  ![image](https://github.com/user-attachments/assets/e78b90df-bb02-4a7e-84cf-d0246ca9b9e6)

 

  Using DynamoDB Streams with EventBridge before routing data to Kinesis Streams offers the flexibility to apply filtering and enrichment on the CDC changes. This approach allows you to handle specific event conditions, transforming or   filtering data as needed before it reaches the Kinesis Stream. It ensures that only relevant data is sent for further processing(in our case only new images) , which can optimize downstream processing and reduce unnecessary data load 
  on the Kinesis Stream. This     method also aligns with the design patterns you've applied in other assignments, making it a consistent approach in your architecture.

## Step 4: Create a Kinesis Data Stream

- Create a Kinesis Data Stream with default settings.

## Step 5: Create an EventBridge Pipe

1. Create an EventBridge Pipe that reads data from DynamoDB Streams and sends it to Kinesis.
2. Add `OrdersRawTable` as the source and the Kinesis Data Stream as the target.
3. Define the partition key in Kinesis using the `orderid` from the DynamoDB data.

## Step 6: IAM Role Configuration

- Create an IAM role with full access to DynamoDB, Kinesis, and EventBridge.
- Attach the necessary policies to allow the EventBridge Pipe to access Kinesis and DynamoDB.

## Step 7: Kinesis Firehose Setup

1. Create a Kinesis Firehose to batch and deliver data from the Kinesis Data Stream to an S3 bucket.
2. Add a transformation Lambda function to clean up the data before storing it in S3.
3. Configure buffer size and buffer interval for near real-time processing.

## Step 8: Setup Athena for Querying S3 Data

1. Create a Glue Crawler to catalog the data stored in S3.
2. Set up a custom classifier to properly parse the JSON data.
3. Use Athena to query the data and gain insights.

## Notes

- DynamoDB attributes are referred to as columns or fields.
- The partition key in DynamoDB acts as the primary key.
- Kinesis Stream shards are determined based on the partition key.
- All changes (inserts, updates, deletes) are captured in Kinesis, providing a complete trail for analysis.

## Conclusion

This project demonstrates the implementation of a real-time data pipeline using AWS services such as DynamoDB, Kinesis, and Athena. The pipeline captures changes in a DynamoDB table and streams them into Kinesis for further processing and analysis.
