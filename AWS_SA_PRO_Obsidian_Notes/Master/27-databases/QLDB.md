**Advanced Architecture of QLDB**

QLDB is a fully managed ledger database provided by AWS, that provides transparent, immutable, and cryptographically verifiable log of all changes to data. The internal architecture consists of a centralized journal that stores an immutable sequence of data changes, along with indexing and query services for retrieving and manipulating data. The underlying storage is based on Amazon's purpose-built key-value store, which allows QLDB to achieve high performance and low latency.

Global scale can be achieved in QLDB through the use of [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]], allowing you to create a single endpoint that maps to multiple regional QLDB databases. This enables low-latency access from clients around the world.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| [[dynamodb]] | For applications requiring high write throughput and flexible schema design. |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] | For relational workloads with ACID compliance. |
| Elasticsearch | For full-text search, analytics, and machine learning tasks. |
| QLDB | For applications requiring an immutable, cryptographically verifiable log of transactions. |

Anti-pattern: Using QLDB as a primary data store when high write throughput or dynamic schema design is required.

**[[appsync|Security]] & Governance**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] should enforce the principle of least privilege. Here's an example JSON policy snippet that grants read-only access to a specific QLDB table:
```json
{
    "Effect": "Allow",
    "Action": [
        "quorum-service-api:BatchGetItem",
        "quorum-service-api:DescribeTable"
    ],
    "Resource": [
        "*"
    ],
    "Condition": {
        "StringEquals": {
            "quorum-service-api:TableName": [
                "your-ledger-name/your-table-name"
            ]
        }
    }
}
```
Cross-account access can be granted using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[sts]] AssumeRole. Organizational Service Control [[policies]] (SCPs) can be used to restrict the creation of QLDB resources within an [[AWS Organization]].

**Performance & Reliability**

Throttling limits in QLDB depend on the size of the journal partition. To manage throttling, implement exponential backoff strategies using SDK retries. High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] can be achieved by deploying QLDB in multiple regions, leveraging [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] for low-latency access.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be implemented using AWS [[billing|Cost Explorer]], setting up [[billing]] alarms, and allocating costs to individual teams or projects. To estimate QLDB costs, consider factors such as provisioned throughput capacity, number of queries, and amount of data stored.

**Professional Exam Scenarios**

Scenario 1: A company needs to build a new application that requires an immutable, cryptographically verifiable log of transactions. They have chosen QLDB as their primary data store but need to ensure that only authorized users can perform read and write operations. How would you secure the QLDB resource?

Correct Answer: Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] with least privilege principles, granting read-only access to specific tables and enforcing write permissions using resource-level permissions. Optionally, enable encryption at rest and in transit.

Incorrect Answer: Create a public QLDB resource without any access restrictions.

Scenario 2: A global organization wants to deploy QLDB across multiple regions while minimizing network latency. What solution would you recommend to meet these requirements?

Correct Answer: Use [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] to create a single endpoint that maps to multiple regional QLDB databases. This will allow clients to connect to the closest QLDB region, reducing network latency.

Incorrect Answer: Deploy QLDB in a single region and rely on [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections for inter-region communication.

### Architecture Diagram
![[QLDB.png]]
