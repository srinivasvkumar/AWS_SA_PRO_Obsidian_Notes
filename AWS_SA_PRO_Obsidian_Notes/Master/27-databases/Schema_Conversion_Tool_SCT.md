**[[AWS Schema Conversion Tool (SCT)]]: Advanced Architecture & [[RDS_Instance_Types|Internals]]**

At its core, SCT is a utility that converts source database schema into an equivalent target schema, supporting multiple RDBMS engines. The tool performs automated transformations while preserving original semantics and provides options for customizing conversion rules.

Internally, SCT leverages the [[glue|AWS Glue]] [[glue|Data Catalog]] to store metadata about imported schemas. It uses the AWS SDK to interact with other services like Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]], or [[sts]]. Global scale is achieved by distributing these components across various regions as needed.

Here's a simplified view of SCT under the hood:

```mermaid
graph LR