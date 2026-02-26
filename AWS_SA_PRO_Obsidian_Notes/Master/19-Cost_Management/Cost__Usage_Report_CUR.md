## Advanced Architecture

At its core, Cost and Usage Reports (CUR) is an AWS service that provides cost and usage data in a programmatically accessible format. The CUR reports contain line items representing your cost and usage at the most granular level available. This includes nested usage types, which provide insights into the products and actions consuming resources.

The CUR service operates at a global scale, aggregating data from all linked AWS accounts within an organization. Under the hood, CUR relies on the [[billing|AWS Billing and Cost Management]] service to collect data, which is then processed by [[AWS_SA_PRO_Obsidian_Notes/Master/07-databases/athena|Amazon Athena]] to generate the report.

To enable CUR, follow these steps:

1. Navigate to the AWS [[billing]] Console
2. Select "[[billing|Cost Explorer]]"
3. Click "Reports" and then "Create report"
4. Name the report, select the desired fields, and choose whether to include any tags
5. Save the report

![](mermaid-architecture.md)

## Comparison & Anti-Patterns

### Table: CUR Alternatives and Anti-Patterns

| Service              | Use Case                                                         | Anti-Pattern                               |
| -------------------- | --------------------------------------------------------------- | ---------------------------------------------- |
| [[billing|Cost Explorer]]        | Basic cost analysis and visualization                             | Limited customizability or historical data   |
| [[Budgets]]             | Creating and tracking [[Budgets]]                                    | Overly restrictive alerts or scope          |
| [[organizations|Consolidated Billing]] | Multi-account cost management                                    | Lack of visibility into individual account costs |
| AWS Resource Groups | Grouping resources based on attributes for cost reporting       | Inaccurate tagging or resource selection      |
| [[organizations|AWS Organizations]]   | Centralized [[billing]], management, and policy enforcement            | Insufficient control over member accounts     |

## [[appsync|Security]] & Governance

To ensure secure access and governance when using CUR, you can implement the following measures:

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cur:DescribeReportDefinitions",
                "cur:DescribeReportDefinitionsForIdentity",
                "cur:GetCostAndUsage",
                "cur:GetCostAndUsageWithResources"
            ],
            "Resource": "*",
            "Condition": {
                "StringEqualsIfExists": {
                    "aws:SourceVpce": "<VPCE_ID>"
                }
            }
        }
    ]
}
```

Replace `<VPCE_ID>` with your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Endpoint ID to allow cross-account access.

### Cross-Account Access

Configure a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] endpoint to allow secure access to CUR without traversing the public internet:

1. Open the Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] console
2. Create an endpoint for the CUR service
3. Attach the necessary permissions policy

### Organization SCPs

Implement Service Control [[policies]] (SCPs) at the organization level to enforce restrictions on CUR usage. For example, deny specific API operations or prevent users from creating new CUR reports.

## Performance & Reliability

CUR has throttling limits in place to maintain performance and reliability. If you encounter throttling issues, implement exponential backoff strategies as needed.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute your CUR setup across multiple regions by setting up separate AWS accounts and enabling CUR for each one.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Use CUR to identify [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] opportunities by analyzing usage trends and identifying areas for potential savings. Implement granular cost controls through tagging, allowing you to track costs down to the resource level.

Example cost calculation:

Suppose you have two [[ec2]] instances with the following configurations:

- Instance A: m5.large, 2 instances, 720 hours per month, $0.08 per hour
- Instance B: r5.xlarge, 3 instances, 800 hours per month, $0.16 per hour

Using CUR, calculate the monthly cost for these instances:

1. Retrieve the number of instance-hours used for each instance type
2. Calculate the total cost for each instance type

Instance A: 2 \* 720 hours \* $0.08 = $115.20
Instance B: 3 \* 800 hours \* $0.16 = $384

Total Monthly Cost: $115.20 + $384 = $509.20

## Professional Exam Scenarios

### Scenario 1: Multi-Account Cost Management

An organization manages multiple AWS accounts and wants to track costs centrally while maintaining visibility into individual account costs. Which services should they use?

Correct answer: [[organizations|Consolidated Billing]], [[organizations|AWS Organizations]], and CUR

Incorrect answer: Using only CUR would not provide centralized [[billing]] or management capabilities.

### Scenario 2: Secure Access to CUR

An AWS administrator needs to grant a team read-only access to CUR data but does not want to expose their entire AWS environment. How should they set up secure access to CUR?

Correct answer: Configure a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] endpoint for CUR and attach an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy limiting access to the required API operations.

Incorrect answer: Sharing CUR credentials or granting full admin access to the team would introduce unnecessary [[appsync|security]] risks.