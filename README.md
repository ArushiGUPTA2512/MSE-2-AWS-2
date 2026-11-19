# AWS Sales Data Analytics – CSV to Parquet Conversion and Performance Comparison

**Project:** Amazon Athena – CTAS, Parquet Conversion and Data Partitioning   
**AWS Region:** Asia Pacific (Mumbai) – ap-south-1

---

# 1. Project Overview

An e-commerce company has a large sales dataset stored in Amazon S3 and wants to improve query performance and reduce data scanning costs.

This project uses Amazon S3 and Amazon Athena to analyze sales data stored in CSV format. The original CSV dataset is converted into Parquet format using CTAS (Create Table As Select), with data partitioning based on year and month.

Finally, the same SQL queries are executed on both CSV and Parquet tables to compare their data scanning and execution performance.

# 2. Project Objectives

- Create an S3 bucket to store sales data.
- Upload the original CSV dataset to Amazon S3.
- Create an Athena database and external table.
- Execute SQL queries on CSV data.
- Calculate monthly sales and total orders.
- Convert CSV data into Parquet format using CTAS.
- Implement year and month partitioning.
- Compare CSV and Parquet query performance.

# 3. Dataset Information

**Dataset Name:** `Activity03_Athena_sales.csv`

The uploaded dataset was inspected before implementation.

| Dataset Property | Value |
|---|---|
| File Format | CSV |
| Total Records | 20,000 |
| Total Columns | 12 |
| First Date | 2025-01-01 |
| Last Date | 2026-06-30 |
| Years Covered | 2025 and 2026 |

### Dataset Columns

| Column | Data Type | Description |
|---|---|---|
| order_id | STRING | Unique order identifier |
| order_date | STRING | Date of order |
| customer_id | STRING | Customer identifier |
| product | STRING | Product name |
| category | STRING | Product category |
| quantity | INT | Quantity purchased |
| unit_price | DOUBLE | Price per unit |
| total_amount | DOUBLE | Total order amount |
| city | STRING | Customer city |
| payment_mode | STRING | Payment method |
| year | INT | Order year |
| month | INT | Order month |

# 4. AWS Services Used

| AWS Service | Purpose |
|---|---|
| Amazon S3 | Stores CSV and Parquet datasets |
| Amazon Athena | Runs SQL queries on S3 data |
| AWS Glue Data Catalog | Stores database and table metadata |

---

# PART A – CSV DATA STORAGE AND ANALYSIS

## STEP 2 – Select AWS Region

1. Open AWS Management Console.
2. Look at the top-right corner.
3. Click the Region dropdown.
4. Select Asia Pacific (Mumbai).
5. Confirm that the selected region is ap-south-1.

**Why are we selecting this region?**

S3 and Athena resources should be in the same region for this project.

**Expected Result:** Mumbai region is selected.

## STEP 3 – Open Amazon S3

1. Go to the AWS Console search bar.
2. Type S3.
3. Under Services, click Amazon S3.
4. Open the S3 dashboard.
5. Click Buckets from the left navigation.
6. Click Create bucket.

**Expected Result:** Create bucket configuration page opens.

## STEP 4 – Create an S3 Bucket

Configure the following settings:

| Field | Configuration |
|---|---|
| Bucket type | General purpose |
| Bucket name | athena-sales-project-arushi-2026 |
| Region | Asia Pacific (Mumbai) |
| Object Ownership | ACLs disabled |
| Block Public Access | Keep all enabled |
| Bucket Versioning | Disable |
| Default Encryption | Keep default |

1. Enter the bucket name.
2. Keep the recommended security settings.
3. Scroll to the bottom.
4. Click Create bucket.
5. Wait for the success message.

**Expected Result:** The bucket appears in the S3 bucket list.

## STEP 5 – Create the sales-csv Folder

1. Open the bucket `athena-sales-project-arushi-2026`.
2. Click Create folder.
3. Enter folder name:

   `sales-csv`

4. Keep other settings unchanged.
5. Click Create folder.

**Expected Result:**

```text
athena-sales-project-arushi-2026/
    └── sales-csv/
```

## STEP 6 – Upload the CSV File

1. Open the sales-csv folder.
2. Click Upload.
3. Click Add files.
4. Browse your computer.
5. Locate the dataset.
6. Select `Activity03_Athena_sales.csv`.
7. Click Open.
8. Scroll down.
9. Click Upload.
10. Wait for upload completion.
11. Click Close.

**Expected Result:**

```text
sales-csv/
    └── Activity03_Athena_sales.csv
```

**What did we achieve?**

Uploaded the original 20,000-record sales dataset into Amazon S3 without modifying the CSV file.

## STEP 7 – Open Amazon Athena

1. Go back to AWS Management Console.
2. Click the search bar.
3. Type Athena.
4. Select Amazon Athena.
5. Open the Athena dashboard.
6. Click Query editor from the left navigation.

**Expected Result:**

You should see:
- Query editor
- SQL writing area
- Database dropdown
- Run button
- Recent queries

## STEP 8 – Configure Athena Query Result Location

Athena needs an S3 location to store query results.

1. Open Athena Query Editor.
2. Click Settings.
3. Find Query result location.
4. Click Edit if required.
5. Enter:

```text
s3://athena-sales-project-arushi-2026/athena-results/
```

6. Keep other settings at default.
7. Click Save.

**Expected Result:** Query result location is saved.

## STEP 9 – Create the Database sales_db

1. Open Query Editor.
2. Select the Editor tab.
3. Paste the following SQL query:

```sql
CREATE DATABASE IF NOT EXISTS sales_db;
```

4. Click Run.
5. Wait for successful execution.
6. Select `sales_db` from the Database dropdown.

**Expected Result:** Database sales_db is created successfully.

The database metadata is stored in AWS Glue Data Catalog.

## STEP 10 – Create the CSV External Table sales_csv

Make sure the selected database is `sales_db`.

Execute:

```sql
CREATE EXTERNAL TABLE sales_csv (
    order_id STRING,
    order_date STRING,
    customer_id STRING,
    product STRING,
    category STRING,
    quantity INT,
    unit_price DOUBLE,
    total_amount DOUBLE,
    city STRING,
    payment_mode STRING,
    year INT,
    month INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION 's3://athena-sales-project-arushi-2026/sales-csv/'
TBLPROPERTIES (
    'skip.header.line.count'='1'
);
```

1. Click Run.
2. Wait for successful execution.
3. Refresh the Tables list on the left if required.

**Expected Result:** A table named `sales_csv` appears.

### Explanation of SQL Components

| SQL Component | Purpose |
|---|---|
| CREATE EXTERNAL TABLE | Creates an external table definition |
| sales_csv | Table name |
| STRING | Text data type |
| INT | Integer data type |
| DOUBLE | Decimal data type |
| FIELDS TERMINATED BY ',' | Identifies comma-separated fields |
| STORED AS TEXTFILE | Reads CSV as text data |
| LOCATION | Connects the table to S3 |
| skip.header.line.count | Ignores the CSV header |

**Important:** An external table stores metadata and reads the original data directly from S3. It does not copy the CSV data into Athena.

## STEP 11 – Verify the CSV Table

Execute:

```sql
SELECT *
FROM sales_csv
LIMIT 10;
```

1. Click Run.
2. Check the Results section.

**Expected Result:**
- 10 rows displayed.
- 12 columns displayed.
- Column names match the uploaded CSV dataset.

### Verify Total Records

Execute:

```sql
SELECT COUNT(*) AS total_records
FROM sales_csv;
```

**Expected Result:**

| total_records |
|---:|
| 20,000 |

**What did we achieve?**

Verified that Athena can successfully read all 20,000 records from the CSV dataset.

## STEP 12 – Execute Monthly Sales Query (Task A)

**Requirement:** Calculate the total number of orders and total sales for January 2025.

Execute:

```sql
SELECT
    year,
    month,
    COUNT(*) AS total_orders,
    SUM(total_amount) AS total_sales
FROM sales_csv
WHERE year = 2025
AND month = 1
GROUP BY year, month;
```

Click Run.

### Actual Dataset Result – January 2025

| Column | Result |
|---|---:|
| year | 2025 |
| month | 1 |
| total_orders | 1,184 |
| total_sales | 4,004,766 |

### SQL Explanation

- SELECT: Selects required columns.
- COUNT(*): Counts sales records.
- SUM(total_amount): Calculates total sales.
- WHERE: Filters January 2025.
- GROUP BY: Groups the results by year and month.

**What did we achieve?**

Successfully calculated January 2025 sales from the original CSV dataset.

**Task A Completed.**

## STEP 13 – Record CSV Data Scanned and Execution Time

This step is important for comparing CSV and Parquet performance.

1. Open Athena Query Editor.
2. Go to Recent queries.
3. Locate the January 2025 query executed on `sales_csv`.
4. Click its Execution ID.
5. Open query execution details.
6. Find:
   - Data scanned
   - Query execution time
7. Record both values.

### CSV Query Performance

| Metric | Actual Value |
|---|---|
| Data scanned | To be recorded from Athena |
| Query execution time | To be recorded from Athena |

**Important:** Use actual execution statistics. Do not enter assumed values.

Save screenshots of the CSV query result and execution statistics.

---

# PART B – CTAS, PARQUET CONVERSION AND DATA PARTITIONING

## STEP 14 – Create the Parquet Output Folder

1. Open Amazon S3.
2. Click Buckets.
3. Open `athena-sales-project-arushi-2026`.
4. Click Create folder.
5. Enter:

   `sales-parquet`

6. Click Create folder.

**Expected S3 Structure:**

```text
athena-sales-project-arushi-2026/
│
├── sales-csv/
│   └── Activity03_Athena_sales.csv
│
└── sales-parquet/
```

**Important:** Keep the sales-parquet folder empty before executing CTAS.

## STEP 15 – Understand CTAS

CTAS stands for Create Table As Select.

It creates a new table using the results of an existing SELECT query.

In this project, CTAS will:

1. Read data from sales_csv.
2. Convert CSV data into Parquet format.
3. Store Parquet files in Amazon S3.
4. Partition data by year and month.
5. Create a new table named sales_parquet.

### Benefits of Parquet

- Columnar storage format.
- Efficient compression.
- Reduced data scanning for suitable queries.
- Improved analytical query performance.

## STEP 16 – Run the CTAS Query

1. Open Amazon Athena.
2. Go to Query Editor.
3. Select database `sales_db`.
4. Paste the following query:

```sql
CREATE TABLE sales_parquet
WITH (
    format = 'PARQUET',
    external_location = 's3://athena-sales-project-arushi-2026/sales-parquet/',
    partitioned_by = ARRAY['year', 'month']
)
AS
SELECT
    order_id,
    order_date,
    customer_id,
    product,
    category,
    quantity,
    unit_price,
    total_amount,
    city,
    payment_mode,
    year,
    month
FROM sales_csv;
```

5. Click Run.
6. Wait for successful execution.

**Expected Result:** A new table named `sales_parquet` is created.

### Important Checks

- Format must be PARQUET.
- External location must point to sales-parquet/.
- year and month must be the last two columns in SELECT.

**What happened?**

Athena read the CSV data and converted it into Parquet files organized by year and month.

## STEP 17 – Verify Parquet Files in S3

1. Open Amazon S3.
2. Open your bucket.
3. Open sales-parquet.
4. Check the partition folders.
5. Open year=2025.
6. Open month=1.
7. Check the files inside.

**Expected Folder Structure:**

```text
sales-parquet/
│
├── year=2025/
│   ├── month=1/
│   ├── month=2/
│   ├── month=3/
│   └── ...
│
└── year=2026/
    ├── month=1/
    ├── month=2/
    ├── month=3/
    └── ...
```

You should find files with the `.parquet` extension.

### What is Partitioning?

Partitioning divides data into separate folders based on column values.

Examples:

- year=2025/month=1 contains January 2025 records.
- year=2025/month=2 contains February 2025 records.
- year=2026/month=1 contains January 2026 records.

This allows Athena to skip irrelevant folders when queries filter by year and month.

**Task B Completed.**

## STEP 18 – Verify the Parquet Table in Athena

1. Open Athena Query Editor.
2. Select database sales_db.
3. Refresh the Tables section.
4. Find sales_parquet.

### Display Available Tables

Execute:

```sql
SHOW TABLES;
```

**Expected Result:**

```text
sales_csv
sales_parquet
```

### Verify Parquet Records

Execute:

```sql
SELECT *
FROM sales_parquet
LIMIT 10;
```

Click Run.

**Expected Result:** Athena displays 10 records from the Parquet table.

The records should match the original CSV data.

---

# PART C – RUN THE SAME QUERY ON PARQUET

## STEP 19 – Execute January 2025 Query on sales_parquet

Execute the same monthly query, changing only the table name.

```sql
SELECT
    year,
    month,
    COUNT(*) AS total_orders,
    SUM(total_amount) AS total_sales
FROM sales_parquet
WHERE year = 2025
AND month = 1
GROUP BY year, month;
```

Click Run.

### Expected Result – January 2025

| Column | Result |
|---|---:|
| year | 2025 |
| month | 1 |
| total_orders | 1,184 |
| total_sales | 4,004,766 |

**What did we achieve?**

Successfully executed the same analytical query on the Parquet table.

Both tables should return the same result, proving that the conversion preserved the sales information.

**Task C Completed.**

## STEP 20 – Record Parquet Data Scanned and Execution Time

1. Open Athena Query History or Recent queries.
2. Find the January 2025 query executed on sales_parquet.
3. Click its Execution ID.
4. Open query execution details.
5. Find:
   - Data scanned
   - Query execution time
6. Record both values.

### Parquet Query Performance

| Metric | Actual Value |
|---|---|
| Data scanned | To be recorded from Athena |
| Query execution time | To be recorded from Athena |

Save screenshots of the Parquet query result and execution statistics.

---

# PART D – PERFORMANCE COMPARISON

## STEP 21 – Compare CSV and Parquet Performance

We now have two tables:

- CSV table: sales_csv
- Parquet table: sales_parquet

Both queries use the same conditions:

- Year = 2025
- Month = 1
- COUNT(*) for total orders
- SUM(total_amount) for total sales

### Performance Comparison

| Metric | CSV | Parquet |
|---|---|---|
| Data scanned | Actual AWS value | Actual AWS value |
| Execution time | Actual AWS value | Actual AWS value |
| Total orders | 1,184 | 1,184 |
| Total sales | 4,004,766 | 4,004,766 |

### Data Scanned Reduction Formula

Data Scanned Reduction (%) =

((CSV Data Scanned - Parquet Data Scanned) / CSV Data Scanned) × 100

### Performance Analysis

Compare the actual data scanned and query execution time for both formats.

Parquet uses columnar storage and partitioning, which can reduce unnecessary data scanning for queries that filter on partition columns.

The actual performance improvement should be calculated from the Athena execution statistics.

---

# 5. Final Project Results

The following tasks were implemented:

1. Created an Amazon S3 bucket.
2. Uploaded the original sales CSV dataset.
3. Created the sales_db database.
4. Created the sales_csv external table.
5. Verified all 20,000 CSV records.
6. Calculated January 2025 sales.
7. Recorded CSV query statistics.
8. Created the sales-parquet output folder.
9. Converted CSV data into Parquet using CTAS.
10. Partitioned data by year and month.
11. Verified Parquet files in S3.
12. Created and verified the sales_parquet table.
13. Executed the same monthly query on Parquet.
14. Compared CSV and Parquet performance.

# 6. Conclusion

This project demonstrates how AWS cloud services can be used for data storage, processing and analytics.

Amazon S3 was used to store the original CSV dataset and converted Parquet files. Amazon Athena was used to create external tables, execute SQL queries and convert data using CTAS. AWS Glue Data Catalog maintained the database and table metadata.

The CSV and Parquet tables produced the same January 2025 sales results. Query execution statistics provide the basis for evaluating the performance differences between the two formats.

