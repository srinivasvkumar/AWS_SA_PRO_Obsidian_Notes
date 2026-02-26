**Advanced [[redshift]] Architecture**

[[redshift]] is a petabyte-scale data warehouse service that leverages a massively parallel processing (MPP) architecture. It uses columnar storage, advanced compression, and zone maps to increase query speed and reduce I/O.

**[[RDS_Instance_Types|Global Scale Considerations]]**

[[redshift]] does not directly support cross-region replication or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]]. However, you can create a custom solution using Data Pipeline, [[lambda]], and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. This involves exporting data from [[redshift]] to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], copying data between [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets in different regions, and then importing data into the target [[redshift]] cluster.

**Under the Hood**

[[redshift]] has these key components:

- **Nodes**: Each [[redshift]] cluster contains one or more nodes of two types: leader node and compute node. The leader node runs queries and coordinates data transfer between nodes. Compute nodes store data and perform computations.
- **Columnar Storage**: [[redshift]] stores data as columns rather than rows, which reduces the amount of IO required for queries.
- **Automated Table Optimization**: [[redshift]] automatically vacuums and analyzes tables to maintain performance.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| [[redshift]] | Large-scale data warehousing |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] | Operational databases |
| [[dynamodb]] | Key-value and document databases |

**Anti-pattern**: Using [[redshift]] for transactional workloads. [[redshift]] is optimized for analytics, not transactions.

**[[appsync|Security]] & Governance**

**Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "redshift:DescribeClusters",
                "redshift:GetClusterCredentials"
            ],
            "Resource": "*"
        }
    ]
}
```

**Cross-Account Access**

Use an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role to grant [[redshift]] read/write access to another account's [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket.

**Organization SCPs**

Use Organizational SCPs to enforce [[control-tower|guardrails]] like restricting [[redshift]] cluster creation to specific VPCs.

**Performance & Reliability**

**Throttling Limits**

[[redshift]] has several throttling limits, including concurrent queries, concurrent connections, and concurrent restore operations. To manage throttling, use exponential backoff strategies.

**HA/DR Patterns**

Create a secondary cluster in a different AZ for high availability. For [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], implement a custom solution using Data Pipeline, [[lambda]], and [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] as mentioned earlier.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include setting up alerts based on usage metrics, resizing clusters, pausing clusters during non-business hours, and using per-node pricing for predictable costs.

**Professional Exam Scenario: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

A company has a 16TB [[redshift]] cluster running continuously. They want to minimize costs without impacting performance. How should they proceed?

- Option 1: Enable per-node pricing
- Option 2: Implement query queueing
- Option 3: Resize the cluster to a smaller instance type

Correct Answer: Option 3. Resizing the cluster provides predictable costs while maintaining performance. Per-node pricing may increase costs, and query queueing could cause delays.

**Professional Exam Scenario: [[appsync|Security]]**

A company wants to allow its data engineering team to analyze data stored in a separate AWS account. What's the best way to provide secure access to [[redshift]]?

- Option 1: Create a role in each account allowing [[redshift]]:DescribeClusters and [[redshift]]:GetClusterCredentials actions
- Option 2: Set up cross-account [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket access
- Option 3: Use Amazon Resource Groups to share resources

Correct Answer: Option 1. Creating a role in each account allows secure access by specifying least privilege permissions. Option 2 is unrelated to [[redshift]], and Option 3 is not related to resource sharing.