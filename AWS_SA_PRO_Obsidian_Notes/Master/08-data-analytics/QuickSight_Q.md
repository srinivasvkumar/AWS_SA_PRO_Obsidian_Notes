Advanced Architecture
---------------------

At its core, [[quicksight]] Q is a Natural Language Query (NLQ) engine that allows users to ask questions of their data in plain English. It operates by analyzing the user's query, identifying the relevant entities and relationships, and translating it into an SQL-like query that can be executed against the underlying datasets.

[[quicksight]] Q uses a combination of machine learning algorithms and a knowledge graph to understand the context and semantics of the user's question. The knowledge graph is built using entities extracted from the user's data and metadata, as well as external sources such as Wikipedia. This enables [[quicksight]] Q to provide accurate results even when the user's query contains ambiguous or incomplete information.

When it comes to [[RDS_Instance_Types|global scale considerations]], [[quicksight]] Q leverages AWS's global infrastructure to distribute processing and storage across multiple regions. This allows [[quicksight]] Q to provide low-latency responses to user queries, regardless of their location. Additionally, [[quicksight]] Q supports cross-region replication, allowing users to create a single analysis that can be shared across multiple regions.

Comparison & Anti-Patterns
---------------------------

| Service | Use Case | Anti-Pattern |
| --- | --- | --- |
| [[quicksight]] Q | Ad-hoc analysis of structured data using natural language queries | Unstructured data analysis, large-scale batch processing, complex transformations |
| [[redshift]] Spectrum | Analysis of data stored in [[Srinivas_Notes/S3|S3]] using SQL | Data stored outside of [[Srinivas_Notes/S3|S3]], real-time analytics, interactive dashboards |
| [[Collab_Notes_detailed/Analytics/Athena|Athena]] | Ad-hoc analysis of data stored in [[Srinivas_Notes/S3|S3]] using SQL | Large-scale batch processing, complex transformations, real-time analytics |

Common design mistakes include attempting to use [[quicksight]] Q for unstructured data analysis, assuming it has the same capabilities as traditional BI tools like Tableau, and not properly configuring the knowledge graph.

[[appsync|Security]] & Governance
----------------------

[[quicksight]] Q supports fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], allowing administrators to control access to specific datasets, folders, and analyses. Here's an example JSON policy that grants access to a specific dataset:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "quicksight:DescribeDataSet",
                "quicksight:PassDataset",
                "quicksight:CreateAnalysis",
                "quicksight:CreateCalculatedField",
                "quicksight:CreateFilter",
                "quicksight:CreateTheme",
                "quicksight:CreateVisual",
                "quicksight:DescribeAnalysis",
                "quicksight:DescribeTemplate",
                "quicksight:DescribeTheme",
                "quicksight:DescribeVisual",
                "quicksight:UpdateAnalysis",
                "quicksight:UpdateTemplate"
            ],
            "Resource": [
                "arn:aws:quicksight:us-west-2:123456789012:dataset/my-dataset"
            ]
        }
    ]
}
```
Cross-account access can be achieved using cross-account delegation, which allows one account to act on behalf of another. This can be useful when sharing resources between different teams or [[organizations]].

Organization Service Control [[policies]] (SCPs) can also be used to enforce [[appsync|security]] [[policies]] at the organization level, ensuring that all accounts within the organization adhere to the same [[appsync|security]] standards. For example, an [[SCP]] could be used to deny access to certain datasets or restrict the actions that can be performed on a [[quicksight]] analysis.

Performance & Reliability
--------------------------

[[quicksight]] Q has several throttling limits in place to ensure fair usage and prevent abuse. These limits include a maximum of 20 concurrent queries per user, a maximum of 500 queries per minute per user, and a maximum of 100,000 rows returned per query. To handle these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], exponential backoff strategies can be implemented to retry failed requests with increasing delays.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] (HA/DR) patterns can be implemented using Amazon [[cloudwatch|CloudWatch alarms]] and [[lambda|AWS Lambda]] functions. For example, if the [[cloudwatch]] alarm detects that the response time for a [[quicksight]] Q query exceeds a specified threshold, a [[lambda]] function could be triggered to create a new analysis or switch to a standby analysis.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
------------------

Granular cost controls can be implemented using AWS [[billing|Cost Explorer]], which provides detailed breakdowns of costs associated with [[quicksight]] Q usage. Administrators can set up custom cost reports based on various parameters, including usage by user, dataset, or analysis.

Here's an example calculation for the cost of running [[quicksight]] Q:

Assume the following usage:

* 100 active users
* 10,000 queries per day
* 5GB of data transferred per month
* $0.10 per GB of data transfer
* $12 per user per month

The monthly cost would be calculated as follows:

* Data transfer cost: 5GB \* $0.10/GB = $0.50
* User cost: 100 users \* $12/user/month = $1200
* Total cost: $0.50 + $1200 = $1200.50

Professional Exam Scenarios
---------------------------

Scenario 1: A company has two separate AWS accounts, one for production workloads and one for development workloads. They want to allow developers in the development account to access datasets in the production account using [[quicksight]] Q. How can they accomplish this?

Correct answer: Using cross-account delegation, the production account can grant the development account permission to access the datasets. This can be done by creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the production account that allows the development account to perform specific [[quicksight]] Q actions, such as passing a dataset or creating an analysis.

Incorrect answer: Using cross-account data sharing, the production account can share the datasets with the development account. While this approach works for other services like [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], it does not work for [[quicksight]] Q.

Scenario 2: A company has a large dataset containing customer information and transactional data. They want to allow analysts to perform ad-hoc analysis using [[quicksight]] Q while ensuring that sensitive customer information is protected. What steps should they take to achieve this?

Correct answer: The company can implement column-level [[appsync|security]] using [[quicksight]] Q's row-level [[appsync|security]] feature. This allows them to specify which columns are visible to the analysts and which columns are hidden. The hidden columns can still be used in calculations and visualizations, but their values will not be displayed to the analysts.

Incorrect answer: The company can redact sensitive customer information before loading it into [[quicksight]] Q. While this approach may protect the data from being accessed through [[quicksight]] Q, it does not address the issue of ad-hoc analysis and may introduce [[api-gateway|errors]] or inconsistencies in the data.