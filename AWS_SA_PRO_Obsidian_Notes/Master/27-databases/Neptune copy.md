## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/07-databases/neptune|Amazon Neptune]] is a scalable graph database service that supports the Property Graph and SPARQL W3C standard query language for RDF. It offers high performance, automatic sharding, and point-in-time recovery capabilities. [[Neptune]] replicates up to six read replicas across three Availability Zones (AZs) in a single region. For global scale, you can set up a regional "multi-master" configuration with two or more [[Neptune]] clusters in different regions. Data writes can be distributed among these clusters using [[dynamodb]] Streams and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]].

Internally, [[Neptune]] uses a distributed storage system with a dual-writable architecture. The storage layer maintains five copies of data, ensuring durability and availability. Workers called "gremlins" perform CRUD operations and maintain cluster consistency. Gremlin workers communicate via a messaging queue, which allows them to process queries concurrently without locking conflicts.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service          | Ideal Use Case                         | Anti-Pattern                |
| ---------------- | ------------------------------------- | ---------------------------- |
| [[Neptune]]          | High-performance graph workloads      | Document-oriented data       |
| [[dynamodb]]         | Key-value store, NoSQL                | Complex transactional logic   |
| [[redshift]]         | Data warehousing, OLAP                  | Real-time analytics           |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]]              | Traditional relational databases      | Non-relational data            |
| [[DocumentDB]]       | Document-oriented data                | Structured hierarchical data  |
| Keyspace (Cassandra)| Time-series, [[iot]], and other distributed workloads | Relational data model     |

Common anti-patterns include storing document-oriented data in [[Neptune]] instead of [[dynamodb]] or [[DocumentDB]], as well as using it for complex transactional logic rather than [[dynamodb]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]].

## [[RDS_Instance_Types|3. Security & Governance]]

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "neptune:DescribeDBInstances",
              "neptune:DescribeDBClusters"
            ],
            "Resource": "*"
        }
    ]
}
```

### Cross-Account Access

To enable cross-account access, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account, attach necessary permissions, and add a trust relationship policy allowing the destination account's users to assume the role. Then, grant the role's permissions to the destination user or group.

### Organization Service Control [[policies]] (SCPs)

Use SCPs at the organization level to restrict [[Neptune]] instance creation based on specific resource types, tags, or [[cloudformation|conditions]]. This prevents unauthorized [[Neptune]] instances from being created within member accounts.

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits depend on the instance class and number of read replicas. To manage throttling, implement exponential backoff strategies when handling HTTP status codes like 429 (Too Many Requests).

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], use [[Neptune]]'s point-in-time recovery feature to restore your database to any point within the last 5 weeks. Alternatively, manually create automated backups and DB snapshots.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls involve monitoring usage metrics such as number of [[Neptune]] instances, instance classes, and duration. Calculate costs using the AWS Pricing Calculator or the [[Neptune]] Console. Implement tagging [[policies]] to track costs by project, team, or environment.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario A

An e-commerce company wants to migrate their existing relational database to [[Neptune]] while maintaining compatibility with their application's current ORM framework. Which approach would best meet their requirements?

*Correct Answer*: Create a [[Neptune]] database with a compatible Property Graph schema and write a [[lambda]] function to convert ORM-generated SQL statements into appropriate [[Neptune]] queries.

*Incorrect Answers*:
  * Using [[dynamodb]] or [[DocumentDB]] because they don't offer graph query capabilities.
  * Modifying the ORM framework directly since it may introduce additional complexity and potential bugs.

### Scenario B

A social media platform needs to support real-time trend analysis and notifications across multiple geographic regions. They have identified [[Neptune]] as the primary database for managing user relationships. What strategy should they adopt to ensure low latency and reliability?

*Correct Answer*: Set up a [[Neptune]] regional multi-master configuration with two or more clusters in different regions. Utilize [[dynamodb]] Streams and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] to distribute data writes among the [[Neptune]] clusters.

*Incorrect Answers*:
  * Using a single [[Neptune]] cluster with read replicas since it doesn't provide true geographical distribution.
  * Employing manual data replication between clusters due to potential inconsistencies and increased operational overhead.