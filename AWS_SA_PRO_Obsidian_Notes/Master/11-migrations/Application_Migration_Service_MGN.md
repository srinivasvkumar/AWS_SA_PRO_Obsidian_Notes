**Advanced Architecture**

At its core, Application Migration Service (MGN) is a highly automated, configurable service that uses replication, validation, and testing to migrate applications to AWS. The internal architecture consists of three main components:

1. **Replication Engine**: This component captures system snapshots (at the storage array or host level) and transfers them to AWS [[Storage Gateway]] using Storage Optimized (SO) instances in Amazon [[ec2]]. It supports multiple change types like block-level, file-level, and network-level changes.

2. **Validation Engine**: Validates the integrity of replicated data by comparing it against production data at various consistency levels. For example, eventual consistency allows you to minimize migration time, while strong consistency ensures that all updates made to the source environment are reflected in the target environment.

3. **Test Framework**: Enables testing of migrated applications within an isolated [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] before actual cutover. Tests can be executed multiple times until there's confidence in the readiness of the application for production traffic.

[[RDS_Instance_Types|Global scale considerations]] involve asynchronous data transfer between geographically dispersed locations using Global Accelerator. By leveraging edge locations closer to the source, MGN reduces latency during migrations.

**Comparison & Anti-Patterns**

| Service               | Use Case                                                              |
| --------------------- | -------------------------------------------------------------------- |
| Database Migration    | Homogeneous database migrations (e.g., Oracle to [[aurora]])            |
| Server Migration     | Heterogeneous server migrations (e.g., Windows to Linux)             |
| VMware Cloud Migration | VMware workloads from on-premises to AWS Outposts/VMC                |
| Storage Migration    | NFS shares, NetApp ONTAP, Windows File Server, etc.                  |
| Batch Migration       | Large scale data sets requiring parallel processing and automation |

Anti-patterns include live migrations without proper validation and testing, overly complex cutover procedures, and mixing different types of migrations under one project.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] may involve allowing specific users to initiate/stop migrations, granting cross-account access, and enforcing centralized control through [[organizations|AWS Organizations]] Service Control [[policies]] (SCPs):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "mgn:*"
      ],
      "Resource": [
          "*"
      ],
      "Condition": {
          "StringNotEqualsIgnoreCase": {
              "aws:PrincipalArn": [
                  "arn:aws:iam::ACCOUNT_ID:role/MyMigrationRole"
              ]
          }
      }
    }
  ]
}
```

**Performance & Reliability**

Throttling limits depend on the instance type used for the Replication Engine. To avoid exceeding these limits, implement exponential backoff strategies when retries are necessary. High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns often involve setting up secondary replication jobs to ensure failover capabilities.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be implemented using AWS [[billing|Cost Explorer]] and tagging resources appropriately. Calculating costs involves understanding the pricing model, which includes charges for SO instances, data transferred, and additional services utilized during migration.

**Professional Exam Scenarios**

Scenario 1: A company wants to migrate a large number of servers running various operating systems. They need to maintain tight control over who can initiate migrations. How would you set up MGN to meet their requirements?

Correct answer: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role named MyMigrationRole, attach a policy granting required permissions to MGN actions, then add a Deny statement in the [[SCP]] at the organization level to restrict migration initiation to only that role.

Incorrect answer: Allow any user with appropriate permissions to initiate migrations, as this does not provide the desired tight control over migration initiation.

Scenario 2: A business has migrated several databases using MGN but now faces throttling issues due to high data transfer rates. What steps should they take to mitigate these issues?

Correct answer: Implement exponential backoff strategies when retrying migration tasks and consider using larger instance types for the Replication Engine to increase throttling limits.

Incorrect answer: Ignore the issue, assuming it will resolve itself or downgrade to a lower tier of service, both of which could lead to prolonged migration times and potential data loss.