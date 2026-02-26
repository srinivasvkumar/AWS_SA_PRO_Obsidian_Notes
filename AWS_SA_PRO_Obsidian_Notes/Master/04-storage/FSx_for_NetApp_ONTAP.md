**[[fsx|FSx for NetApp ONTAP]] Advanced Architecture**

[[fsx|FSx for NetApp ONTAP]] is a fully managed service that provides high-performance shared storage using NetApp ONTAP data management software. It supports NFS, SMB, and iSCSI protocols, allowing you to deploy various workloads such as databases, analytics, and file shares.

Internally, [[fsx|FSx for NetApp ONTAP]] uses NetApp ONTAP 9-based storage servers and creates a volume for each file system. The service automatically replicates data across multiple Availability Zones (AZs) within an AWS Region, providing data durability and resilience. Each [[fsx]] for NetApp ONTAzure (AZ) hosts one or more storage servers, and clients connect to the service through Network File System (NFS), Server Message Block (SMB), or Internet Small Computer System Interface (iSCSI) protocols.

Global scale can be achieved by setting up a single AWS account per Region, creating a separate [[fsx|FSx for NetApp ONTAP]] instance in each target Region, and configuring cross-Region replication between them. This approach allows applications running in different Regions to access their respective file systems while maintaining low-latency connections.

**Comparison & Anti-Patterns**

| Service               | Use Case                                                              |
| --------------------- | -------------------------------------------------------------------- |
| [[fsx|FSx for Lustre]]        | High-performance computing, machine learning, and electronic design automation (EDA) workloads |
| [[fsx]] for Windows File Server | Windows-based applications requiring SMB access                     |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]]                   | NFSv4.1-based applications, particularly those with a microservices architecture |
| Amazon [[Srinivas_Notes/S3|S3]]             | Object storage, backup and archive, and static website hosting       |

Anti-patterns include using [[fsx|FSx for NetApp ONTAP]] when performance requirements do not justify its higher costs compared to other services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] or [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. Also, avoid using [[fsx|FSx for NetApp ONTAP]] for workloads that require object storage features like server-side encryption or versioning.

**[[appsync|Security]] & Governance**

To implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for [[fsx|FSx for NetApp ONTAP]], create JSON [[policies]] similar to these examples:

* Allow read-only access to all [[fsx|FSx for NetApp ONTAP]] resources in an account:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "fsx:DescribeFileSystems"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
* Grant full control of a specific [[fsx|FSx for NetApp ONTAP]] resource:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "fsx:CreateFileSystem",
                "fsx:DeleteFileSystem",
                "fsx:DescribeFileSystems",
                "fsx:UpdateFileSystem"
            ],
            "Resource": [
                "arn:aws:fsx:us-west-2:123456789012:netapp-ontap/filesystem/fs-01234567890abcdef0"
            ]
        }
    ]
}
```
Cross-account access requires granting permissions to both accounts involved. For example, if Account A wants to allow Account B to perform actions on its [[fsx|FSx for NetApp ONTAP]] resources, add the following policy statement to Account A's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role or user:
```json
{
    "Effect": "Allow",
    "Action": [
        "fsx:Describe*"
    ],
    "Resource": "*",
    "Condition": {
        "StringEquals": {
            "aws:SourceVpc": "arn:aws:ec2:us-west-2:111122223333:vpc/vpc-0abcd1234efgh5678"
        }
    }
}
```
In Account B, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that grants users permission to perform actions on Account A's resources:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "fsx:Describe*"
            ],
            "Resource": [
                "arn:aws:fsx:us-west-2::netapp-ontap/filesystem/*",
                "arn:aws:fsx:us-west-2::storage-virtualmachine/*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:PrincipalOrgID": "o-123456789012"
                },
                "ArnLike": {
                    "aws:SourceAccount": "arn:aws:fsx:us-west-2:111122223333"
                }
            }
        }
    ]
}
```
For organization-wide Service Control [[policies]] (SCPs), ensure that they allow necessary actions and resources related to [[fsx|FSx for NetApp ONTAP]].

**Performance & Reliability**

Throttling limits depend on the number of [[fsx|FSx for NetApp ONTAP]] file systems and API calls made within a given period. To mitigate throttling issues, implement exponential backoff strategies using techniques like linear or binary increases in wait time between retries.

High availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] ([[dr]]) patterns involve configuring [[fsx|FSx for NetApp ONTAP]] in multiple AZs and Regions, respectively. In addition, enabling automatic backups and creating custom recovery points provide additional [[dr]] capabilities.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include monitoring usage metrics like provisioned capacity, number of file systems, and active client connections. Calculate costs based on the pricing model, which includes capacity, IOPS, and snapshot storage. By understanding how these factors impact costs, [[organizations]] can optimize spending.

**Professional Exam Scenarios**

Scenario 1:
> An organization needs to store large amounts of data for a big data processing application while ensuring high availability and low latency. They want to use [[fsx|FSx for NetApp ONTAP]] but also want to minimize costs. What should they do?

Correct answer: Implement a multi-account strategy with one centralized account holding [[fsx|FSx for NetApp ONTAP]] resources and separate accounts for compute resources. Enable sharing between accounts using [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering, and configure cross-region replication for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes. Monitor usage metrics and adjust storage capacities accordingly to optimize costs.

Incorrect answer: Create a single [[fsx|FSx for NetApp ONTAP]] instance with maximum storage capacity and enable cross-region replication. While this scenario meets the high availability requirement, it does not minimize costs.

Scenario 2:
> An organization has strict compliance requirements for storing sensitive data. They need to implement a secure solution using [[fsx|FSx for NetApp ONTAP]]. What steps should they follow?

Correct answer: Implement least privilege [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], leveraging predefined AWS managed [[policies]] where possible. Set up a dedicated account for [[fsx|FSx for NetApp ONTAP]] resources and enable network traffic flow [[vpc-flow-logs|logging]]. Configure cross-account access only when necessary and monitor audit logs for unusual activities.

Incorrect answer: Share [[fsx|FSx for NetApp ONTAP]] resources between multiple accounts without implementing proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and access restrictions. This approach may expose sensitive data to unauthorized users and violate compliance requirements.