## Advanced Architecture

At its core, X-Ray is a distributed tracing system that allows developers to understand the behavior of their applications. It provides insights into performance issues, application bugs, and other unexpected behaviors. Understanding X-Ray's [[RDS_Instance_Types|internals]] can help you make informed decisions when designing and implementing your solutions.

X-Ray has several components:

- **Daemons**: Run on [[ec2]] instances or containers, responsible for collecting trace data.
- **Service**: An HTTP API that receives and stores the collected trace data.
- **[[lambda|AWS Lambda]]**: Integrates with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] to automatically collect trace data.
- **Proxies**: Injected into [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] and Amazon [[api-gateway|API Gateway]], they collect trace data from incoming requests.

**[[RDS_Instance_Types|Global Scale Considerations]]**: X-Ray supports multi-region deployments by replicating traces across regions. This ensures that users see consistent information about their applications' performance, regardless of location. However, it's important to note that X-Ray charges per region, so enabling multiple regions may increase costs.

## Comparison & Anti-Patterns

| Service           | Use Case                                                              |
| ----------------- | -------------------------------------------------------------------- |
| **[[cloudwatch]]**    | Monitoring metrics and logs; not focused on distributed tracing     |
| **Distro Sampler** | Random sampling for load testing and profiling; less precise than X-Ray |
| **Honeycomb**     | High-cardinality observability; requires additional infrastructure |

**Anti-Patterns**:

- Using X-Ray as a general-purpose monitoring solution instead of [[cloudwatch]].
- Relying on X-Ray for load testing and profiling, which should be handled by Distro Sampler or similar tools.

## [[appsync|Security]] & Governance

X-Ray integrates with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] for fine-grained control over user permissions. For cross-account access, you can create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in one account and allow another account to perform actions using that role. Additionally, you can enforce organization-wide restrictions through Service Control [[policies]] (SCPs):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "xray:*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:Account": ["123456789012", "098765432109"]
        }
      }
    }
  ]
}
```

This example denies all X-Ray actions except for those performed in accounts 123456789012 and 098765432109.

## Performance & Reliability

X-Ray throttles requests based on the number of entities created within a specific period. To mitigate throttling issues, implement exponential backoff strategies in your code.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], ensure that your X-Ray daemons run in multiple Availability Zones. This way, if one AZ goes down, your application will continue to function without interruption.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

X-Ray offers granular cost controls through usage-based pricing. The primary cost drivers are the number of traces stored and the duration of storage. To optimize costs, consider setting up [[billing]] alarms for unusual X-Ray spending or configure data retention periods according to your requirements.

Calculation Example:
Suppose you have an application with 1 million requests per month, each generating a single trace. If storing a trace for 30 days costs $0.03, then your monthly X-Ray cost would be:

(1,000,000 \* 30 days) \* $0.03 = $9,000

## Professional Exam Scenarios

### Scenario 1: Multi-account Strategy

You are working on a multi-account environment for a large enterprise. The company wants to centralize X-Ray data in a separate AWS account to simplify management and reduce costs. How can you achieve this?

Correct Answer: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the centralized account and delegate permissions to other accounts. Attach an [[SCP]] to restrict X-Ray actions to the centralized account.

Incorrect Answer: Set up [[AWS_SA_PRO_Obsidian_Notes/Master/08-data-analytics/others|AWS Data Exchange]] to share X-Ray data between accounts.

Justification: [[AWS_SA_PRO_Obsidian_Notes/Master/08-data-analytics/others|AWS Data Exchange]] is not designed for sharing X-Ray data directly. Instead, use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and SCPs to securely manage X-Ray data across multiple accounts.

### Scenario 2: Fine-Grained Access Control

Your company uses [[organizations|AWS Organizations]] and wants to grant access to X-Ray only for specific teams working on different projects. How can you accomplish this while minimizing potential [[appsync|security]] risks?

Correct Answer: Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to define individual team permissions for X-Ray actions. Use condition keys to limit resource access based on tags or specific project identifiers.

Incorrect Answer: Share X-Ray data with [[AWS_SA_PRO_Obsidian_Notes/Master/08-data-analytics/others|AWS Lake Formation]] and set up fine-grained access control at the database level.

Justification: While Lake Formation is useful for managing data lakes, it does not provide direct integration with X-Ray for fine-grained access control purposes.