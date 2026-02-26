### Advanced Architecture

File Gateway is a protocol-level, NFS- and SMB-based, software appliance that serves as a file server in your on-premises environment while providing cloud storage capabilities via Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. The gateway runs within a virtual machine on VMware ESXi or Microsoft Hyper-V.

At the core of File Gateway is the Storage Controller component responsible for maintaining cache and metadata about files stored on Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. The Storage Controller handles read and write requests from clients, optimizing performance by [[api-gateway|caching]] frequently accessed data on local disks. For better durability, it stores metadata on Amazon [[dynamodb]].

When operating at global scale, you can deploy File Gateway in multiple regions and link them together using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] or [[Storage Gateway]]'s Multi-Region capability. This setup enables centralized management while retaining low-latency access to data across different geographical locations.

### Comparison & Anti-Patterns

| Service        | Ideal Use Case                                                                   |
|----------------|-----------------------------------------------------------------------------|
| File Gateway  | NFS/SMB workloads requiring high throughput, low latency, and [[Srinivas_Notes/S3|S3]] object storage.    |
| [[fsx]] for Windows | High-performance, fully managed native Windows file system integration.         |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]]            | Regional, highly available, POSIX compliant shared file storage for Linux workloads. |

Anti-pattern: Using File Gateway for sharing files between multiple offices without considering WAN performance and potential throttling. Instead, consider using [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] or [[transfer-family|AWS Transfer Family]] to transfer files securely over the internet.

### [[appsync|Security]] & Governance

For cross-account access, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account allowing the target account's ARN as a trusted entity. Then, attach a policy granting necessary permissions to the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket. In the target account, create a corresponding [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user with programmatic access.

Example [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::examplebucket/*"
      ]
    }
  ]
}
```
To enforce [[appsync|security]] and governance across accounts, implement Organization Service Control [[policies]] (SCPs) restricting actions and resources.

### Performance & Reliability

Throttling limits depend on the type of storage used. File Gateway supports two types: Gateway-cached volumes and Gateway-stored volumes. Each has specific throttling limits related to API operations, file operations, and network bandwidth. To manage these limits, employ exponential backoff strategies when exceeded.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns include deploying multiple File Gateways in separate Availability Zones, utilizing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and failover configurations, and storing data across multiple AWS Regions using Multi-Region capabilities.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Implement granular cost controls by monitoring usage metrics such as API calls, file operation rates, and network bytes sent/received. Set up alerts based on thresholds and [[Budgets]]. Additionally, enable lifecycle [[policies]] to transition older infrequently accessed files to lower-cost [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] like [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Intelligent-Tiering or [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]].

Calculation example:

Suppose you store 1 PB of data on File Gateway with Gateway-cached volume configuration. Assume you have a 1 Gbps connection and pay $0.08 per GB-month for [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Standard storage. Your monthly costs would be approximately $16,368 (excluding data transfer fees).

### Professional Exam Scenarios

Scenario 1: A company needs to provide fast, low-latency access to their 50 TB dataset spread across three locations in US East (N. Virginia), US West (Oregon), and Europe (Ireland). They want to maintain one set of data and control access centrally. How would you implement a solution using File Gateway?

Correct Answer: Implement File Gateway in each location, creating a single [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket spanning all three regions using [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]'s bucket-level replication feature. Configure cross-region replication to ensure all objects created in any region propagate to the [[AWS_SA_PRO_Obsidian_Notes/Master/16-other/kendra|others]]. Finally, apply [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[policies]] to regulate access to the [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket.

Incorrect Answer: Deploy File Gateway in a single location and use [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering to connect other sites. This approach increases latency due to non-local data access and does not address the need for centralized control.

Scenario 2: A customer requires a scalable, durable, and highly available solution for storing large volumes of financial data with strict compliance requirements. They prefer using NFSv4. Which AWS service best fits their needs?

Correct Answer: File Gateway provides NFSv4 support, and since it's built upon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], offers scalability, durability, and high availability. By implementing appropriate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and leveraging [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] features like bucket versioning and Object Lock, customers can meet stringent compliance requirements.

Incorrect Answer: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] does not offer NFSv4 support, making it unsuitable for this scenario. While [[fsx]] for Windows provides NFSv4 compatibility, its focus on Windows file systems makes it less ideal than File Gateway for managing large volumes of financial data.