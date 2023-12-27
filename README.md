## Overview

This project involves utilizing AWS services like S3, Athena, and Python libraries (awswrangler, boto3, pyathena) to handle data ingestion and analysis tasks. The code reads CSV files, inserts data into an S3 bucket in Parquet format, creates corresponding tables in Amazon Athena, and performs SQL queries for data analysis.

## Table of Contents

- [Requirements](#requirements)
- [Setup](#setup)
- [AWS Structure](#aws-structure)
- [Relationships](#relationships)
- [Queries](#queries)

## Requirements
- AWS account and services: S3, Athena
- Python libraries: `pandas`,`pyathena`, `awswrangler`, `boto3`, `os`, `dotenv`

## Setup
Install necessary Python libraries using ```pip install -r requirements.txt```.
Set up environmental variables for AWS credentials (AWS_ACCESS_KEY, AWS_SECRET_KEY, AWS_REGION, etc.).
Ensure CSV files (Person.Person.csv, Production.Product.csv, etc.) are accessible in the specified location.

## AWS Structure

### S3 Buckets
Ensure you have a S3 Bucket created with the desired name. Data files (such as CSV, Parquet, JSON) are always stored in S3 buckets as they can serve as an Datalake receiving structured and non-structured data. These files commonly should be raw data, logs, or structured data files generated from various sources and further transformations.

### Amazon Athena
S3 buckets + Athena query console offers an efficient and low-cost strategy for creating fast analysis as Athena directly queries data in S3 without needing a predefined schema or database setup, eliminating traditional database maintenance costs.

# Scripts
Follow [teste_eng_jr.ipynb](https://github.com/viniciusfjacinto/data-engineering-test/blob/main/teste_eng_jr.ipynb) code instructions.

It's important that you save your user information into environment variables using dotenv library

The script will load .csv files as Pandas dataframes and then insert it into the desired S3 Bucket, creating a new Athena table for querying every .csv individually like the image below:
![image](https://github.com/viniciusfjacinto/data-engineering-test/assets/87664450/f2588105-9814-4850-9acd-e1537a1acce8)


# Relationships

