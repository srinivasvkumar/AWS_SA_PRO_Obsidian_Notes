## 1. Advanced [[glue]] [[glue|Data Catalog]] Architecture

The [[glue]] [[glue|Data Catalog]] is a central repository for metadata about data stored in the AWS ecosystem. It enables various services such as [[emr]], [[redshift]], [[AWS_SA_PRO_Obsidian_Notes/Master/Analytics/Athena|Athena]], and [[kinesis|Kinesis Data Firehose]] to discover and interact with data using a unified schema. The catalog stores table definitions, partitions, database connections, and other metadata.

Internally, the [[glue]] [[glue|Data Catalog]] uses Apache Hive Metastore under the hood. This allows compatibility with tools that support Hive including Spark, Hive, and Presto.

### [[RDS_Instance_Types|Global Scale Considerations]]

For multi-region or global deployments, it's crucial to understand that the [[glue]] [[glue|Data Catalog]] operates within a single region. To achieve global availability, you can create separate catalogs in each required region. However, manual synchronization of schemas across regions would be necessary.

To simplify this process, consider using infrastructure as code (IaC) tools like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] or Terraform to automate schema deployment. Alternatively, leverage custom scripts to export and import catalog definitions between regions.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service              | Use Case                                           |
| -------------------- | ------------------------------------------------- |
| [[glue]] [[glue|Data Catalog]]    | Centralized metadata management for AWS analytics |
| Hive Metastore       | On-premises Hadoop clusters                      |
| [[dynamodb]]             | Key-value store without SQL querying capabilities   |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]]                 | Transactional databases requiring ACID compliance   |

### Anti-patterns

* Using the [[glue]] [[glue|Data Catalog]] as a primary data store instead of an analytical data lake.
* Applying the [[glue]] [[glue|Data Catalog]] for transactional workloads better suited for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] or [[dynamodb]].

## [[RDS_Instance_Types|3. Security & Governance]]

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] should follow the principle of least privilege. For example, granting access only to specific resources and actions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "glue:GetDatabase*, glue:GetTable*"
      ],
      "Resource": [
          "arn:aws:glue:us-east-1:123456789012:database/*"
      ]
    }
  ]
}
```

Cross-account access can be enabled through a resource-based policy attached to the [[glue|Data Catalog]]:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::111122223333:root"
        ]
      },
      "Action": [
        "glue:GetDatabase*, glue:GetTable*"
      ],
      "Resource": [
          "arn:aws:glue:us-east-1:123456789012:database/*"
      ],
      "Condition": {
          "StringEquals": {
              "aws:SourceVpc": "vpc-abcdefgh"
          }
      }
    }
  ]
}
```

Organization Service Control [[policies]] (SCPs) provide centralized control over permissions at the organization level. For instance, restricting the creation of [[glue]] resources to specific accounts:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyCreateGlueResources",
      "Effect": "Deny",
      "Action": [
        "glue:CreateDatabase",
        "glue:CreateTable",
        ...
      ],
      "Resource": "*",
      "Condition": {
          "StringNotEqualsIgnoreCase": {
            "aws:Account": [
              "123456789012",
              "222233334444"
            ]
          }
      }
    }
  ]
}
```

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits depend on the type of operation. For example, creating a new database has a limit of 1 call per second. In case of throttling [[api-gateway|errors]], implement exponential backoff strategies to ensure reliability.

High availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] ([[dr]]) patterns involve deploying separate catalogs in different regions and replicating schema changes manually or automatically using custom scripts or IaC tools.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls include monitoring usage of specific tables and databases. Calculate costs based on the number of API calls made and the amount of metadata stored.

To optimize costs, consider deleting unused resources, enabling versioning for tables, and applying lifecycle management [[policies]] to remove old versions.

## 6. Professional Exam Scenario

Scenario 1: A company needs to manage metadata for their data lake across multiple regions. They want to maintain a single source of truth while ensuring high availability.

Correct Answer 1: Create separate [[glue]] [[glue|Data Catalog]] instances in each region and set up cross-region synchronization using infrastructure as code tools or custom scripts.

Incorrect Answer 1: Use a single [[glue]] [[glue|Data Catalog]] instance since it supports multi-region access.

Scenario 2: An organization wants to restrict [[glue]] [[glue|Data Catalog]] usage to specific teams and prevent accidental deletion of resources.

Correct Answer 2: Implement fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for [[glue]] [[glue|Data Catalog]] operations and apply Service Control [[policies]] (SCPs) at the organizational level.

Incorrect Answer 2: Allow unrestricted access to all teams and rely on manual cleanup procedures.