**[[RDS_Instance_Types|1. Advanced Architecture]]**

At its core, [[dms]] is a service that helps migrate databases by replicating data from a source database to a target database. The process involves setting up an endpoint for each database, creating a task to perform the migration, and monitoring the progress of the migration using [[cloudwatch]] metrics.

[[dms]] uses a variety of mechanisms under the hood to accomplish these migrations. For homogeneous migrations (e.g., Oracle to Oracle), [[dms]] uses change data capture (CDC) to keep track of changes made during the migration. This allows for minimal downtime migrations. For heterogeneous migrations (e.g., Oracle to MySQL), [[dms]] performs a full load of the data followed by ongoing replication using CDC or log-based incremental replication.

Global scale can be achieved in [[dms]] through the use of replication instances. These instances can be placed in different regions and used to replicate data across those regions. Additionally, [[dms]] supports multi-account strategies, allowing you to create tasks that span multiple accounts.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Case |
| --- | --- |
| [[dms]] | Homogeneous and heterogeneous migrations with low downtime requirements. |
| Database snapshots and AMI copies | Small, one-time migrations. |
| [[Storage Gateway]] | Hybrid cloud migrations. |

Anti-patterns include:

* Performing large, one-time migrations using [[dms]] instead of using database snapshots or AMI copies.
* Using [[dms]] for hybrid cloud migrations when [[Storage Gateway]] is more appropriate.

**[[RDS_Instance_Types|3. Security & Governance]]**

To implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for [[dms]], you can use JSON [[policies]] similar to the following:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dms:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Cross-account access can be granted by attaching an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role to the source and target databases and assuming that role within the [[dms]] account. Alternatively, you can use organization SCPs to restrict access to specific resources within the account.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

[[dms]] has throttling limits that vary depending on the type of operation being performed. To handle throttling [[api-gateway|errors]], it's recommended to use exponential backoff strategies.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns can be implemented using multi-region replication instances and read replicas. By placing replication instances in different regions and configuring them to replicate to each other, you can ensure that your data is always available even if one region goes down.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls can be implemented by using resource-level permissions and [[billing]] alarms. By limiting which resources can be accessed by [[dms]], you can prevent unexpected charges. [[billing]] alarms allow you to set thresholds for your expected monthly costs and receive notifications if those thresholds are exceeded.

Calculation example:

Assuming you have a single replication instance running continuously for a month at $0.30 per hour, the monthly cost would be:

$0.30/hour \* 24 hours/day \* 30 days/month = $21.60/month

Additionally, there may be additional costs for data transfer and storage depending on the amount of data being migrated.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
You are designing a solution to migrate a large Oracle database to a new Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] for Oracle instance. The database must be kept up-to-date during the migration process. Which [[dms]] features should you use?

Correct answer:
Use [[dms]] with CDC to keep the source and target databases in sync during the migration process.

Incorrect answer:
Do not use [[dms]] with CDC since the source and target databases are not compatible.

Scenario 2:
Your company has strict [[appsync|security]] [[policies]] and requires all [[dms]] activities to be logged and audited. How can you implement this using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]]?

Correct answer:
Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that allows [[dms]] actions and attach it to a group containing the necessary users. Enable [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]] [[vpc-flow-logs|logging]] for the [[dms]] API calls.

Incorrect answer:
Use [[cloudwatch|CloudWatch logs]] to monitor [[dms]] activity. This will only provide real-time monitoring and not meet the audit requirement.