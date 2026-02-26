```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### [[Enterprise Database Architecture]] Deep-Dive

#### Under-the-Hood Mechanics:
- **Data Flow**: In an [[enterprise environment]], data typically flows through various components like [[application servers]], [[load balancers]], and the database itself. For relational databases such as [[Amazon RDS]] or NoSQL databases like [[dynamodb]], data is processed and stored in specific formats.
- **Consistency Models**:
  - **Strong Consistency**: Ensures that all transactions occur immediately (e.g., [[dynamodb]] strongly consistent reads).
  - **Eventual Consistency**: Guarantees consistency over time but not immediately (e.g., [[dynamodb]] eventually consistent reads, [[Amazon S3]]).
- **Underlying Infrastructure Logic**:
  - Multi-AZ deployment for high availability.
  - Automated backups and point-in-time recovery.
  - Integration with AWS services like [[AWS_SA_PRO_Obsidian_Notes/Master/VPC]], [[iam]], [[cloudwatch]].

#### Exhaustive Feature List
1. **Relational Databases (Amazon [[00_Master_Links_and_Intro|RDS]])**:
   - Support for multiple database engines: MySQL, PostgreSQL, Oracle, SQL Server, MariaDB.
   - Multi-AZ deployment and read replicas.
   - Automated backups and point-in-time recovery.
2. **NoSQL Databases ([[dynamodb]])**:
   - Key-Value Storage with adjustable throughput.
   - On-demand capacity and automatic scaling.
   - Global tables for multi-region deployments.
3. **Document Databases (Amazon [[DocumentDB]], [[Amazon Dynamo DB Streams]])**:
   - Support for JSON documents.
   - Integration with [[AWS Lambda]] for real-time processing.
4. **Time-Series Data Stores**:
   - [[Amazon Timestream]]: Optimized for time-series data.
5. **Database Migration Services**:
   - [[AWS DMS]] for migrating databases between different environments.

#### Step-by-Step Implementation
1. **Assess Requirements**: Define database needs, including type (relational vs. NoSQL), performance, and availability requirements.
2. **Select Database Service**:
   - For relational data: Use [[00_Master_Links_and_Intro|RDS]] with appropriate engine (MySQL, PostgreSQL).
   - For key-value or document data: Use [[dynamodb]].
3. **Design Schema & Tables**: Plan table structure, indexes, keys.
4. **Deploy on AWS**:
   - Create [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] and subnets.
   - Set up [[appsync|security]] groups and [[00_Master_Links_and_Intro|IAM]] roles.
5. **Configure High Availability**:
   - Enable Multi-AZ for [[00_Master_Links_and_Intro|RDS]].
   - Use read replicas or global tables in [[dynamodb]].
6. **Monitor & Optimize**:
   - Integrate with [[cloudwatch]] for monitoring.
   - Regularly [[nonportable_instructions|review]] performance metrics.

#### Technical Limits (2026)
- **[[00_Master_Links_and_Intro|RDS]]**: Soft limits like maximum number of instances per region; hard limits include instance types and storage sizes.
- **[[dynamodb]]**: Maximum read/write capacity units, table size limits.

#### CLI & API Grounding
- **AWS CLI Commands**:
  ```bash
  aws rds create-db-instance
  aws dynamodb create-table
  ```
- **API Flags**:
  - [[00_Master_Links_and_Intro|RDS]]: `--multi-az`, `--backup-retention-period`.
  - [[dynamodb]]: `ProvisionedThroughput`, `GlobalSecondaryIndexes`.

#### Pricing & TCO
- **Cost Drivers**: Instance types, storage size, IOPS (Input/Output Operations Per Second), data transfer.
- **Optimization Strategies**:
  - Use [[00_Master_Links_and_Intro|Reserved Instances]] for [[00_Master_Links_and_Intro|RDS]].
  - Enable on-demand capacity mode in [[dynamodb]].

#### Competitor Comparison
- **AWS vs. Azure SQL Database**: Azure has more integration with Microsoft tools, while AWS offers a broader service ecosystem.
- **AWS vs. Google Cloud Spanner**: Spanner is globally distributed and strongly consistent; [[dynamodb]] is eventually consistent but offers simpler setup.

#### Integration
- **[[cloudwatch]]**: For monitoring performance metrics.
- **[[00_Master_Links_and_Intro|IAM]]**: For access control and [[appsync|security]] [[policies]].
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: For network isolation and [[appsync|security]].

#### Use Cases
1. **Financial Services**: High availability and strong consistency with [[00_Master_Links_and_Intro|RDS]] Multi-AZ deployment.
2. **E-commerce**: Scalable NoSQL solutions like [[dynamodb]] for transaction processing.
3. **[[iot]] Applications**: Time-series data storage with [[Timestream]].

#### Diagrams
- Placeholder: Architectural diagram showing the flow of data from application servers to database services, highlighting [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] integration and [[appsync|security]] groups.

#### Exam Traps
1. > [!danger] Distractor:
   - [[dynamodb]] always requires manual scaling; on-demand mode automatically scales.
2. > [!danger] Distractor:
   - [[00_Master_Links_and_Intro|RDS]] read replicas do not support Multi-AZ configurations.

#### Decision Matrix: Features vs. Use Cases
| Feature/Use Case         | Financial Services            | E-commerce                     | [[iot]] Applications              |
|--------------------------|-------------------------------|--------------------------------|-------------------------------|
| Database Type (RDS/DynamoDB) | [[rds]] with Oracle or SQL Server | [[dynamodb]]                       | [[Timestream]]                    |
| Consistency Model        | Strong                        | Eventual                       | Strong                         |
| High Availability        | Multi-AZ                      | Global Tables                  | Global Databases               |

#### 2026 Updates
- Enhanced [[appsync|security]] features like new encryption options.
- Improved performance optimizations for both [[00_Master_Links_and_Intro|RDS]] and [[dynamodb]].

#### Exam Scenarios
1. > [!check] Implementation:
   - **Scenario**: Design a highly available database architecture for an e-commerce application.
     - **Answer**: Use [[dynamodb]] with global tables and enable on-demand capacity mode.
2. > [!check] Implementation:
   - **Scenario**: Migrate an Oracle database to AWS while ensuring high availability and strong consistency.
     - **Answer**: Deploy [[00_Master_Links_and_Intro|RDS]] for Oracle in Multi-AZ configuration with read replicas.

#### Interaction Patterns
- [[00_Master_Links_and_Intro|RDS]] interacts with [[AWS_SA_PRO_Obsidian_Notes/Master/VPC]] via [[appsync|security]] groups and [[00_Master_Links_and_Intro|IAM]] roles for access control.
- [[dynamodb]] integrates with [[lambda]] for real-time processing of updates through streams.

#### Strategic Alignment
Understanding the nuances between different database options ensures effective exam performance, focusing on high-weight topics like architecture design and service [[api-gateway|integrations]].