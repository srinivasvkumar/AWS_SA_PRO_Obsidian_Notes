**Advanced Architecture**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]] is a serverless edge computing capability that allows you to run JavaScript code in front of a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distribution. This enables low-latency customizations at the edge without deploying or managing infrastructure. The internal architecture consists of three main components: [[Lambda@Edge]], Viewer Request/Response Triggers, and Origin Request/Response Triggers.

*[[Lambda@Edge]]*: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]] leverage [[Lambda@Edge]], which extends [[lambda|AWS Lambda]] functionality to enable running code across AWS locations globally while providing the same features as standard [[lambda]]. It supports two execution environments: Node.js 12.x and Python 3.8.

*Viewer Request/Response Triggers*: These triggers allow you to modify incoming requests from viewers and outgoing responses before they reach the viewer. For example, adding headers, URL rewrites, and A/B testing.

*Origin Request/Response Triggers*: These triggers let you customize how [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] interacts with your origin servers. They can be used for tasks such as request collapsing, [[api-gateway|caching]] static content based on query strings, and modifying outbound cache headers.

[[RDS_Instance_Types|Global scale considerations]] include handling over 200TBps of data transfer and sub-100ms latencies worldwide. Under the hood, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]] execute within the AWS Nitro system, which abstracts underlying hardware resources while ensuring consistent performance regardless of location.

**Comparison & Anti-Patterns**

| Service               | Use Case                                                              |
| --------------------- | -------------------------------------------------------------------- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]]   | Lightweight transformations, header manipulation, URL rewrites       |
| [[lambda|AWS Lambda]]           | Heavy computational workloads, third-party API calls                |
| [[appsync|AWS AppSync]]           | Real-time applications requiring GraphQL APIs and WebSocket support  |
| AWS [[api-gateway|API Gateway]]      | RESTful services, OAuth authorization, proxy integration            |

Anti-patterns:

* Overusing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]] for heavy computations better suited for [[lambda|AWS Lambda]].
* Mixing multiple functionalities in one [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] Function causing confusion and maintenance issues.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve granting permissions to specific actions using JSON policy statements. Example:

```json
{
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/MyRole"
    },
    "Action": "cloudfront:UpdateFunction",
    "Resource": "arn:aws:cloudfront:us-east-1:123456789012:function/MyFunction/*",
    "Condition": {
        "StringEquals": {
            "AWS:SourceVPC": "vpcs/vpc-abcdefgh"
        }
    }
}
```

Cross-account access requires creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account and allowing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to assume the role. Organization Service Control [[policies]] (SCPs) can enforce usage [[control-tower|guardrails]] by limiting who can create, update, or delete [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]].

**Performance & Reliability**

Throttling limits depend on the type of function and location. For example, Viewer Request/Response Functions have a maximum invocation rate of 100K requests per minute. To handle high traffic scenarios, implement exponential backoff strategies using retry logic with Jitter to avoid overwhelming the system.

HA/DR patterns involve distributing functions across multiple regions to reduce latency and provide failover capabilities. By default, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]] are available in all supported AWS Regions.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include monitoring and adjusting the number of function invocations, duration, and memory usage. Calculation example:

* Function A: 50M requests, 100ms average duration, 128MB memory = $0.0000002 \* 50M + ($0.0000000000000108 \* 128MB \* 50M) = $0.00108 + $0.00624 = $0.00732 (excluding data transfer costs)

**Professional Exam Scenarios**

Scenario 1:
You need to optimize a media streaming website's performance by reducing latency when serving thumbnails. The thumbnail images are currently stored in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and served via [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]. Which action would you take?

Correct Answer: Implement a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] Function to resize thumbnails on-the-fly, reducing the amount of data transferred and improving load times.

Incorrect Answer: Migrate thumbnails to [[lambda|AWS Lambda]] and implement a serverless application to generate and serve them.

Scenario 2:
Your organization uses [[organizations|AWS Organizations]] with a centralized [[billing]] model. You want to limit [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] Function creation only to specific teams while maintaining full control over their execution. How would you achieve this goal?

Correct Answer: Create an Organization Service Control Policy ([[SCP]]) restricting [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] Function creation at the root level and then create a group containing allowed teams. Grant these teams permission to create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]] through an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy.

Incorrect Answer: Use separate AWS accounts for each team and manage cross-account access using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles. While effective, this approach introduces additional management complexity and higher costs compared to using an [[SCP]].