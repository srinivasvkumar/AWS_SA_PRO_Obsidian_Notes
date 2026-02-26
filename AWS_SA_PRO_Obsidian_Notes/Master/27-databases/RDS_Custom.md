**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[rds|RDS Custom]] allows you to run your own isolated database instance software stack on Amazon Web Services (AWS) infrastructure. This is achieved by using [[rds|RDS Custom]] DB Engines, which are custom-built database engines that leverage the capabilities of the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] API, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] regional endpoints, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]]. The [[RDS_Instance_Types|internals]] involve an [[ec2]] instance running a specific database engine version while being managed by [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] under the hood.

At the global scale, [[rds|RDS Custom]] enables Multi-Region and Multi-Availability Zone (Multi-AZ) deployments, ensuring high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities. However, it does not support automatic failover or read replicas as standard [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] engines do. Instead, you need to implement custom solutions such as Multi-Master Replication or Global Tables in [[dynamodb]] to achieve similar functionality.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service          | Features                                                                                   | Use Cases                                           |
| ---------------- | ----------------------------------------------------------------------------------------- | -------------------------------------------------- |
| [[rds|RDS Custom]]       | Complete control over database engine, custom configurations                             | Proprietary databases, specialized workloads      |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] (Standard)   | Managed maintenance, automated backups, read replicas, Multi-AZ                         | Most common relational database use cases           |
| [[DocumentDB]]      | Fully managed MongoDB-compatible document store                                          | Modern application [[cloudformation|stacks]] requiring flexible schema |
| Keyspace ([[Collab_Notes_detailed/Databases/Neptune|Neptune]])| Fully managed graph database with Apache TinkerPop and SPARQL support                | Graph processing, AI/ML applications              |
| [[Timestream]]     | Time series data management and analysis                                                | [[iot]], operational applications, monitoring systems |

Anti-patterns include attempting to use [[rds|RDS Custom]] when standard [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] engines can meet the requirements, leading to increased complexity without tangible benefits. Another mistake is trying to apply [[rds|RDS Custom]] for horizontal scaling, which is better suited for distributed databases like [[dynamodb]] or Cassandra.

**[[RDS_Instance_Types|3. Security & Governance]]**

Access to [[rds|RDS Custom]] instances requires proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and roles. For cross-account access, use the `sts:AssumeRole` permission along with an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the destination account:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::123456789012:role/dba"
        }
    ]
}
```

To enforce [[appsync|security]] at scale, create Service Control [[policies]] (SCPs) within [[organizations|AWS Organizations]]:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": "rds-db:connect",
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:PrincipalArn": [
                        "arn:aws:iam::*:role/allowed_roles/*"
                    ]
                }
            }
        }
    ]
}
```

**[[RDS_Instance_Types|4. Performance & Reliability]]**

[[rds|RDS Custom]] has throttling limits based on the underlying [[ec2]] instance type used. To prevent [[api-gateway|errors]] due to exceeding these limits, implement exponential backoff strategies in your application code.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], utilize Multi-Region and Multi-AZ deployments with manual failover procedures. Implement automated backup strategies using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] snapshots and [[ec2]] instance backups.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls for [[rds|RDS Custom]] can be implemented through tagging resources and applying usage-based cost allocation reports. Additionally, monitor and optimize costs by tracking the number of active connections, storage utilization, and I/O operations.

**6. Professional Exam Scenario**

Scenario: Your company operates a proprietary real-time analytics system that processes billions of events daily. Due to its unique architecture, no existing managed database solution meets the performance and scalability requirements.

Question: Which approach would allow you to efficiently host this system on AWS?

Correct answer: Utilizing [[rds|RDS Custom]] with a highly-tuned, proprietary database engine tailored to the specific needs of the real-time analytics system.

Incorrect answer: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] (Standard) since it provides automated maintenance and backups. *Explanation:* While [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Standard offers many advantages, it may not provide sufficient performance and scalability for the given scenario.

Correct answer: Employing a Multi-Region deployment strategy with [[rds|RDS Custom]] to ensure high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]].

Incorrect answer: Ignoring the need for Multi-Region deployments. *Explanation:* Given the critical nature of the real-time analytics system, ignoring potential downtime or data loss isn't acceptable.

---

Scenario: A large organization wants to restrict access to their [[rds|RDS Custom]] instances only via approved roles.

Question: How should they enforce this policy across multiple accounts?

Correct answer: Create an Organization [[SCP]] that denies [[rds|RDS Custom]] connect permissions unless the request originates from an allowed role.

Incorrect answer: Implement a resource-based policy directly on [[rds|RDS Custom]] instances. *Explanation:* Resource-based [[policies]] don't span multiple accounts easily, making them unsuitable for enforcing [[policies]] across an organization.

Correct answer: Set up [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles with the required permissions and grant permissions to those roles using the Organization [[SCP]].

Incorrect answer: Attempt to enforce the policy using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Instance Profiles. *Explanation:* [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Instance Profiles aren't supported by [[rds|RDS Custom]], so this method won't work.

### AI Linked Diagram
![[Screenshot 2026-01-01 at 10.30.11 PM.png]]


### AI Linked Diagram
![[Screenshot 2026-01-01 at 10.32.14 PM.png]]
