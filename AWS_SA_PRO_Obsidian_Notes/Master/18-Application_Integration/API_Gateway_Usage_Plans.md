**[[RDS_Instance_Types|1. Advanced Architecture]]: [[api-gateway|API Gateway]] Usage Plans**

[[api-gateway|API Gateway]] Usage Plans allow for advanced management of API access, including throttling, quota enforcement, and API key validation. They can span multiple [[api-gateway|stages]] within an API, and work across all methods in those [[api-gateway|stages]]. Usage plans are particularly useful in multi-account strategies, as they enable centralized control over who can access APIs and how they're accessed.

The internal mechanics of Usage Plans involve several components:

- **API Keys**: These are used to authenticate incoming requests and enforce rate limits. API keys are created within Usage Plans and can be enabled or disabled individually.
- **Throttling**: This is applied per API Key, and enforces request rates at both the method level and the overall API level. Burst limits and rate limits can be configured separately.
- **Quotas**: Quotas restrict the number of calls made by each API key during a specific time period. Quotas can be set up on a per-stage basis.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Advantages | Disadvantages | Use Cases |
| --- | --- | --- | --- |
| [[api-gateway|API Gateway]] w/Usage Plans | Centralized control, metering, and [[appsync|security]] | Higher cost than other options | Multi-account API access, managing partner access |
| [[lambda]] Authorizers | Dynamic authorization logic | Increased complexity, potential performance impact | Fine-grained, policy-based authorization |
| [[cognito]] User Pools | Secure [[api-gateway|authentication]], easy integration with mobile & web apps | Limited flexibility, tightly coupled with [[cognito]] | Mobile & web applications requiring user management |

Anti-pattern: Using Usage Plans when simple access control is sufficient. Opt for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles or permissions directly attached to the API instead.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for Usage Plans may include [[cloudformation|conditions]] based on resource tags, IP addresses, and API keys. For example:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "arn:aws:execute-api:region::*/methods/*/paths/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceVpce": "vpce-12345678"
                },
                "StringEqualsIfExists": {
                    "apiKey": ["abc123", "def456"]
                }
            }
        }
    ]
}
```
Cross-account access can be granted using appropriate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, while Service Control [[policies]] (SCPs) within [[organizations|AWS Organizations]] can limit which services members can create Usage Plans for.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits depend on the chosen usage plan. The default is 10,000 requests per day, but this can be increased up to 10 million requests per day. When these limits are reached, clients should implement exponential backoff strategies to avoid constant retries and potential exhaustion of available connections.

HA/DR patterns can be implemented by deploying [[api-gateway|API Gateway]] resources across multiple regions and enabling regional endpoints. Usage Plans can then be duplicated across these endpoints to ensure consistent access [[policies]] and quotas.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls can be achieved by creating separate Usage Plans for different groups of users or applications. This allows you to monitor costs more closely and adjust thresholds accordingly. Additionally, you can enable detailed monitoring and [[vpc-flow-logs|logging]] for individual Usage Plans, helping identify potential areas for optimization.

**6. Professional Exam Scenario: Multi-Account API Access**

Suppose your organization has two accounts: Account A (dev) and Account B (prod). In Account A, there's an [[api-gateway|API Gateway]] with a REST API that needs to be accessible from Account B via a single URL. How would you accomplish this?

*Create a Usage Plan in Account A and share it with Account B.*

Correct answer:

1. Create a Usage Plan in Account A with desired quotas and throttle settings.
2. Add required API [[api-gateway|stages]] to the Usage Plan.
3. Generate an API key and attach it to the Usage Plan.
4. Share the Usage Plan with Account B by adding the ARN of the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] entity in Account B to the `principalId` field.
5. Instruct users in Account B to use the generated API key to call the API.

Incorrect answer:

1. Attach an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in Account A to the [[api-gateway|API Gateway]], allowing cross-account access.
2. Enable the API in Account B with the same stage as the API in Account A.
3. Call the API in Account B without specifying any API key.

This approach does not provide proper metering and [[appsync|security]] mechanisms since it lacks rate limiting and API key validations.