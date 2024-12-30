# etl-pipeline-aws-sdk-pandas
Performing the extract stage of an ETL data pipeline with AWS SDK for Pandas.


## Intro

Let's assume we have the following resources deployed in an AWS environment:

- An Amazon S3 bucket, where transaction data is added to the bucket in the JSON data format representing an order. Item data for each transaction is added to the bucket in the CSV (comma-separated values) format. The transaction and items can be linked together using the _transaction_id_ field common to both sets of data.
- An Amazon RDS MySQL database instance, which contains store data with a _store_id_.
- An AWS SecretsManager secret containing credentials for the RDS database instance.
- A DynamoDB table, which records when a customer was last active on the fictional company's website. A _customer_id_ field is used to link a DynamoDB record with a transaction.
- A CloudWatch Logs log group, stores a fraud score for the transaction, representing how likely the transaction is to be fraudulent with a rating between 0 and 100.
- An AWS Lambda configured to process new objects as they are added to the transactions prefix in the Amazon S3 bucket using Amazon EventBridge.

The goal is to assemble a Lambda function that can be used to extract the data and store it in S3 in the Parquet format. This represents the Extract step of an Extract, Transform, and Load (ETL) pipeline.

The _jsonfilepath_ Jupyter notebook returns a list of all JSON files in the transaction prefix.
  
## Extracting the Transaction Data

The _fetchdata_ Jupyter notebook defines a function that combines data from multiple sources to create a rich transaction record:

- Store data from a database
- Transaction data from S3
- Loyalty data embedded in the transaction
- Fraud score from a CloudWatch Logs log group
- Customer activity data from DynamoDB

The result is a single DataFrame containing all relevant information about the transaction.

## Writing Extracted Transaction Data to S3

The next step after creating a data frame with the combined transaction data, is to load the data in a location where it can be used or transformed further.

It's common when working with data pipelines to use a columnar format to store data. Parquet is an example of a columnar format. Using it has the following benefits:

- It's usually quicker to read than row or record based alternatives
- Enables efficient access of just the required columns
- Columnar data formats usually require less disk space than row or record based options
  
In AWS, Amazon S3 is often used to store data either as a final or intermediate step in a data pipeline. It's also common to organize the data in a way that makes reading and querying it easier, this is called partitioning.
A good general way to partition data is to do so by date, since looking up data by date ranges is common when querying data.

The _loadfunction_ Jupyter notebook defines a function named _load_data_ that takes a transaction data frame, an S3 bucket name, and a prefix. The data frame is written to a location in the bucket based upon the current date.

## Developing a Lambda handler

The last part of the lambda function development is the handler, which serves as the entry point into the function's code.

The _lambdahandler_ Jupyter notebook does the following:

- Extracts the bucket name and object key from the event payload message
- Checks if the object key ends with .json 
- Calls the extract_data and load_data functions you defined previously

We can define a sample EventBridge event and call the _lambda_handler_ function, which uses a Boto3 S3 client to list objects under the extracted prefix in the Amazon S3 bucket.

```
lambda_handler(sample_event, None)

response = boto3.client('s3').list_objects(Bucket=bucket_name, Prefix='extracted/')
print([obj['Key'] for obj in response['Contents']])
```

## Architecture


![Untitled Diagram drawio](https://github.com/user-attachments/assets/37c4dbc1-de02-475f-8b6f-29b6078ff8e3)
