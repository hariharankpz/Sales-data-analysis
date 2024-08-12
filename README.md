# Real-Time Sales Data Analysis

## Project Overview

This project aims to develop a real-time sales data analysis system to monitor and analyze sales transactions as they occur. The system enables immediate insights into sales performance, customer behavior, and inventory status. It provides real-time dashboards and supports ad-hoc queries to facilitate quick decision-making and operational efficiency.

## Architecture Diagram

![Uploading image.png…]()


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

