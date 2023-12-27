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

It's important that you save your user information into environment variables using dotenv library.

https://sparkbyexamples.com/python/using-python-dotenv-load-environment-variables/

Also you need to habilitate your AWS access and secret keys to use then with libraries Boto3 (for inserting data and acessing S3 buckets) and Pyathena (for querying data). 

https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html

The script will load .csv files as Pandas dataframes and then insert it into the desired S3 Bucket, creating a new Athena table for querying every .csv individually like the image below:


![image](https://github.com/viniciusfjacinto/data-engineering-test/assets/87664450/f2588105-9814-4850-9acd-e1537a1acce8)

# Relationships
The schema below show how data tables are related to each other in Athena:

![image](https://github.com/viniciusfjacinto/data-engineering-test/assets/87664450/8f745a4a-cb83-43b8-b930-30fc1f74d8e6)

# Queries

Here we expose the queries that answers the five questions proposed in the test (in portuguese):

1.	Escreva uma query que retorna a quantidade de linhas na tabela Sales.SalesOrderDetail pelo campo SalesOrderID, desde que tenham pelo menos três linhas de detalhes.
```
SELECT SalesOrderID, COUNT(*) AS NumberOfDetails
FROM "teste-eng-jr".sales_orderdetail
GROUP BY SalesOrderID
HAVING COUNT(*) >= 3
```

2.	Escreva uma query que ligue as tabelas Sales.SalesOrderDetail, Sales.SpecialOfferProduct e Production.Product e retorne os 3 produtos (Name) mais vendidos (pela soma de OrderQty), agrupados pelo número de dias para manufatura (DaysToManufacture).
```
SELECT
    pp.Name AS ProductName,
    pp.DaysToManufacture,
    SUM(sod.OrderQty) AS TotalOrderQuantity
FROM
"teste-eng-jr".sales_orderdetail sod
INNER JOIN "teste-eng-jr".sales_specialofferproduct ssop
  ON sod.specialofferid = ssop.specialofferid and sod.productid = ssop.productid
INNER JOIN "teste-eng-jr".production_product pp
  ON pp.productid = ssop.productid
GROUP BY
    pp.Name, pp.DaysToManufacture
ORDER BY
    SUM(sod.OrderQty) DESC
LIMIT 3
```

3.	Escreva uma query ligando as tabelas Person.Person, Sales.Customer e Sales.SalesOrderHeader de forma a obter uma lista de nomes de clientes e uma contagem de pedidos efetuados.
```
SELECT
    pe.businessentityid as PersonID,
    CONCAT(coalesce(pe.FirstName,''), ' ', COALESCE(pe.middlename,''), ' ', COALESCE(pe.LastName,'')) AS CustomerName,
    COUNT(soh.SalesOrderID) AS NumberOfOrders
FROM
    "teste-eng-jr".Person_Person pe
INNER JOIN
    "teste-eng-jr".sales_customer sc ON pe.BusinessEntityID = sc.PersonID
INNER JOIN
    "teste-eng-jr".sales_orderheader soh ON sc.CustomerID = soh.CustomerID
GROUP BY
    1,2
ORDER BY
    COUNT(soh.SalesOrderID) desc
```

4.	Escreva uma query usando as tabelas Sales.SalesOrderHeader, Sales.SalesOrderDetail e Production.Product, de forma a obter a soma total de produtos (OrderQty) por ProductID e OrderDate.
```
SELECT
    sod.ProductID,
    pp.name ProductName,
    soh.OrderDate,
    SUM(sod.OrderQty) AS TotalOrderQuantity
FROM
"teste-eng-jr".sales_orderheader soh
INNER JOIN "teste-eng-jr".sales_orderdetail sod
ON sod.salesorderid = soh.salesorderid
INNER JOIN "teste-eng-jr".production_product pp
ON pp.productid = sod.productid
GROUP BY
    sod.ProductID, pp.name, soh.OrderDate
```

5.	Escreva uma query mostrando os campos SalesOrderID, OrderDate e TotalDue da tabela Sales.SalesOrderHeader. Obtenha apenas as linhas onde a ordem tenha sido feita durante o mês de setembro/2011 e o total devido esteja acima de 1.000. Ordene pelo total devido decrescente.

```
SELECT
    SalesOrderID,
    OrderDate,
    TotalDue
FROM
    "teste-eng-jr".sales_orderheader
WHERE
    YEAR(CAST(OrderDate as TIMESTAMP)) = 2011
    AND MONTH(CAST(OrderDate AS TIMESTAMP)) = 9
    AND TotalDue > 1000
ORDER BY
    TotalDue DESC;
```
