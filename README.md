# etl-pipeline-aws-sdk-pandas
Performing the extract stage of an ETL data pipeline with AWS SDK for Pandas.


## Intro

Let's assume we have the following resources deployed in an AWS environment:

- An Amazon S3 bucket, where transaction data is added to the bucket in the JSON data format representing an order. Item data for each transaction is added to the bucket in the CSV (comma-separated values) format. The transaction and items can be linked together using the _transaction_id_ field common to both sets of data.
- An Amazon RDS MySQL database instance, which contains store data with a _store_id_.
- An AWS SecretsManager secret containing credentials for the RDS database instance.
- A DynamoDB table, which records when a customer was last active on the fictional company's website. A _customer_id_ field is used to link a DynamoDB record with a transaction.
- A CloudWatch Logs log group, stores a fraud score for the transaction, representing how likely the transaction is to be fraudulent with a rating between 0 and 100.



