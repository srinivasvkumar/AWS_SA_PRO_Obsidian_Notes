**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] is a physical device provided by AWS that allows customers to transfer large amounts of data (up to 100 PB) from their location to an AWS datacenter. The [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] consists of a semi-trailer truck containing up to 45 petabytes of data storage. Upon request, AWS will deliver the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] to your location, where you can load it with your data using high-speed networking equipment. Once loaded, AWS transports the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] back to one of its datacenters and uploads the data to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]].

The [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] uses multiple layers of [[appsync|security]] measures to protect your data during transport. These measures include GPS tracking, alarm monitoring, and 24/7 video surveillance. Data stored on the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] is encrypted using the AWS Key Management Service ([[kms]]).

[[RDS_Instance_Types|Global scale considerations]]:

* [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] supports multi-region replication, allowing customers to migrate data to another region after the initial migration.
* For large-scale migrations spanning multiple regions or countries, customers can coordinate simultaneous migrations using multiple Snowmobiles.

Under the hood mechanics:

* Data transfer from the customer site to the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] occurs via high-speed networking equipment, such as 40 Gbps and 100 Gbps Ethernet connections.
* After AWS receives the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]], data is automatically transferred to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] using the AWS [[snow|Snowball Edge]] client.
* Customers can monitor the status of the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] migration in real time using the AWS Management Console.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service              | Use Case                                                                   |
| -------------------- | ------------------------------------------------------------------------- |
| [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]]          | Large-scale (exabyte-scale) migrations requiring high throughput         |
| [[Snowball]]            | Medium-scale (petabyte-scale) migrations                                   |
| [[Srinivas_Notes/S3|S3]] Transfer Acceleration | Small-scale (sub-petabyte) migrations over long distances               |
| Direct Connect       | High-bandwidth connectivity between on-premises networks and AWS services |

Anti-patterns:

* Using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] for small-scale migrations (<100 TB) due to the high costs associated with the service.
* Storing sensitive data on the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] without proper encryption and access control mechanisms.

**[[RDS_Instance_Types|3. Security & Governance]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "snowmobile:DescribeJob",
                "snowmobile:ListJobs"
            ],
            "Resource": "*"
        }
    ]
}
```

Cross-account access:

* To allow cross-account access to the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] service, add the following policy statement to the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user or role:

```json
{
    "Effect": "Allow",
    "Action": [
        "snowmobile:CreateJob",
        "snowmobile:UpdateJob",
        "snowmobile:CancelJob"
    ],
    "Resource": "*",
    "Condition": {
        "StringEquals": {
            "aws:SourceVault": "<source_account_ID>"
        }
    }
}
```

Organization SCPs:

* Implement Service Control [[policies]] (SCPs) at the organization level to restrict usage of [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] to specific accounts or regions.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits:

* Each [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] job has a maximum limit of 120 days to complete the migration.
* If the data transfer rate exceeds the capacity of the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]], AWS may throttle the data transfer to ensure stability and prevent data corruption.

Exponential backoff strategies:

* When encountering throttling [[api-gateway|errors]], implement exponential backoff strategies to retry the operation at increasing intervals.

HA/DR patterns:

* To ensure high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], consider using multi-region replication with [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]].

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls:

* Monitor [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] usage costs using AWS [[billing|Cost Explorer]] and set up [[billing]] alerts to trigger when usage exceeds a specified threshold.
* Consider using the AWS Pricing Calculator to estimate the cost of a [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] migration before initiating the job.

Calculation example:

* A single [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] provides 100 petabytes of data storage, with a cost of $0.005 per GB per month.
* Total cost = 100 PB \* (1024 GB/PB) \* ($0.005/GB) = $51,200 per month

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
A media company needs to migrate 50 petabytes of raw video content from its data center to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. They require high throughput and secure data transfer. Which solution would best meet these requirements?

Correct answer: [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]]
Justification: [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] provides high throughput data transfer capabilities and encryption for secure data transfer. It also meets the size requirement of 50 petabytes.

Incorrect answer: [[Snowball]]
Justification: While [[Snowball]] is suitable for smaller-scale migrations, it cannot handle the required data volume of 50 petabytes.

Scenario 2:
A financial institution wants to migrate 1 petabyte of sensitive data from its on-premises environment to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. They need to ensure that only authorized users have access to the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] service.

Correct answer: Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and Service Control [[policies]] (SCPs) to restrict usage of the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] service to specific accounts and regions.

Justification: By implementing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and SCPs, the financial institution can ensure that only authorized users have access to the [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/Snowmobile|Snowmobile]] service, preventing unauthorized data migration.