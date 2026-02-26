## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, [[lambda]] aliases and versions provide a mechanism for managing deployments, enforcing immutability, and enabling fine-grained access control in serverless applications. An *alias* is a pointer to a specific version of your [[lambda]] function, allowing you to direct traffic to different variants without changing the function's name. Understanding how these components work together will help you build scalable and robust serverless applications.

### [[RDS_Instance_Types|Global Scale Considerations]]

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] can be deployed across multiple regions for better latency and resiliency. To achieve this, you need to create separate versions and aliases in each region. However, maintaining consistent configurations and tracking changes become more challenging as the number of regions increases.

To address this issue, you can use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] [[cloudformation|StackSets]] or [[service-catalog|AWS Service Catalog]] to manage and deploy resources consistently across accounts and regions. This ensures that your [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], including their versions and aliases, have identical configurations in every location.

Additionally, AWS App Mesh and AWS Cloud Map can be used to route requests between services and maintain a consistent view of your application's components, regardless of their deployment locations.

### Under the Hood Mechanics

When creating a new [[lambda]] function, AWS automatically assigns it a version number (e.g., "v1"). As you make updates to your code, AWS creates new versions while preserving older ones. Alias creation enables the selection of specific versions to receive incoming events.

[[lambda|Lambda versions and aliases]] interact closely with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] like Amazon [[api-gateway|API Gateway]], [[step-functions|AWS Step Functions]], and Amazon [[sns]]. For instance, when an [[api-gateway|API Gateway]] triggers a [[lambda]] function via an alias, any downstream systems must adhere to the same versioning strategy for consistent behavior.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service           | Use Case                                                              |
|-------------------|------------------------------------------------------------------------|
| [[lambda|Lambda Versions]]   | Versioning code for rollbacks, A/B testing, and blue/green deployments |
| [[cicd|AWS CodePipeline]]  | Implementing continuous integration and delivery ([[cicd|CI/CD]]) pipelines    |
| AWS Amplify       | Simplifying frontend development using pre-built UI components         |
| [[glue|AWS Glue]]          | Building ETL jobs to process data stored in AWS data stores             |

Anti-pattern: Using [[lambda|Lambda versions and aliases]] for long-term storage of historical data. Instead, utilize AWS [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] lifecycle [[policies]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]] for archival purposes.

## [[RDS_Instance_Types|3. Security & Governance]]

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "aws:SourceVpce": "${aws_vpc.main.vpce_id}"
        }
      }
    }
  ]
}
```
Cross-account access:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "lambda:InvokeFunction",
      "Resource": "arn:aws:lambda:us-east-1:012345678901:function:MyFunctionName",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:execute-api:us-east-1:123456789012:*/*/*/MyApiName/*"
        }
      }
    }
  ]
}
```
Organization SCPs:
```json
{
  "Sid": "DenyUnapprovedLambdaVersions",
  "Effect": "Deny",
  "Actions": [
    "lambda:CreateVersion",
    "lambda:PublishVersion"
  ],
  "Resource": [
    "*"
  ],
  "Condition": {
    "StringNotEqualsIfExists": {
      "[[lambda]]:FunctionResourceId": [
        "arn:aws:[[lambda]]:${aws_caller_identity.current.account_id}:function:*: