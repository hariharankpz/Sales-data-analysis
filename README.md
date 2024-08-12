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

# Step 1: Mock Data Generator for Input

This step involves creating a mock data generator that will create schema and input records. The generated data will be inserted into a DynamoDB table. The data is inserting into dynamoDB via python file (mock_data_generator_for_dynamodb.py)

# Step 2: DynamoDB Table Creation

1. Create a DynamoDB table named `OrdersRawTable`.
2. Define `orderid` as the partition key.
3. Keep the settings as default.
4. Add a tag: `dev`.
5. Create the table.
   ![image](https://github.com/user-attachments/assets/b09827ff-8d27-4439-9b0f-b0dab9e327b0)

   

# Step 3: Enable DynamoDB Streams

- Enable DynamoDB Streams on the `OrdersRawTable`.
- Select `New image` and turn on the stream.
- This will capture any changes (inserts, updates, deletes) to items in the table.

   ![image](https://github.com/user-attachments/assets/10cd45e0-cbfb-4f4d-a2cb-36a8d6449c97)


   ![image](https://github.com/user-attachments/assets/df1362e1-e00c-4528-8427-d56c1ea414b6)


 

  Using DynamoDB Streams with EventBridge before routing data to Kinesis Streams offers the flexibility to apply filtering and enrichment on the CDC changes. This approach allows you to handle specific event conditions, transforming or   filtering data as needed before it reaches the Kinesis Stream. It ensures that only relevant data is sent for further processing(in our case only new images) , which can optimize downstream processing and reduce unnecessary data load 
  on the Kinesis Stream. This     method also aligns with the design patterns you've applied in other assignments, making it a consistent approach in your architecture.

# Step 4: Create a Kinesis Data Stream

- Create a Kinesis Data Stream with default settings.
  ![image](https://github.com/user-attachments/assets/f10a4877-b341-4b61-b8d4-0a3236671d4c)


# Step 5: Create an EventBridge Pipe

1. Create an EventBridge Pipe that reads data from DynamoDB Streams and sends it to Kinesis.
2. Add `OrdersRawTable` as the source and the Kinesis Data Stream as the target.
3. Define the partition key in Kinesis using the `orderid` from the DynamoDB data. Partition key: $.dynamodb.Keys.orderid.S
  ![image](https://github.com/user-attachments/assets/75941f3e-bcf0-419c-992f-7e18d3fe7a8a)
  ![image](https://github.com/user-attachments/assets/7b50b8f8-e974-4b1c-a9a0-f617dcf17c24)




# Step 6: IAM Role Configuration

- Create an IAM role with full access to DynamoDB, Kinesis, and EventBridge.
- Attach the necessary policies to allow the EventBridge Pipe to access Kinesis and DynamoDB.

# Step 7: Kinesis Firehose Setup

## 1. Create a Kinesis Firehose Delivery Stream

Create a Kinesis Firehose delivery stream to batch and deliver data from the Kinesis Data Stream to an S3 bucket.

![Kinesis Firehose Setup](https://github.com/user-attachments/assets/373a5d4f-139e-4ea9-8525-71cf18993dc1)

## 2. Add a Transformation Lambda Function

Add a transformation Lambda function to clean up the data before storing it in S3. The data coming from the source may contain unnecessary metadata fields that need to be removed or modified.

- Create a Lambda function that runs in the Python runtime.
- Refer to the script in `transformation_layer_with_lambda.py` for the Lambda function code.
- Ensure that the Lambda function has the necessary permissions to access Kinesis Firehose.

![Lambda Transformation Setup](https://github.com/user-attachments/assets/537d9bee-4c79-470b-8d16-aa04e11712d7)

- Select the option below and choose the Lambda function (Transformer Lambda function - `transformation_layer_with_lambda.py`).

![Lambda Function Selection](https://github.com/user-attachments/assets/12fb5ec1-9571-40ed-976d-3b94287ee09c)

## 3. Configure Buffer Size and Buffer Interval for Near Real-Time Processing

When using AWS Lambda to transform your data in Amazon Kinesis Data Firehose before delivering it to the destination, you can configure two important settings: **Buffer Size** and **Buffer Interval**. These settings control how Kinesis Data Firehose batches the incoming data before sending it to your Lambda function for processing.

### Buffer Size

The buffer size parameter specifies the amount of data, in MBs, that Kinesis Data Firehose buffers before invoking your Lambda function. This means that once the buffer accumulates the specified amount of data, Kinesis Data Firehose triggers your Lambda function to process the buffered data.

- **Minimum value:** 1 MB
- **Maximum value:** 3 MB

### Buffer Interval

The buffer interval parameter specifies the maximum amount of time, in seconds, that Kinesis Data Firehose buffers incoming data before invoking your Lambda function. If the buffer size is not reached within this interval, Kinesis Data Firehose triggers your Lambda function based on the time interval.

- **Minimum value:** 60 seconds (1 minute)
- **Maximum value:** 900 seconds (15 minutes)

### How They Work Together

Kinesis Data Firehose will invoke your Lambda function to process buffered data when either the buffer size or the buffer interval condition is met, whichever comes first. This ensures that your data is processed in a timely manner without being delayed indefinitely.

#### Example Configuration

If you set:
- **Buffer Size:** 3 MB
- **Buffer Interval:** 300 seconds (5 minutes)

Kinesis Data Firehose will invoke your Lambda function as soon as 3 MB of data is buffered or 5 minutes have passed since the last invocation, whichever happens first.

![Buffer Configuration](https://github.com/user-attachments/assets/d419e9d8-f4e5-46a3-be58-a3d1ebf537d3)

## 4. Add S3 Bucket for Data Storage

Finally, configure the S3 bucket where the transformed data from Kinesis Firehose will be stored. Enable Dynamic Partitioning to store data in a Hive-like partitioned structure.

![S3 Setup](https://github.com/user-attachments/assets/0684acc7-8852-4d58-b051-e2426ec161ec)




# Step 8: Setup Athena for Querying S3 Data

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
