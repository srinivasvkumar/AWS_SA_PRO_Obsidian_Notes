---
aliases:
- "AWS Glue"
- "Data Catalog"
- "Glue Job"
---

# [[00_Master_Links_and_Intro|AWS Glue]]

- Is a serverless ETL (Extract, Transform, Load) product
- Similar to AWS Datapipeline (which can do ETL) but it uses servers ([[emr]] clusters)
- Glue is used to move data and transform data between source and destination
- Glue also crawls data sources and generates the [[00_Master_Links_and_Intro|AWS Glue]] Data catalog
- Supports a range of data collection as data sources: [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]], [[rds]], JDBC compatible data sources and [[dynamodb]]
- Supports data streams as data sources such as [[kinesis]] Data Streams and Apache Kafka
- Data targets supported: [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]], [[rds]], JDBC data sources

## Data Catalog

- Collection of persistent metadata about data sources in a region
- Provides one uniq data catalog in every region per account
- It helps avoid data silos: makes data not visible in an organization be visible a able to be browsed
- Data Catalog can be used by Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/07-databases/athena|athena]], [[redshift]] Spectrum, [[emr]] and [[AWS_SA_PRO_Obsidian_Notes/Master/08-data-analytics/others|AWS Lake Formation]]
- Data is discovered by configuring crawlers and givin it credentials to access data sources

## Glue Job

- Extract, Transform an Load jobs
- Jobs can do transformation by using scripts created by us
- Jobs are serverless, AWS maintains a pool of resources
- We are only billed by resources we consume