## Overview

The project was created to be used as a test for a Data Engineer position and involves using AWS services like S3, Athena, and Python libraries (awswrangler, boto3, pyathena) to handle data ingestion and analysis tasks. The code reads CSV files, inserts data into a S3 bucket in Parquet format, creates corresponding tables in Amazon Athena, and performs SQL queries for data analysis according to business questions.

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

![image](https://github.com/viniciusfjacinto/data-engineering-test/assets/87664450/a8aff113-ac02-4640-b3bb-45aa7f83a940)

### S3 Buckets

Ensure you have a S3 Bucket created with the desired name. Data files (such as CSV, Parquet, JSON) are always stored in S3 buckets as they can serve as an Datalake receiving structured and non-structured data. These files commonly should be raw data, logs, or structured data files generated from various sources and further transformations.

### Amazon Athena
S3 buckets + Athena query console offers an efficient and low-cost strategy for creating fast analysis as Athena directly queries data in S3 without needing a predefined schema or database setup, eliminating traditional database maintenance costs.

# Scripts
Follow [main.ipynb](https://github.com/viniciusfjacinto/data-engineering-test/blob/main/main.ipynb) code instructions.

It's important that you save your user information into environment variables using dotenv library.

https://sparkbyexamples.com/python/using-python-dotenv-load-environment-variables/

Also you need to habilitate your AWS access and secret keys to use then with libraries Boto3 (for inserting data and acessing S3 buckets) and Pyathena (for querying data). 

https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html

The script will load .csv files as Pandas dataframes and then insert it into the desired S3 Bucket, creating a new Athena table for querying every .csv individually like the image below:

![image](https://github.com/viniciusfjacinto/data-engineering-aws/assets/87664450/74f60822-4f75-445f-b24b-7bedee5ebd0f)

# Relationships
The schema below show how data tables are related to each other in Athena:

![image](https://github.com/viniciusfjacinto/data-engineering-test/assets/87664450/8f745a4a-cb83-43b8-b930-30fc1f74d8e6)

# Queries

Here we expose the queries that answers the five questions proposed in the test (in portuguese):

1.	Write a query that returns the number of rows in the Sales.SalesOrderDetail table by the SalesOrderID field, provided they have at least three detail rows.
```
SELECT SalesOrderID, COUNT(*) AS NumberOfDetails
FROM "raw".sales_orderdetail
GROUP BY SalesOrderID
HAVING COUNT(*) >= 3
```

2.	Write a query that joins the tables Sales.SalesOrderDetail, Sales.SpecialOfferProduct, and Production.Product and returns the top 3 products (by Name) sold the most (by the sum of OrderQty), grouped by the number of days to manufacture (DaysToManufacture).
```
SELECT
    pp.Name AS ProductName,
    pp.DaysToManufacture,
    SUM(sod.OrderQty) AS TotalOrderQuantity
FROM
"raw".sales_orderdetail sod
INNER JOIN "raw".sales_specialofferproduct ssop
  ON sod.specialofferid = ssop.specialofferid and sod.productid = ssop.productid
INNER JOIN "raw".production_product pp
  ON pp.productid = ssop.productid
GROUP BY
    pp.Name, pp.DaysToManufacture
ORDER BY
    SUM(sod.OrderQty) DESC
LIMIT 3
```

3.	Write a query joining the tables Person.Person, Sales.Customer, and Sales.SalesOrderHeader to obtain a list of customer names and a count of orders placed.
```
SELECT
    pe.businessentityid as PersonID,
    CONCAT(coalesce(pe.FirstName,''), ' ', COALESCE(pe.middlename,''), ' ', COALESCE(pe.LastName,'')) AS CustomerName,
    COUNT(soh.SalesOrderID) AS NumberOfOrders
FROM
    "raw".Person_Person pe
INNER JOIN
    "raw".sales_customer sc ON pe.BusinessEntityID = sc.PersonID
INNER JOIN
    "raw".sales_orderheader soh ON sc.CustomerID = soh.CustomerID
GROUP BY
    1,2
ORDER BY
    COUNT(soh.SalesOrderID) desc
```

4.	Write a query using the tables Sales.SalesOrderHeader, Sales.SalesOrderDetail, and Production.Product to obtain the total sum of products (OrderQty) per ProductID and OrderDate.
```
SELECT
    sod.ProductID,
    pp.name ProductName,
    soh.OrderDate,
    SUM(sod.OrderQty) AS TotalOrderQuantity
FROM
"raw".sales_orderheader soh
INNER JOIN "raw".sales_orderdetail sod
ON sod.salesorderid = soh.salesorderid
INNER JOIN "raw".production_product pp
ON pp.productid = sod.productid
GROUP BY
    sod.ProductID, pp.name, soh.OrderDate
```

5.	Write a query showing the SalesOrderID, OrderDate, and TotalDue fields from the Sales.SalesOrderHeader table. Obtain only the rows where the order was made during September 2011 and the total due is above 1,000. Order by TotalDue descending.

```
SELECT
    SalesOrderID,
    OrderDate,
    TotalDue
FROM
    "raw".sales_orderheader
WHERE
    YEAR(CAST(OrderDate as TIMESTAMP)) = 2011
    AND MONTH(CAST(OrderDate AS TIMESTAMP)) = 9
    AND TotalDue > 1000
ORDER BY
    TotalDue DESC;
```
