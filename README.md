## Overview

This project involves utilizing AWS services like S3, Athena, and Python libraries (awswrangler, boto3, pyathena) to handle data ingestion and analysis tasks. The code reads CSV files, inserts data into an S3 bucket in Parquet format, creates corresponding tables in Amazon Athena, and performs SQL queries for data analysis.

## Table of Contents

- [Requirements](#requirements)
- [Setup](#setup)
- [MongoDB Setup](#mongodb-setup)
- [Scripts](#scripts)
- [Aggregations](#aggregations)

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

The script will load .csv data 


Then it wil insert those data into `cars_db.carros` and `cars_db.montadoras` using insert_many command.

# Aggregations
In Mongo Compass we will then JOIN carros and montadoras by the column 'Montadora' with the purpose of bringing column 'Pais' to our table.
Next we're going to count how many car models there are by 'Pais'
At the end we will export the resulting data to a .json

Here are the steps:
1) Joining data
```
[
    {
        $lookup: {
            from: "montadoras",
            localField: "Montadora",
            foreignField: "Montadora",
            as: "joined_data"
        }
    },
    {
        $unwind: "$joined_data"
    },
    {
        $project: {
            _id: 1,  
            Carro: 1, 
            Cor: 1,  
            Montadora: 1, 
            Pais: "$joined_data.Pais"  // Include the Pais field from the joined collection
        }
    }
]
```
2) Aggregating by 'Pais'
```

db.carros.aggregate([
  {
    $lookup: {
      from: "montadoras",
      localField: "Montadora",
      foreignField: "Montadora",
      as: "joined_data",
    },
  },
  {
    $unwind: "$joined_data",
  },
  {
    $project: {
      _id: 1,
      Carro: 1,
      Cor: 1,
      Montadora: 1,
      Pais: "$joined_data.Pais", // Include the Pais field from the joined collection
    },
  },
  {
    $group: {
      _id: "$Pais",
      Carros: { $sum: 1 }, // Count the number of documents in each group
    },
  },
  {
    $out: "carros_por_pais", // Output the result to a new collection named 'grouped_result'
  },
])
```

#### The last query creates a new collection named `carros_por_pais` which we will export to .json using the 'Export Data' button in Compass

![image](https://github.com/viniciusfjacinto/data-engineering-test/assets/87664450/60099351-61b5-45c2-98f3-be09c6f7ff99)

