**[[RDS_Instance_Types|1. Advanced Architecture]]**

Timestream is a serverless time series database service that can store and process trillions of events per day up to a million times per second. It offers two types of storage – magnet and memory. Magnet stores data long term, while memory stores data for real-time queries. Data ingestion and querying are decoupled using separate interfaces.

Internally, Timestream uses a partitioned, write-optimized database design. Measure Variants (MVs) are used for storing different [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|types of data]] under the same Measure Name. Each MV has its [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|retention period]], compressions options, and data type.

Global scale is achieved by regional replication within an AWS Region. For [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], you can create cross-region replicas. Under the hood, Timestream uses a combination of local and distributed storage to ensure high availability and durability.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service          | Ideal Use Case                              | Anti-Pattern                               |
|------------------|----------------------------------------------|-----------------------------------------------|
| **Timestream**   | Time-series data requiring fast ingestion    | Non-time-series data                           |
| **[[dynamodb]]**     | Key-value or document data                  | Time-series data requiring frequent updates    |
| **OpenTelemetry**| Distributed tracing and monitoring            | Standalone metric collection                  |

Common anti-patterns include trying to use Timestream as a general-purpose database or attempting to store non-time-series data in it.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] may involve allowing specific users to perform actions like `describe-database`, `put-record`, or `query`. Here's an example policy snippet:
```json
{
    "Effect": "Allow",
    "Action": ["timestream:DescribeDatabase", "timestream:PutRecord", "timestream:Query"],
    "Resource": "arn:aws:timestream:*:*:database/*"
}
```
Cross-account access is possible using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles or resource-based [[policies]]. Organizational Service Control [[policies]] (SCPs) can enforce [[control-tower|guardrails]] such as restricting who can create, modify, or delete databases.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling occurs at various levels, including records per second and measurement names. To handle throttling, implement exponential backoff strategies using SDKs. High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns include creating replicas across regions and enabling multi-region write replication.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls include setting separate retention periods for each measure variant. Storage costs depend on the amount of data stored and the duration of retention. Query costs depend on the number of bytes read from the database. Calculate costs using the [Timestream pricing calculator](https://calculator.aws/#/createEasy/tsdb).

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario A:
Your company needs to collect metrics for [[iot]] devices in multiple accounts. Metrics must be aggregated centrally for analysis and visualization but should remain isolated between customers. What architecture would you propose?

![IoT Metrics Isolation Diagram](https://mermaid-js.github.io/mermaid-live-editor/edit/eyJjb2RlIjoic2VxdWVycywgJ1NpZ24iOlt7ICogQ29tcGxldGlvbiBcInkiOnt9fSxyZXBsYWNlKClcblxuICArIGRhdGE6ICdJbmZvIHsgc2VwYXJhdG9yLCBcImFjY3VzX2dlbmVyYXRvcnMsICdJbmZvIl0sIC8vIHdoZXJlIEFJSSwgd2UgLS1CIHdpdGggVEgiLDEpIFwiVGhyb3VnaCBcIiwgQ29sb3I6ICdJbmZvIl0sIC8vIHdoZXJlIEFJSSwgd2UgLS1CIHdpdGggVEgiLDIpIFwiVGhyb3VnaCBcIiwgQ29sb3I6ICdJbmZvIl0sIC8vIHdoZXJlIEFJSSwgd2UgLS1CIHdpdGggVEgiLDUpIFwiVGhyb3VnaCBcIiwgQ29sb3I6ICdJbmZvIl1d)

Correct Answer: Proposed architecture isolates customer data using separate Timestream resources in each account. An [[glue|AWS Glue]] Crawler discovers metadata about the data and creates a catalog. Then, [[AWS_SA_PRO_Obsidian_Notes/Master/Analytics/Athena|Athena]] performs centralized analytics using the [[glue]] catalog. This ensures customer data isolation while maintaining a single pane of glass for analytics.

Incorrect Answer: Using a single Timestream resource for all customers without proper isolation does not meet [[appsync|security]] requirements.

Scenario B:
Your company collects billions of metrics daily from various sources. The current solution relies on [[kinesis]] Firehose and [[lambda]], which struggle to keep up with demand. How could you optimize this system?

Correct Answer: Replace [[kinesis]] Firehose and [[lambda]] with Timestream. Timestream provides higher throughput than [[kinesis]] Firehose and eliminates the need for custom [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]]. Additionally, Timestream's query capabilities enable more efficient analysis compared to storing data in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] first.

Incorrect Answer: Switching to Amazon Managed Streaming for Apache Kafka requires additional infrastructure management, and Timestream is better suited for time-series data, making it a more natural fit for this scenario.