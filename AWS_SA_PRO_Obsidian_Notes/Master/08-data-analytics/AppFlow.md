**Advanced Architecture**

AppFlow is a fully managed integration service that enables you to transfer data between AWS services and SaaS applications without writing custom code. It supports various connectors such as Salesforce, Zendesk, ServiceNow, etc. AppFlow uses flow templates that define the source, target, transformation, and scheduling of data movements.

Internally, AppFlow uses [[lambda|AWS Lambda]] functions to perform data transformations and enrichments. The [[lambda]] function is created when you configure a flow using a connector that requires one, such as Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] or [[redshift|Amazon Redshift]]. AppFlow manages these [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] automatically, so you don't need to worry about their maintenance.

For [[RDS_Instance_Types|global scale considerations]], AppFlow supports cross-Region data transfers. However, it does not support cross-account data transfers directly. To achieve this, you can create a [[lambda]] function in the destination account to transfer data from the source account.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| AppFlow | Real-time data synchronization between SaaS apps and AWS services. Data transformations and enrichments. |
| [[glue|AWS Glue]] | Batch ETL jobs for data warehousing and big data processing. |
| AWS DataBridge | Asynchronous event-driven data movement between different services. |

Anti-pattern: Using AppFlow for batch data processing instead of real-time data synchronization.

**[[appsync|Security]] & Governance**

To enable cross-account access, you can use AWS [[sts]] AssumedRoleProvider to assume an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the destination account. Here's an example JSON policy for enabling cross-account access:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::source_account_ID:root"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpce": "vpce-xxxxxxxx"
                }
            }
        }
    ]
}
```
Additionally, you can enforce organization-wide [[appsync|security]] [[policies]] using Service Control [[policies]] (SCPs) in [[organizations|AWS Organizations]]. For example, you can restrict the creation of new flows based on specific tags or deny access to certain connectors.

**Performance & Reliability**

AppFlow has throttling limits for creating and updating flows. The throttling limit is 5 requests per second. If you exceed this limit, AppFlow returns a `ThrottlingException`. To handle this scenario, you can implement exponential backoff strategies using the AWS SDK.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], AppFlow supports multiple regions and availability zones. You can configure your flows to use a primary region for data processing and a secondary region for backup and failover.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

AppFlow charges based on the number of successful flow executions and data processed. To optimize costs, you can do the following:

* Schedule flows during off-peak hours.
* Use Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] lifecycle [[policies]] to archive old data.
* Implement tagging strategies to monitor and control costs at scale.

Here's an example calculation for AppFlow costs:

Suppose you have a flow that processes 1 GB of data per day and runs once a day. The total monthly cost would be:

(Number of days in a month \* Data processed per day) / 1024 \* Price per GB = Total cost

Assuming 30 days in a month and a price of $0.04 per GB, the total monthly cost would be:

(30 \* 1 GB) / 1024 \* $0.04 = $0.12

**Professional Exam Scenarios**

Scenario 1: A company wants to transfer data from Salesforce to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and apply some data transformations before storing the data in [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. Which service should they use?

Correct answer: AppFlow

Justification: AppFlow supports Salesforce as a connector and allows applying data transformations using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]].

Incorrect answer: AWS DataBridge

Justification: DataBridge does not support Salesforce as a connector and is designed for asynchronous event-driven data movement.

Scenario 2: A customer needs to transfer data between two AWS accounts and wants to ensure that only authorized users can create flows. What is the recommended approach?

Correct answer: Create a cross-account [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role using [[sts]] AssumedRoleProvider and attach a policy that allows creating flows.

Justification: This approach allows the customer to grant access to specific users or roles while maintaining a centralized [[appsync|security]] model.

Incorrect answer: Configure a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] endpoint and allow access to the required resources.

Justification: [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]] do not provide cross-account access control and are used for private connectivity between services.