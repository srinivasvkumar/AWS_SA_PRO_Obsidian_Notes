**[[RDS_Instance_Types|1. Advanced Architecture]]**

HTTP APIs in Amazon [[api-gateway|API Gateway]] provide a lightweight, efficient way to build and deploy RESTful services. They operate at the HTTP level, supporting various methods like GET, POST, PUT, DELETE, etc. Unlike REST APIs, they don't support AWS service integration or data [[api-gateway|mapping templates]].

Internally, an HTTP API consists of API endpoints that map to backend services (known as [[api-gateway|integrations]]) through URLs. It uses Edge-optimized regional endpoints by default but can be configured for regional or private [[api-gateway|integrations]]. For global scale, you can use Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudMap]] to define custom domain names with regional API endpoints backed by [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Network Load Balancer (NLB)]].

[Mermaid Diagram]
graph LR
A[HTTP API] --> B[Edge-Optimized Regional Endpoint]
B --> C[Integration Backend]
D[API] -- "Uses" --> E[CloudMap]
E -- "Defines" --> F[Custom Domain Name]
F -- "Backed By" --> G[ALB or NLB]

Quotas for HTTP APIs include a maximum limit of 250 API methods per account and 1000 concurrent connections per stage. You can request quota increases from AWS Support.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Feature | HTTP API | REST API | [[lambda]] Proxy |
|---|---|---|---|
| Method Support | Basic CRUD operations only | Full CRUD + [[Git_hub_notes/certified-aws-solutions-architect-professional-main/16-other/kendra|others]] | All [[lambda]] triggers |
| Payload Size | ~20 MB (compressed) | ~10 MB (uncompressed) | ~28 MB (uncompressed) |
| Performance | Faster than REST due to less overhead | Slower than HTTP due to more overhead | Varies based on [[lambda]] execution time |
| Cost | Lower costs due to less overhead | Higher costs due to more overhead | Varies based on [[lambda]] execution time |

Anti-patterns include using REST APIs when simple HTTP APIs would suffice, leading to unnecessary complexity and higher costs. Another mistake is not enabling [[api-gateway|caching]] on responses, which can result in suboptimal performance.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve allowing specific IP addresses or restricting access to particular HTTP verbs. Here's an example JSON policy snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "arn:aws:execute-api:region:account-id:*/*/path/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": [
                        "x.x.x.x/x",
                        "y.y.y.y/y"
                    ]
                },
                "HttpVerb": {
                    "aws:SourceVpc": ["GET", "POST"]
                }
            }
        }
    ]
}
```

Cross-account access requires setting up a correct principal, action, and resource triad in the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy. Organizational Service Control [[policies]] (SCPs) can enforce restrictions across accounts within an organization.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits depend on the chosen API type. For HTTP APIs, there's a default limit of 10,000 requests per second, which can be increased via support tickets. Implementing exponential backoff strategies helps manage [[api-gateway|errors]] during high load scenarios.

HA/DR patterns involve deploying multiple API instances across different regions and leveraging [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] to reroute traffic if one region becomes unavailable.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls include monitoring usage metrics such as API calls, data transfer, and cache hits/misses. Calculating costs involves understanding the pricing model: $3.50 per million API calls + data transfer charges.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario A: An e-commerce company wants to implement a new search feature using an external search service. The solution must handle global scale while minimizing latency. Which architecture would you recommend?

Recommended Architecture: Use an HTTP API with Edge-optimized regional endpoints backed by the external search service. Configure DNS failover using [[route53]] [[route53|health checks]] to ensure high availability. Enable [[api-gateway|caching]] on responses for optimal performance.

Incorrect Answer: Using a REST API adds unnecessary overhead and complexity.

Scenario B: A media company needs to securely expose their video transcoding service to several partner companies. Each partner should have access only to certain transcoding profiles.

Recommended Architecture: Set up separate HTTP APIs for each partner, limiting their access to specific transcoding profile(s) using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]. Use [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]] for private communication between partners and the transcoding service.

Incorrect Answer: Implementing this with [[lambda]] proxy [[api-gateway|integrations]] lacks necessary [[appsync|security]] features and complicates management.