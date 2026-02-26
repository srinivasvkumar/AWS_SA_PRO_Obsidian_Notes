**Advanced Architecture**

[[snow|Snowball Edge]] is a physical device provided by AWS that allows customers to move large amounts of data into and out of AWS services. It is designed to handle edge computing workloads, including local data processing, machine learning, and data transfer. [[snow|Snowball Edge]] supports several modes of operation:

* **Local mode**: Data is stored locally on the device and can be processed using [[lambda|AWS Lambda]] functions or Docker containers.
* **Edge-to-AWS mode**: Data is automatically transferred to AWS services such as Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]], or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon SageMaker]] when the device is connected to the internet.
* **Cluster mode**: Multiple devices can be clustered together to form a local cluster for high availability and parallel processing.

[[snow|Snowball Edge]] devices have up to 100 TB of storage capacity and support several interfaces, including Ethernet, USB, and serial connections. They also include an Intel Xeon D processor and up to 32 GB of memory.

When designing a solution using [[snow|Snowball Edge]], it's important to consider global scale requirements. [[snow|Snowball Edge]] devices can be shipped globally, but they must be returned to the country in which they were originally ordered. To achieve global distribution, multiple [[snow|Snowball Edge]] devices can be used in conjunction with AWS Region Replication to replicate data across regions.

At the core of [[snow|Snowball Edge]] is its ability to perform data transfer securely and efficiently. This involves several components:

* **Client-side encryption**: [[snow|Snowball Edge]] uses client-side encryption to protect data during transit. The device generates a unique encryption key for each job and encrypts the data using AES-256.
* **Data flow**: Data flows from the customer's network through the [[snow|Snowball Edge]] device and then onto AWS services. The device acts as a gateway, performing any necessary data transformations before sending the data to AWS.
* **Data integrity**: [[snow|Snowball Edge]] uses a variety of mechanisms to ensure data integrity during transit. This includes checksums, redundant arrays of independent disks (RAID) protection, and erasure coding.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| [[snow|Snowball Edge]] | Large-scale data migration or edge computing workloads |
| Amazon [[Srinivas_Notes/S3|S3]] Transfer Acceleration | Small-scale data migration over long distances |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/11-migrations/datasync|AWS DataSync]] | Data migration between on-premises environments and AWS |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]] | Establishing private connectivity between on-premises environments and AWS |

Anti-pattern: Using [[snow|Snowball Edge]] for small-scale data transfers. [[snow|Snowball Edge]] is designed for large-scale data migrations and edge computing workloads. For smaller data transfers, other services like Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration or [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] may be more appropriate.

**[[appsync|Security]] & Governance**

To implement [[appsync|security]] and governance [[iam|best practices]] with [[snow|Snowball Edge]], you should follow these guidelines:

* Implement least privilege access control using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]. Here's an example JSON policy that grants access to a specific [[snow|Snowball Edge]] device:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "snowballedge:DescribeJob",
                "snowballedge:GetJob",
                "snowballedge:ListJobs",
                "snowballedge:CancelJob"
            ],
            "Resource": "arn:aws:snowballedge::*:job/*",
            "Condition": {
                "StringEquals": {
                    "snowballedge:job-device-serial-number": [
                        "<Your Device Serial Number>"
                    ]
                }
            }
        }
    ]
}
```
* Implement cross-account access using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]]. Here's an example JSON policy that grants access to another account:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::<Account ID>:root"
            },
            "Action": [
                "snowballedge:DescribeJob",
                "snowballedge:GetJob",
                "snowballedge:ListJobs",
                "snowballedge:CancelJob"
            ],
            "Resource": "arn:aws:snowballedge::*:job/*",
            "Condition": {
                "StringEquals": {
                    "snowballedge:job-device-serial-number": [
                        "<Your Device Serial Number>"
                    ]
                }
            }
        }
    ]
}
```
* Implement organization SCPs to enforce [[control-tower|guardrails]] around resource usage. For example, you could create an [[SCP]] that denies all [[snow|Snowball Edge]] actions except those explicitly allowed.

**Performance & Reliability**

[[snow|Snowball Edge]] has several throttling limits that you should be aware of:

* Up to five jobs can be created per [[snow|Snowball Edge]] device at a time.
* Each job can transfer up to 80 TB of data.
* Each [[snow|Snowball Edge]] device can transfer up to 100 TB of data.

To avoid exceeding these limits, you should implement exponential backoff strategies when creating jobs. This involves waiting a progressively longer amount of time between retries if a request fails due to throttling.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns for [[snow|Snowball Edge]] involve using multiple devices in different locations. For example, you could use one device in each region to store data locally and replicate data to other regions using AWS Region Replication. If a device fails, the remaining devices can continue to operate without interruption.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls for [[snow|Snowball Edge]] involve understanding usage patterns and optimizing based on those patterns. For example, you could schedule data transfers during off-peak hours to reduce costs associated with data transfer rates. Additionally, you could use [[snow|Snowball Edge]] devices only when needed and return them to AWS when not in use to minimize storage costs.

Here's an example calculation of [[snow|Snowball Edge]] costs:

* Cost of [[snow|Snowball Edge]] device: $250 per month
* Cost of data transfer: $0.09 per GB
* Total cost: $250 + (<Amount of data transferred> \* $0.09)

**Professional Exam Scenarios**

Scenario 1: You need to migrate 10 PB of data from your on-premises environment to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. Which solution would provide the most cost-effective and efficient method for migrating the data?

Correct answer: [[snow|Snowball Edge]] provides the most cost-effective and efficient method for migrating large amounts of data. It offers faster data transfer rates than Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration and reduces the need for expensive network infrastructure.

Incorrect answer: [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] is not the most cost-effective option for migrating 10 PB of data since it charges based on the amount of data transferred.

Scenario 2: You need to implement a hybrid cloud solution that allows you to process data both on-premises and in AWS. Which solution would enable you to achieve this goal while minimizing data transfer costs?

Correct answer: [[snow|Snowball Edge]] provides a hybrid cloud solution that enables you to process data both on-premises and in AWS. By using [[snow|Snowball Edge]] in local mode, you can process data locally and then transfer it to AWS when needed. This reduces data transfer costs compared to continuously transferring data over the network.

Incorrect answer: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]] does not provide a hybrid cloud solution since it establishes private connectivity between on-premises environments and AWS. It does not allow for local data processing.