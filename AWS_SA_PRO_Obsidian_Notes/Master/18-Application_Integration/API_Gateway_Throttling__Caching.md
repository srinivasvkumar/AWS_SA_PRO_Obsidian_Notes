**Advanced Architecture**

[[api-gateway|API Gateway]] throttling and [[api-gateway|caching]] are critical components of any scalable serverless application. At its core, [[api-gateway|API Gateway]] is a fully managed service that allows developers to create, publish, maintain, monitor, and secure APIs at any scale.

Internally, [[api-gateway|API Gateway]] uses a distributed architecture based on a network of servers and caches. The system automatically scales to meet incoming request traffic while maintaining high availability and low latency. To ensure fair usage and prevent abuse, [[api-gateway|API Gateway]] implements various throttling mechanisms.

Throttling in [[api-gateway|API Gateway]] operates at two levels:

1. **Rate-based throttling**: This mechanism restricts the number of requests within a specified time window. It can be configured at both account level and method level. Account-level throttling applies to all APIs in an account, while method-level throttling provides more granular control over specific API methods.
2. **Token-based throttling (burst mode)**: This mechanism uses a token bucket algorithm to manage bursty traffic. Each token represents one or more requests, depending on the configuration. By default, each API has a token bucket with 500 tokens and a refill rate of 10 tokens per second.

[[RDS_Instance_Types|Global scale considerations]] include edge-optimized regional endpoints, which allow clients to connect to the nearest endpoint for lower latency and higher throughput. Edge-optimized regional endpoints also support custom domain names, enabling you to provide a consistent brand experience across multiple regions.

**Comparison & Anti-Patterns**

| Service               | Suitable For                                                          | Not Suitable For                                                       |
| --------------------- | -------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| [[api-gateway|API Gateway]] Throttling | High-traffic APIs requiring fine-grained throttling controls         | Applications with unpredictable spikes in traffic                      |
| [[lambda]] Concurrency    | Serverless workloads requiring automatic scaling and resource isolation | Stateful applications or long-running operations                        |
| Elastic Beanstalk     | Containerized and non-containerized web applications                   | Real-time or event-driven applications without HTTP interfaces           |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|App Runner]]           | Containerized web applications requiring minimal management            | Long-running batch jobs or event-driven applications                     |

Common anti-patterns include using [[api-gateway|API Gateway]] throttling as a substitute for proper capacity planning and load testing. Overreliance on throttling may lead to unexpected [[api-gateway|errors]] or performance degradation during traffic peaks.

**[[appsync|Security]] & Governance**

To implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], consider using JSON statements like this example:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "execute-api:Invoke",
      "Resource": "arn:aws:execute-api:us-east-1:*:*/*/POST/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": [
            "192.0.2.0/24",
            "203.0.113.0/24"
          ]
        }
      }
    }
  ]
}
```
Cross-account access can be granted by creating a policy statement allowing specific actions on the [[api-gateway|API Gateway]] ARN. Additionally, organization Service Control [[policies]] (SCPs) can enforce restrictions on [[api-gateway|API Gateway]] resources across accounts.

**Performance & Reliability**

To handle throttling limits, implement exponential backoff strategies using SDKs or libraries. In Python, for instance, you could use the following code:
```python
import time

def call_api(attempts):
    if attempts > 0:
        time.sleep(2 ** attempts)
        # Call API here
    else:
        raise Exception("Too many retries")
```
HA/DR patterns involve deploying [[api-gateway|API Gateway]] endpoints across multiple regions and leveraging [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] for failover.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be achieved by configuring individual API stage costs, including data transfer fees and API calls. Calculating costs requires understanding the [[billing]] model, which includes charges for REST and WebSocket APIs, private [[api-gateway|integrations]], and data transfer.

**Professional Exam Scenarios**

Scenario 1: A company wants to limit the number of requests made to their API from a single IP address. They have set up rate-based throttling but want to further restrict the maximum number of requests per minute. How would you configure additional throttling rules?

Correct Answer: Implement token-based throttling with a smaller token bucket size and faster refill rate than the current account-level throttling settings.

Incorrect Answer: Increase the rate limit for rate-based throttling.

Justification: Increasing the rate limit does not restrict requests from a single IP address. Token-based throttling enforces stricter limits on a per-client basis.

Scenario 2: A media streaming platform needs to serve millions of users concurrently. They plan to use [[api-gateway|API Gateway]] to distribute content but are concerned about potential throttling [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]]. What steps should they take to optimize their solution for performance and reliability?

Correct Answer: Distribute [[api-gateway|API Gateway]] endpoints across multiple regions, enable [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] integration, and implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] for failover.

Incorrect Answer: Set up a single [[api-gateway|API Gateway]] instance with increased throttle limits.

Justification: Distributing endpoints reduces latency and improves reliability compared to a single centralized instance. Enabling [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] integration offloads traffic to the edge network, while [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] ensure high availability by rerouting traffic away from failed resources.