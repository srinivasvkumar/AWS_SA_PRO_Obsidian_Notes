**Advanced Architecture**

[[api-gateway|API Gateway]]'s [[RDS_Instance_Types|internals]] consist of various components like caches, throttling mechanisms, canary deployments, and regional endpoints. The [[api-gateway|API Gateway]]'s underlying architecture is designed to support high throughput, distribute requests globally, and provide low latency connections.

[[RDS_Instance_Types|Global Scale Considerations]]:
- Edge-optimized APIs leverage [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to distribute content worldwide, reducing latency.
- Regional APIs utilize regional endpoint services in each supported region.

Under the Hood Mechanics:
- Request/Response flow: API calls are routed from clients to [[api-gateway|API Gateway]], followed by integration points such as [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] or HTTP(S) backends.
- [[api-gateway|Caching]]: Deployed API [[api-gateway|stages]] can have cache settings configured to improve performance and reduce request processing costs.
- Deployment [[api-gateway|stages]]: [[api-gateway|Stages]] represent different environments (e.g., testing, staging, production) with separate API configurations and custom domain names.

**Comparison & Anti-Patterns**

| Factors               | [[api-gateway|API Gateway]]                 | Alternatives                         |
| --------------------- | ---------------------------- | ------------------------------------ |
| Protocol Support      | HTTP(S) only                | gRPC, WebSocket using ALB/NLB       |
| Integration Options   | [[lambda]], HTTP(S) Backend     | Directly invoke [[lambda]], [[appsync]]    |
| Scaling              | Built-in                    | Custom solutions ([[ecs]], [[eks]])          |
| Cost                 | Higher base cost             | Lower base cost but more management |

Common anti-patterns include:
- Using REST APIs when GraphQL would be more efficient.
- Implementing real-time communication using long polling instead of WebSockets.
- Overusing [[api-gateway|API Gateway]] for simple tasks that could be handled directly by [[lambda]].

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "execute-api:Invoke",
      "Resource": "arn:aws:execute-api:region:account-id:*/*",
      "Condition": {
        "StringEquals": {
          "aws:SourceVpc": "vpc-abcdefgh"
        }
      }
    }
  ]
}
```
Cross-Account Access:
- Enable resource [[policies]] on [[api-gateway|API Gateway]] to allow cross-account access.
- Create a custom [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role with specific permissions and attach it to the calling account.

Organization Service Control [[policies]] (SCPs):
- Enforce organizational [[appsync|security]] rules by setting up SCPs at the organization level.
- Prevent unauthorized access to critical resources like [[api-gateway|API Gateway]].

**Performance & Reliability**

Throttling Limits:
- Burst limit: 5,000 requests per second per stage.
- Rate limit: 10,000 requests per second per stage.

Exponential Backoff Strategies:
- Implement client-side retry logic using an exponential backoff algorithm to handle transient [[api-gateway|errors]].
- Set initial timeout values and increment them exponentially for retries.

HA/DR Patterns:
- Distribute API traffic across multiple regions using edge-optimized APIs.
- Utilize [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and DNS failover to ensure high availability.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular Cost Controls:
- Monitor usage metrics and set alarms for threshold breaches.
- Utilize tags to organize your APIs and track spending.

Calculation Examples:
- Calculate [[api-gateway|API Gateway]] costs based on data transfer, API calls, and cache usage.
- Estimate total cost by adding other related services ([[lambda]], [[dynamodb]], etc.).

**Professional Exam Scenarios**

Scenario 1:
A company wants to implement a centralized [[api-gateway|API gateway]] solution for their applications running in multiple AWS accounts. They require granular control over resources and need to enforce consistent [[appsync|security]] [[policies]].

Correct Answer:
Use Amazon [[api-gateway|API Gateway]] with Resource [[policies]] and Organizational SCPs.

Justification:
Resource [[policies]] enable [[api-gateway|API Gateway]] to interact with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] while enforcing [[appsync|security]] [[policies]]. Organizational SCPs help maintain consistency across accounts.

Incorrect Answers:
- Using [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]]: Not suitable for external APIs.
- Sharing resources between accounts: Provides limited governance capabilities.

Scenario 2:
An e-commerce platform needs to process millions of orders daily with minimal latency. Their current [[api-gateway|API Gateway]] setup has trouble meeting these requirements during peak hours.

Correct Answer:
Implement Edge-Optimized APIs and distribute API traffic across multiple regions.

Justification:
Edge-Optimized APIs utilize [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] to distribute content worldwide, reducing latency. Multiple regions increase capacity and decrease the likelihood of bottlenecks.

Incorrect Answers:
- Adding more instances of the API: Increases complexity without addressing latency issues.
- Upgrading to a higher API plan: May not provide sufficient improvement in performance.