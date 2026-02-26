Advanced Architecture
---------------------

At its core, [[billing]] Conductor is an advanced cost management tool that allows [[organizations]] to allocate and manage costs across multiple AWS accounts. It operates by ingesting [[billing]] data from the AWS Cost and Usage Report, which contains detailed usage and cost information. The [[billing]] Conductor then processes this data using custom rules and allocation strategies, generating detailed reports and insights.

Internally, [[billing]] Conductor uses a combination of [[lambda|AWS Lambda]] functions, Amazon [[dynamodb|DynamoDB tables]], and Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets to process and store [[billing]] data. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] are used to perform tasks such as data validation, transformation, and aggregation, while [[dynamodb|DynamoDB tables]] store metadata about the [[billing]] data and allocation rules. [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets are used to store the raw Cost and Usage Report data, as well as the generated reports.

[[RDS_Instance_Types|Global scale considerations]] are handled by distributing the processing of [[billing]] data across multiple regions. This is achieved by setting up a separate set of [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], [[dynamodb|DynamoDB tables]], and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets in each region. The [[billing]] Conductor then uses [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] to transfer the raw Cost and Usage Report data from the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket in the region where it was generated to the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket in the region where it will be processed.

Comparison & Anti-Patterns
---------------------------

| Service | Use Case |
| --- | --- |
| [[billing]] Conductor | Multi-account cost allocation and management |
| AWS [[billing|Cost Explorer]] | Ad-hoc cost analysis and reporting |
| [[billing|AWS Budgets]] | Creating and tracking [[Budgets]] for AWS costs |
| [[billing|AWS Cost and Usage Reports]] | Detailed usage and cost information |

Common anti-patterns when using [[billing]] Conductor include:

* Using [[billing]] Conductor as a replacement for AWS [[billing|Cost Explorer]] or [[billing|AWS Budgets]], rather than as a complement.
* Allocating costs based on arbitrary factors rather than actual usage.
* Failing to properly secure access to the [[billing]] Conductor and the underlying data.

[[appsync|Security]] & Governance
----------------------

Access to [[billing]] Conductor can be controlled using AWS Identity and Access Management ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]]) [[policies]]. For example, you can create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role with permissions to access the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], [[dynamodb|DynamoDB tables]], and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets used by the [[billing]] Conductor, and then assign this role to users or groups who need to access the [[billing]] Conductor.

Cross-account access can be granted using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]]. For example, you can create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the AWS account containing the [[billing]] Conductor with permissions to access the resources in another account, and then grant permission to assume this role to users or groups in the other account.

Organization Service Control [[policies]] (SCPs) can also be used to enforce granular control over the actions that can be performed in the [[billing]] Conductor. For example, you can use an [[SCP]] to prevent users from creating new [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] or [[dynamodb|DynamoDB tables]] in the [[billing]] Conductor account.

Performance & Reliability
--------------------------

[[billing]] Conductor has built-in throttling limits to prevent it from overwhelming the underlying AWS services. These limits can be monitored and adjusted using [[cloudwatch]] metrics. If the throttling limits are exceeded, the [[billing]] Conductor will automatically retry the failed operation after a short delay. If the operation continues to fail, the [[billing]] Conductor will progressively increase the delay using an exponential backoff strategy.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns for [[billing]] Conductor can be implemented using AWS Multi-Region/Multi-Az deployment patterns. For example, you can deploy the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], [[dynamodb|DynamoDB tables]], and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets used by the [[billing]] Conductor in multiple regions, and use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] to route traffic to the nearest region. In case of a regional outage, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] will automatically redirect traffic to the next nearest region.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
-----------------

[[billing]] Conductor provides granular cost controls through its custom allocation rules. These rules allow you to allocate costs based on a variety of factors, including usage, tags, and custom attributes. By carefully defining these rules, you can gain detailed insight into your AWS costs and optimize them accordingly.

For example, suppose you have two departments, A and B, and they both use Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. However, department A stores mostly static content, while department B stores frequently accessed dynamic content. To optimize costs, you could define a custom attribute "content\_type" and tag each [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket with this attribute. Then, you could create a [[billing]] Conductor rule that allocates 90% of the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] costs to department A and 10% to department B, based on the "content\_type" attribute.

Professional Exam Scenarios
----------------------------

### Scenario 1

You are working as a solutions architect for a large organization with multiple AWS accounts. The organization wants to implement a centralized cost management solution using [[billing]] Conductor. The solution must support the following requirements:

* The solution should be able to allocate costs based on custom attributes.
* The solution should be able to generate daily and monthly reports.
* The solution should be highly available and scalable.

Which of the following architectures would meet these requirements?

![Architecture Diagram](mermaid\:here)

Correct Answer: The architecture depicted in the diagram meets all the requirements. It uses [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] to transfer the raw Cost and Usage Report data to an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket in the same region as the [[billing]] Conductor. The [[billing]] Conductor then processes the data using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] and stores the results in a [[dynamodb]] table. Finally, the results are stored in an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket and made available for download through Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration.

Incorrect Answers:

* Using a single [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket for storing both the raw Cost and Usage Report data and the generated reports would not meet the requirement for high availability and scalability.
* Using a single [[lambda]] function to process the Cost and Usage Report data would not allow for parallel processing and thus would not meet the requirement for high availability and scalability.

### Scenario 2

You are working as a solutions architect for a medium-sized organization with a single AWS account. The organization wants to implement a cost management solution using [[billing]] Conductor. The solution must support the following requirements:

* The solution should be able to allocate costs based on tags.
* The solution should be able to track [[Budgets]] and send alerts when budget thresholds are exceeded.

Which of the following architectures would meet these requirements?

![Architecture Diagram](mermaid\:here)

Correct Answer: The architecture depicted in the diagram does not meet all the requirements. While it uses [[billing]] Conductor to allocate costs based on tags, it does not provide a way to track [[Budgets]] and send alerts when budget thresholds are exceeded.

Incorrect Answers:

* Using [[billing|AWS Budgets]] in addition to [[billing]] Conductor would meet the requirement for tracking [[Budgets]] and sending alerts.
* Using AWS [[billing|Cost Explorer]] in addition to [[billing]] Conductor would provide additional cost analysis and reporting capabilities.