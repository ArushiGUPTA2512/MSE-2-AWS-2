# AWS Sales Data Analytics: CSV vs Parquet Performance Benchmarking

**Project Repository:** MSE-2-AWS-2  
**Platform:** Amazon Web Services (AWS)  
**Region:** Asia Pacific (Mumbai) – ap-south-1  
**Database:** sales_db

---

# 1. Project Overview

This project demonstrates how to store, process, and analyze sales data using Amazon S3 and Amazon Athena.

The project uses a CSV dataset stored in Amazon S3. Amazon Athena is used to create external tables and execute SQL queries on the dataset. The CSV data is then converted into Parquet format using CTAS (Create Table As Select) and partitioned by year and month.

Finally, the same analytical query is executed on both CSV and Parquet tables to compare data scanned and query execution time.

## 2. Project Objectives

- Store sales data in Amazon S3.
- Create an Athena database and external table.
- Execute SQL queries on CSV data.
- Calculate monthly sales and total orders.
- Convert CSV data into Parquet format.
- Implement year and month partitioning.
- Compare CSV and Parquet query performance.

## 3. AWS Services Used

| Service | Purpose |
|---|---|
| Amazon S3 | Stores CSV and Parquet datasets |
| Amazon Athena | Executes SQL queries |
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

S3 and Athena resources should be configured in the same region for this project.

**Expected Result:** AWS Console displays Mumbai as the selected region.

## STEP 3 – Open Amazon S3

1. Go to the AWS Console search bar.
2. Type S3.
3. Under Services, select Amazon S3.
4. Open the S3 dashboard.
5. Click Buckets from the left navigation.
6. Click Create bucket.

**Expected Result:** The Create bucket configuration page opens.

## STEP 4 – Create an S3 Bucket

Configure the bucket settings:

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

**Expected Result:** Your bucket appears in the S3 bucket list.

**What did we achieve?**

Created an S3 bucket to store the original CSV dataset, converted Parquet files and Athena query results.

## STEP 5 – Create the sales-csv Folder

1. Open the bucket:
   athena-sales-project-arushi-2026
2. Click Create folder.
3. Enter the folder name:

   sales-csv

4. Keep other options unchanged.
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
5. Locate the extracted dataset.
6. Select Activity03_Athena_sales.csv.
7. Click Open.
8. Scroll down on the S3 upload page.
9. Click Upload.
10. Wait for upload completion.
11. Click Close.

**Expected Result:**

```text
sales-csv/
    └── Activity03_Athena_sales.csv
```

**What did we achieve?**

Uploaded the original sales dataset to Amazon S3 without changing the CSV file.

## STEP 7 – Open Amazon Athena

1. Go back to AWS Management Console.
2. Click the search bar.
3. Type Athena.
4. Select Amazon Athena.
5. Open the Athena dashboard.
6. Click Query editor in the left navigation.

**Expected Result:**

You should see:
- Query editor
- SQL writing area
- Database dropdown
- Run button
- Recent queries / Query history

We will use the SQL editor for the remaining queries.

## STEP 8 – Configure Athena Query Result Location

Athena requires an S3 location to store query output.

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

**Expected Result:** The query result location is saved.

**Why is this required?**

Whenever Athena executes a query, it stores query output and execution metadata in the configured S3 location.

Note: You do not need to manually create the athena-results folder.

## STEP 9 – Create the Database sales_db

1. Open Athena Query Editor.
2. Select the Editor tab.
3. Click inside the SQL editor.
4. Execute the following query:

```sql
CREATE DATABASE IF NOT EXISTS sales_db;
```

5. Click Run.
6. Wait for successful execution.
7. Select sales_db from the Database dropdown.

**Expected Result:**

Query successful.

The database sales_db is now available in the AWS Glue Data Catalog.

## STEP 10 – Create the CSV External Table sales_csv

1. Make sure the selected database is sales_db.
2. Open the SQL editor.
3. Paste the following query:

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

4. Click Run.
5. Wait for query execution.
6. Refresh the Tables list on the left if necessary.

**Expected Result:**

A new table named sales_csv appears.

### Explanation of SQL Components

| SQL Component | Purpose |
|---|---|
| CREATE EXTERNAL TABLE | Creates an external table definition |
| sales_csv | Name of the table |
| STRING | Text data type |
| INT | Integer data type |
| DOUBLE | Decimal data type |
| FIELDS TERMINATED BY ',' | Identifies comma-separated fields |
| STORED AS TEXTFILE | Reads CSV as text data |
| LOCATION | Connects the table to S3 |
| skip.header.line.count | Ignores the CSV header |

**Important:** An external table does not copy the data into Athena. It stores the table definition and reads the data directly from S3.

## STEP 11 – Verify the CSV Table

Execute the following query:

```sql
SELECT *
FROM sales_csv
LIMIT 10;
```

1. Click Run.
2. Check the Results section.

**Expected Result:**

Athena should display 10 rows and 12 columns.

Columns include:
- order_id
- order_date
- customer_id
- product
- category
- quantity
- unit_price
- total_amount
- city
- payment_mode
- year
- month

### Verify Total Records

Execute:

```sql
SELECT COUNT(*) AS total_records
FROM sales_csv;
```

**Expected Result:** 20,000 records.

**What did we achieve?**

Verified that Athena can read the CSV file correctly from Amazon S3.

## STEP 12 – Execute Monthly Sales Query (Task A)

**Requirement:** Calculate the number of orders and total sales for January 2025.

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

### Expected Result – January 2025

| Column | Result |
|---|---:|
| year | 2025 |
| month | 1 |
| total_orders | 1,184 |
| total_sales | 4,004,766 |

### SQL Explanation

- SELECT: Selects the required columns.
- COUNT(*): Counts the number of records.
- SUM(total_amount): Calculates total sales.
- WHERE: Filters January 2025.
- GROUP BY: Groups results by year and month.

**What did we achieve?**

Successfully queried January 2025 sales data from the original CSV table.

**Task A Completed.**

## STEP 13 – Record CSV Data Scanned and Execution Time

This step is important because we will compare CSV and Parquet performance.

1. Open Athena Query Editor.
2. Go to Recent queries or Query history.
3. Locate the January 2025 query executed on sales_csv.
4. Click its Execution ID.
5. Open query execution details.
6. Find:
   - Data scanned
   - Query execution time
7. Record both values.

### CSV Query Performance

| Metric | Value |
|---|---|
| Data scanned | To be recorded from AWS |
| Execution time | To be recorded from AWS |

**Important:** Do not enter assumed values. Use the actual execution statistics from Athena.

Save screenshots of:
- CSV monthly query result.
- CSV query execution statistics.

---

# PART B – CTAS, PARQUET CONVERSION AND DATA PARTITIONING

## STEP 14 – Create the Parquet Output Folder

1. Open AWS Console.
2. Search for S3.
3. Click Buckets.
4. Open your bucket:

   athena-sales-project-arushi-2026

5. Click Create folder.
6. Enter:

   sales-parquet

7. Click Create folder.

**Expected S3 Structure:**

```text
athena-sales-project-arushi-2026/
│
├── sales-csv/
│   └── Activity03_Athena_sales.csv
│
└── sales-parquet/
```

**Important:** Keep sales-parquet empty. Athena CTAS will write the converted files here.

## STEP 15 – Understand CTAS

CTAS stands for Create Table As Select.

It is used to create a new table using the results of an existing SELECT query.

In this project, CTAS will:

1. Read data from sales_csv.
2. Convert CSV data into Parquet format.
3. Save Parquet files in Amazon S3.
4. Divide the data into year and month partitions.
5. Create a new Athena table called sales_parquet.

### Why Parquet?

Parquet is a columnar storage format designed for analytical workloads.

Benefits:
- Efficient column-based storage.
- Better compression.
- Reduced data scanning for suitable queries.
- Improved analytical query performance.

## STEP 16 – Run the CTAS Query

1. Open AWS Console.
2. Go to Amazon Athena.
3. Open Query Editor.
4. Select database sales_db.
5. Paste the following SQL:

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

6. Click Run.
7. Wait for successful execution.

**Expected Result:**

A new table named sales_parquet appears.

### Important Checks

- Format must be PARQUET.
- External location must point to sales-parquet/.
- year and month must be the last two columns in SELECT.

**What happened?**

The CTAS operation read the CSV records and wrote converted Parquet files to Amazon S3, organized by year and month.

## STEP 17 – Verify Parquet Files in S3

1. Open Amazon S3.
2. Select your bucket.
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

You should find files with the .parquet extension.

### What is Partitioning?

Partitioning divides data into separate folders based on column values.

For example:
- year=2025/month=1 contains January 2025 records.
- year=2025/month=2 contains February 2025 records.
- year=2026/month=1 contains January 2026 records.

This allows Athena to skip irrelevant partitions when a query filters by year and month.

**Task B Completed.**

## STEP 18 – Verify the Parquet Table in Athena

1. Go to Amazon Athena.
2. Open Query Editor.
3. Select database sales_db.
4. Refresh the Tables section.
5. Find sales_parquet.

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

Both tables should appear.

### Verify Parquet Records

Execute:

```sql
SELECT *
FROM sales_parquet
LIMIT 10;
```

Click Run.

**Expected Result:**

Athena should display 10 records from the Parquet table.

The records should match the original CSV data.

---

# PART C – RUN THE SAME QUERY ON PARQUET

## STEP 19 – Execute January 2025 Query on sales_parquet

We will execute the same monthly sales query that was previously executed on the CSV table.

The only difference is the table name.

Execute:

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

Both tables should return the same result, showing that the conversion preserved the sales information.

**Task C Completed.**

## STEP 20 – Record Parquet Data Scanned and Execution Time

Now we need the performance details of the Parquet query.

1. Open Athena Query History or Recent queries.
2. Find the January 2025 query executed on sales_parquet.
3. Click its Execution ID.
4. Open query execution details.
5. Find:
   - Data scanned
   - Query execution time
6. Record the values.

### Parquet Query Performance

| Metric | Value |
|---|---|
| Data scanned | To be recorded from AWS |
| Execution time | To be recorded from AWS |

Save screenshots of:
- Parquet monthly query result.
- Parquet query execution statistics.

---

# PART D – PERFORMANCE COMPARISON

## STEP 21 – Compare CSV and Parquet Performance

Now we have two sets of execution details:

- CSV table: sales_csv
- Parquet table: sales_parquet

Both queries use the same conditions:
- Year = 2025
- Month = 1
- COUNT(*) for total orders
- SUM(total_amount) for total sales

### Performance Comparison Table

| Metric | CSV | Parquet |
|---|---|---|
| Data scanned | Enter actual value | Enter actual value |
| Execution time | Enter actual value | Enter actual value |
| Total orders | 1,184 | 1,184 |
| Total sales | 4,004,766 | 4,004,766 |

### Data Scanned Reduction

Use the following formula:

Data Scanned Reduction (%) =

((CSV Data Scanned - Parquet Data Scanned) / CSV Data Scanned) × 100

### Conclusion

The project demonstrates how CSV data can be converted into Parquet format using Amazon Athena CTAS.

Both formats produce the same analytical results. The actual data scanned and execution time measurements can be used to evaluate the performance difference.

Parquet with partitioning can reduce unnecessary data scanning for queries that filter on partition columns.

---

# FINAL PROJECT OUTCOME

By completing this project, we successfully demonstrated:

1. Amazon S3 bucket creation and dataset storage.
2. Athena database creation.
3. External table creation on CSV data.
4. SQL-based monthly sales analysis.
5. Data conversion from CSV to Parquet using CTAS.
6. Year and month partitioning.
7. Verification of CSV and Parquet tables.
8. Performance comparison using Athena execution statistics.

## Screenshots

Add your actual AWS screenshots in the screenshots folder.

| Screenshot | Description |
|---|---|
| 01-s3-bucket.png | S3 bucket creation |
| 02-csv-upload.png | CSV dataset uploaded |
| 03-athena-database.png | sales_db database |
| 04-csv-table.png | sales_csv table |
| 05-csv-query-results.png | January CSV query output |
| 06-csv-statistics.png | CSV execution statistics |
| 07-parquet-folder.png | Parquet partition folders |
| 08-parquet-table.png | sales_parquet table |
| 09-parquet-query-results.png | January Parquet query output |
| 10-parquet-statistics.png | Parquet execution statistics |
| 11-query-history.png | Athena Query History |

---

## Author

**Arushi Gupta**

GitHub: https://github.com/ArushiGUPTA2512

Repository: MSE-2-AWS-2
