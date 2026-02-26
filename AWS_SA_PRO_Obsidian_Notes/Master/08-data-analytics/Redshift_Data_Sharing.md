**Advanced Architecture: [[redshift]] Data Sharing**

[[redshift]] Data Sharing allows a direct, fast, and secure data sharing between [[redshift]] clusters within an organization without requiring complex ETL operations or creating additional copies of the data. It operates at the table level, enabling fine-grained control over shared datasets.

Internally, [[redshift]] Data Sharing creates a local database object called "data share" in the source cluster, which maintains metadata about shared tables and their corresponding objects in target clusters. The actual data transfer occurs via network traffic encryption using TLS protocol.

[[RDS_Instance_Types|Global Scale Considerations]]:
- Data Shares can be created across different regions by setting up a [[redshift]] Spectrum endpoint in the target region and configuring cross-region table placement.
- For multi-account setups, you can create Data Shares in one account and consume them in other accounts within the same region.

Under the Hood Mechanics:
- Data Sharing utilizes PostgreSQL's feature called "Foreign Table," which enables remote query execution against underlying tables in another cluster.
- Querying shared tables is performed as if they were local tables with some [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] like no support for UPDATE, DELETE, or INSERT statements, and lack of VACUUM command.

**Comparison & Anti-Patterns**

| Service | Comparison Criteria | Suitability |
| --- | --- | --- |
| [[redshift]] Data Sharing | Real-time query performance | High |
|  | Secure data sharing | High |
|  | Fine-grained control over shared data | Medium |
| Federated Database Connection | Centralized [[api-gateway|authentication]] management | Low |
|  | Simplified application development | Medium |
|  | Limited query performance | Low |
| Data Copying | Complete data isolation | High |
|  | Separate storage and compute resources | High |
|  | Increased complexity and latency | Low |

Anti-patterns include:
- Using [[redshift]] Data Sharing when real-time query performance isn't necessary.
- Applying Data Sharing for large-scale data integration projects with multiple data sources and destinations.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "redshift:DescribeClusters",
                "redshift:CreateDataShare",
                "redshift:ModifyDataShare",
                "redshift:DeleteDataShare",
                "redshift:AuthorizeDataShare",
                "redshift:RevokeDataShare"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpc": "${vpcId}"
                }
            }
        }
    ]
}
```
Cross-account Access:
- Create a Data Share in the source account and then grant permissions to specific users or groups in the target account via `ALTER DATASHARE GRANT USAGE ON SHARE share_name TO user_name;` statement.

Organization SCPs:
- Implement Organizational Units (OUs) based on business units or departments and apply SCPs to restrict or allow certain actions related to [[redshift]] Data Sharing.

**Performance & Reliability**

Throttling Limits:
- Maximum concurrent queries per cluster: 50 (shared tables count towards this limit)
- Maximum concurrent connections per cluster: 200

Exponential Backoff Strategies:
- Implement custom retry mechanisms in applications that interact with [[redshift]] Data Sharing to handle transient [[api-gateway|errors]].
- Include random jitter to avoid hitting throttling limits due to simultaneous retries from multiple components.

HA/DR Patterns:
- Ensure read replicas are enabled for high availability and scalability.
- Set up backup schedules and automated failover processes for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular Cost Controls:
- Monitor and track costs associated with each Data Share.
- Implement tagging [[policies]] for individual clusters and Data Shares to optimize resource allocation and usage.

Calculation Examples:
- Cluster A has 100GB of shared data with Cluster B, while Cluster C consumes 80% of the total shared data from both clusters.
  - Calculate the monthly cost for Cluster C: (Cluster C's total data consumption / Total shared data) \* Monthly [[redshift]] cost

**Professional Exam Scenarios**

Scenario 1:
Suppose you have two [[redshift]] clusters, A and B, in separate accounts. You want to enable data sharing between these clusters. Which steps should you follow?

Correct Answer:
1. Configure a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering connection between the two accounts.
2. Create a Data Share in Cluster A.
3. Grant permission to users or groups in Cluster B to access the Data Share.
4. Consume the Data Share in Cluster B.

Incorrect Answers:
a. Create a snapshot of Cluster A and copy it to Cluster B.
b. Set up a federated database connection between Cluster A and Cluster B.

Reasoning:
The correct answer involves configuring a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering connection and following the proper Data Share creation and permission granting process. The incorrect answers do not address the need for real-time query performance or efficient data sharing.

Scenario 2:
You work for a company with multiple business units, each having its own AWS account. The marketing department wants to analyze sales data stored in a [[redshift]] cluster located in the finance department's AWS account. What approach would you recommend?

Correct Answer:
1. Enable [[redshift]] Data Sharing between the marketing and finance departments' clusters.
2. Create a Data Share in the finance department's cluster.
3. Grant permission to users or groups in the marketing department's cluster to access the Data Share.
4. Authorize users in the marketing department's cluster to query the shared data.

Incorrect Answers:
a. Set up a cross-account role delegation between the two accounts.
b. Create a new [[redshift]] cluster in the marketing department's AWS account and copy the sales data from the finance department's [[redshift]] cluster.

Reasoning:
The correct answer ensures real-time query performance and secure data sharing between the two clusters. The incorrect answers either introduce unnecessary complexity or do not provide optimal query performance.