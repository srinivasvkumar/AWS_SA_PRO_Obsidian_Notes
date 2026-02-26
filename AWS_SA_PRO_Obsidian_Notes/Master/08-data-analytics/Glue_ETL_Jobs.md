## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, [[glue]] is a fully managed extract, transform, and load (ETL) service that makes it easy to prepare and load data for analytics. It consists of a centralized metadata repository, an ETL engine that can execute Python or Scala code, and a catalog of crawlers and jobs.

### [[RDS_Instance_Types|Internals]]

The metadata repository, known as the [[glue]] [[glue|Data Catalog]], stores information about databases and tables. This information is used by [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] such as [[AWS_SA_PRO_Obsidian_Notes/Master/Analytics/Athena|Athena]], [[redshift]] Spectrum, and [[emr]]. The ETL engine, called [[glue]] ETL, allows you to define custom logic for processing your data.

### [[RDS_Instance_Types|Global Scale Considerations]]

[[glue]] is designed to operate at global scale, allowing you to create and manage resources in multiple regions. To achieve this, [[glue]] uses a combination of regional and global resources. For example, the [[glue|Data Catalog]] is a global resource, while [[glue]] ETL jobs are regional.

When working with [[glue]] in a multi-region environment, there are a few things to keep in mind. First, the [[glue|Data Catalog]] is not automatically replicated across regions. If you need to access the same metadata in multiple regions, you must manually copy the metadata to each region. Second, [[glue]] ETL jobs are specific to a single region. If you need to process data in multiple regions, you must create separate [[glue]] ETL jobs for each region.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

There are several alternatives to [[glue]], including AWS [[dms]], [[kinesis|Kinesis Data Firehose]], and [[lambda]]. Here is a comparison of [[glue]] with these services:

| Service | Use Case |
| --- | --- |
| [[glue]] | Structured ETL workloads with large datasets |
| [[dms]] | Homogeneous database migrations |
| [[kinesis|Kinesis Data Firehose]] | Real-time streaming data ingestion |
| [[lambda]] | Serverless compute for event-driven applications |

Some common anti-patterns when using [[glue]] include:

* Using [[glue]] for real-time data processing: [[glue]] is designed for batch processing, not real-time data processing.
* Using [[glue]] for small datasets: [[glue]] has a minimum billable amount of 5 minutes per job run, which may be too expensive for small datasets.

## [[RDS_Instance_Types|3. Security & Governance]]

[[glue]] supports fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], cross-account access, and organization SCPs.

Here is an example [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that allows a user to create and manage [[glue]] jobs in a specific account:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "glue:CreateJob",
                "glue:UpdateJob",
                "glue:DeleteJob"
            ],
            "Resource": "arn:aws:glue:us-west-2:123456789012:job/*"
        }
    ]
}
```

To enable cross-account access, you can use the `glue:AddDevEndpointToPolicy` and `glue:AddJobsToPolicy` actions.

Organization SCPs can be used to enforce organizational [[policies]], such as requiring all [[glue]] jobs to be encrypted with a specific key.

## [[RDS_Instance_Types|4. Performance & Reliability]]

[[glue]] has throttling limits that vary based on the type of operation. For example, the maximum number of concurrent job runs is 25, while the maximum number of crawler runs is 5.

If you encounter throttling [[api-gateway|errors]], you can implement exponential backoff strategies to retry failed requests.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] can be achieved by creating [[glue]] resources in multiple regions. To ensure consistent metadata across regions, you can use the [[glue]] [[glue|Data Catalog]] API to synchronize metadata between regions.

## [[RDS_Instance_Types|5. Cost Optimization]]

[[glue]] offers granular cost controls through the use of job bookmarks and error handling. Job bookmarks allow you to continue processing data from where the previous job run stopped, reducing the amount of data that needs to be processed. Error handling allows you to specify how [[glue]] should handle failures, such as retrying failed tasks or skipping over bad records.

Here is an example cost calculation for a [[glue|Glue job]] that runs once per day and processes 1 TB of data:

| Resource | Quantity | Cost |
| --- | --- | --- |
| [[glue]] [[glue|Data Catalog]] | 1 | $0.15 / month |
| [[glue]] ETL Job | 1 / day | $0.50 / job |
| [[glue]] ETL Worker Hours | 100 / day | $0.000016 / hour |

Total cost = $0.15 + ($0.50 \* 30 days) + ($0.000016 \* 100 hours \* 30 days) = $1.65 / month

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario 1: Multi-account Strategy

You are designing a multi-account strategy for a large enterprise that wants to use [[glue]] for ETL processing. Each department within the enterprise will have their own AWS account.

How would you design this solution to meet the following requirements:

* Centralized management of [[glue]] resources
* Isolation of [[glue]] resources between departments
* Secure sharing of data between departments

#### Solution

To meet these requirements, you could use a combination of [[organizations|AWS Organizations]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles.

First, create a central AWS account that will be used for managing [[glue]] resources. Then, create separate AWS accounts for each department within the enterprise.

Next, use [[organizations|AWS Organizations]] to create a hierarchy of accounts. Within the hierarchy, apply a service control policy ([[SCP]]) that denies access to [[glue]] resources outside of the department's account.

To share data between departments, use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles. In the source department's account, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role that grants permission to the destination department's [[glue|Glue job]] to access the required data. In the destination department's account, grant permission to the [[glue|Glue job]] to assume the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role created in the source department's account.

Finally, use the [[glue]] [[glue|Data Catalog]] to store metadata for all departments. This ensures centralized management of [[glue]] resources.

#### Justification

This solution meets the requirement for centralized management of [[glue]] resources by using a central AWS account for managing [[glue]] resources. It also meets the isolation of [[glue]] resources between departments by using separate AWS accounts for each department. Additionally, it meets the secure sharing of data between departments by using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles to grant permission to [[glue]] jobs to access data in other departments' accounts.

### Scenario 2: Quotas and Integration

You are designing a solution that involves using [[glue]] for ETL processing. Your solution requires processing 5 TB of data per day, but you have encountered throttling issues.

How would you optimize your solution to meet the following requirements:

* Process 5 TB of data per day
* Minimize throttling issues
* Implement exponential backoff strategies

#### Solution

To optimize your solution for processing 5 TB of data per day, you could use a combination of [[glue|Glue job]] bookmarks and error handling.

First, use [[glue|Glue job]] bookmarks to continue processing data from where the previous job run stopped. This reduces the amount of data that needs to be processed, minimizing the risk of throttling issues.

Next, use error handling to specify how [[glue]] should handle failures. For example, you could configure the [[glue|Glue job]] to retry failed tasks or skip over bad records.

To implement exponential backoff strategies, use the AWS SDK to catch throttling exceptions and retry the request with increasing delays.

#### Justification

This solution meets the requirement for processing 5 TB of data per day by using [[glue|Glue job]] bookmarks to reduce the amount of data that needs to be processed. It also meets the requirement for minimizing throttling issues by using error handling to specify how [[glue]] should handle failures. Finally, it meets the requirement for implementing exponential backoff strategies by using the AWS SDK to catch throttling exceptions and retry the request with increasing delays.