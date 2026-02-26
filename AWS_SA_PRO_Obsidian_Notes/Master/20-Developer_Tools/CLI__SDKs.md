---
aliases:
- "Python example"
---

**[[RDS_Instance_Types|1. Advanced Architecture]]**

At the professional level, understanding the [[RDS_Instance_Types|internals]] of AWS CLI & SDKs is crucial for building scalable and efficient solutions.

* **Service Quotas:** Understanding default service quotas and how to request quota increases is essential when designing large-scale systems. For example, using Amazon [[sns]] topics, the default maximum number of subscriptions per topic is 10 million. If you need more than that, you can request a quota increase.
* **[[RDS_Instance_Types|Global Scale Considerations]]:** When working with global applications, it's important to understand how data replication works across regions. For instance, [[dynamodb|DynamoDB Global Tables]] provide a fully managed solution for multi-region tables, while [[AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/datasync|AWS DataSync]] offers asynchronous data transfers between your AWS resources in different regions.
* **Under the Hood:** To optimize performance, it's useful to know what happens under the hood. For example, when calling AWS API operations through the AWS CLI or an SDK, they communicate via HTTP/HTTPS using a [[kms|signing]] process based on the AWS Sigv4 algorithm. This ensures secure communication between client and service.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service                      | When to Use                                                                         | Alternatives               |
| ---------------------------- | ---------------------------------------------------------------------------------- | --------------------------- |
| AWS CLI                     | Ad-hoc tasks, quick automation, scripting                                          | SDKs, [[cloudformation]]        |
| AWS SDKs                    | Longer-running processes, application development                                   | AWS CLI, [[lambda]], [[glue]]       |
| AWS Cloud Development Kit    | Model-driven development, infrastructure-as-code                                  | [[cloudformation]], Terraform  |
| AWS Serverless Application | Building event-driven serverless applications at scale                           | Containers, [[ec2]] instances  |

Anti-patterns include:

* Using CLI commands in production code instead of SDKs
* Mixing infrastructure management and application logic in the same repository

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve multiple actions, resources, and [[cloudformation|conditions]]. Here's an example policy allowing specific users to perform `dynamodb:UpdateItem` action on a [[dynamodb]] table only if the current date is within a specified range:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "dynamodb:UpdateItem",
      "Resource": "arn:aws:dynamoddb:us-west-2:123456789012:table/example-table",
      "Condition": {
        "DateGreaterThan": {"aws:CurrentTime": "2022-01-01T00:00:00Z"},
        "DateLessThan": {"aws:CurrentTime": "2023-01-01T00:00:00Z"}
      }
    }
  ]
}
```

Cross-account access involves creating [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles in the trusted account and assuming them from the trusting account. Organizational Service Control [[policies]] (SCPs) limit permissions across accounts within an organization.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits vary depending on the service. For example, Amazon [[sqs]] has a maximum of 120,000 messages per second per queue. Handling throttling includes implementing exponential backoff strategies such as:

```yaml
# Python example
import time

max_retries = 5
base_sleep_time = 1
for attempt in range(max_retries):
    try:
        # Call your AWS API here
        break
    except Exception as e:
        if attempt < max_retries - 1:
            time.sleep(base_sleep_time * (2 ** attempt))
        else:
            raise e
```

HA/DR patterns depend on specific services. Generally, you would deploy resources across multiple AZs, enable cross-region replication where available, and leverage [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] for automatic failover.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls involve setting up [[billing]] alarms, [[Budgets]], and [[billing|cost explorer]] reports to track spending trends. Additionally, using tools like AWS Cost Anomaly Detection helps identify unexpected costs.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
> Your company needs to build an automated system that scales quickly during peak times but minimizes costs during off-peak hours. The system should automatically adjust capacity based on demand.

Correct answer: Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Auto Scaling]] with [[ec2]] instances or [[lambda|AWS Lambda]] functions combined with AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|App Runner]] or Amazon [[ecs]] [[Fargate]]. Implement granular cost controls using AWS [[billing|Cost Explorer]] and [[billing]] alarms.

Incorrect answer: Use static [[ec2]] instances without auto-scaling.

Scenario 2:
> As a Solutions Architect, you must develop a multi-account strategy for managing several AWS accounts spread across different departments. Each department has its own set of requirements and restrictions regarding resource usage.

Correct answer: Leverage [[organizations|AWS Organizations]], create separate OUs for each department, apply SCPs to restrict permissions, and utilize [[service-quotas|AWS Service Quotas]] to manage individual account limits.

Incorrect answer: Manage all accounts separately without any centralized oversight or governance.