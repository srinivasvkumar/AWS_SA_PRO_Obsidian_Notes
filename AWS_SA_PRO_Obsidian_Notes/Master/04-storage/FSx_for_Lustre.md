**[[fsx|FSx for Lustre]]: Advanced Architecture**

[[fsx|FSx for Lustre]] is a high-performance file system that provides a storage solution for applications requiring tight integration with compute instances. The service utilizes the Lustre file system, which allows parallel access to shared data, providing high aggregate throughput and low latencies.

Internally, [[fsx|FSx for Lustre]] uses the following components:

- *Metalock Service*: A highly available, centralized metadata management service.
- *Object Storage Service ([[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]])*: Stores file system metadata, including inodes and dentries.
- *Junction Points*: Enable connecting multiple [[fsx|FSx for Lustre]] file systems.
- *Storage Servers*: Provide the file system's raw storage capacity.

When designing global architectures, [[fsx|FSx for Lustre]] supports multiple regions within an AWS Region, but does not provide global data replication or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities. For these scenarios, implement cross-region read replicas using separate [[fsx|FSx for Lustre]] file systems and custom replication scripts.

**[[fsx|FSx for Lustre]]: Comparison & Anti-Patterns**

| Feature | [[fsx|FSx for Lustre]] | [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] | [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] |
| --- | --- | --- | --- |
| Max Throughput | ~10 GB/s/file system | ~380 MB/s/volume | Up to 10 GB/s |
| Data Persistence | Persistent across restarts | Not persistent | Persistent |
| File System Type | Lustre | NFSv3, iSCSI | NFSv4.1, SMB |
| Use Cases | HPC, machine learning, video processing | Virtual machines, boot volumes | Containers, web servers, content management |

Anti-patterns include using [[fsx|FSx for Lustre]] as a general-purpose file system, storing large amounts of infrequently accessed data, or utilizing it for small workloads that can be served by alternative services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]].

**[[fsx|FSx for Lustre]]: [[appsync|Security]] & Governance**

Implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] by using JSON snippets similar to the example below, allowing specific actions for [[fsx|FSx for Lustre]] resources:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "fsx:Describe*",
                "fsx:CreateFileSystem"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:PrincipalOrgID": "o-123456789012"
                }
            }
        }
    ]
}
```
For cross-account access, create a role in the source account with permissions to interact with [[fsx|FSx for Lustre]] resources, then assume the role from the destination account using [[sts]] AssumeRole API.

Use Organization SCPs to enforce restrictions on [[fsx|FSx for Lustre]] usage at scale. For instance, you can prevent users from creating [[fsx|FSx for Lustre]] resources outside their designated OUs or restrict specific API actions.

**[[fsx|FSx for Lustre]]: Performance & Reliability**

Throttling limits depend on the number of [[fsx|FSx for Lustre]] file systems per region and vary based on the AWS Support plan. To mitigate throttling issues, implement exponential backoff strategies when making repeated API calls.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], utilize multiple subnets distributed across different Availability Zones. This setup ensures continued operation even if one AZ experiences an outage. Additionally, enable automatic backup configurations to create periodic snapshots of your [[fsx|FSx for Lustre]] file systems.

**[[fsx|FSx for Lustre]]: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]****

Granular cost controls can be achieved by implementing tagging strategies for [[fsx|FSx for Lustre]] resources. Calculate costs using the pricing calculator, taking into account factors such as storage size, throughput mode, and deployment type.

To minimize costs, consider deleting unused file systems, enabling lifecycle management [[policies]] to automatically transition between throughput modes, and selecting the appropriate deployment type for your workload requirements.

**Professional Exam Scenario #1**

Assume you have been asked to design a multi-account architecture for a research organization working on genome sequencing. They require a scalable, high-performance file system to process large datasets. Due to compliance regulations, they need to store all metadata related to the file system in a separate account. How would you implement this?

Correct answer: Implement [[fsx|FSx for Lustre]] in the primary account and configure a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering connection between the two accounts. Store metadata in the secondary account using Object Lock, ensuring immutability and retention rules. Utilize cross-account access with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[sts]] AssumeRole to allow secure interactions between both accounts.

Incorrect answer: Attempt to store metadata directly within the [[fsx|FSx for Lustre]] file system. This approach is incorrect because [[fsx|FSx for Lustre]] stores only file system metadata, not user-defined metadata.

**Professional Exam Scenario #2**

Suppose you are tasked with optimizing the performance of a machine learning pipeline running on [[ec2]] instances connected to an [[fsx|FSx for Lustre]] file system. The current training time is too long, and there are occasional timeouts while reading data. What steps would you take to address these concerns?

Correct answer: Evaluate the number and configuration of [[ec2]] instances used for model training. Increase the number of [[ec2]] instances to improve aggregate throughput and reduce timeout occurrences. If needed, upgrade the [[fsx|FSx for Lustre]] file system to a higher throughput mode. Consider using larger storage servers to increase overall throughput.

Incorrect answer: Migrate the ML pipeline to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon SageMaker]]. While SageMaker offers managed machine learning capabilities, this change may not necessarily result in better performance than the existing [[fsx|FSx for Lustre]] setup. Moreover, switching to SageMaker could introduce additional complexity without resolving the underlying performance concerns.