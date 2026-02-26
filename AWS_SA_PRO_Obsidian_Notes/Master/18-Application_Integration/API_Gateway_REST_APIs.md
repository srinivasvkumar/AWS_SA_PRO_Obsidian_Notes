**Application Integration: [[api-gateway|API Gateway]] REST APIs**

**[[RDS_Instance_Types|1. Advanced Architecture]]**

*Internal Mechanics*: [[api-gateway|API Gateway]] REST APIs operate as a front-end proxy to your backend services. They handle request routing, [[api-gateway|caching]], throttling, [[vpc-flow-logs|logging]], and [[appsync|security]]. [[api-gateway|API Gateway]] uses a simplified HTTP(S) endpoint model, abstracting away underlying server infrastructure.

*[[RDS_Instance_Types|Global Scale Considerations]]*: For global deployments, you can create regional API endpoints in multiple regions and configure them to use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] as a reverse proxy. This gives you a single global URL while minimizing latency via Amazon's extensive edge network.

```mermaid
graph LR
subgraph US East (N. Virginia)
ApiGateway[REST API] --> BackendService
end
subgraph Europe (Ireland)
CloudFront[CloudFront]
end
CloudFront --> ApiGateway
```

*Under The Hood*: Each API method invokes one or more integration backends. These could be [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]], HTTP(S) endpoints, or AWS services like [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] or [[dynamodb]]. Method execution is decoupled from the API call, allowing for independent scaling and fault tolerance.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | When To Use |
|---|---|
| [[api-gateway|API Gateway]] REST API | Stateless, high-volume, bi-directional data flows. Ideal when an abstraction layer over various backends is needed. |
| [[appsync]] | Real-time data synchronization between clients and AWS services. Suitable for mobile, [[iot]], and web applications requiring GraphQL support. |
| [[lambda]] | Event-driven processing of individual records. Recommended for simple, lightweight microservices. |

*Anti-Patterns*: Overusing [[api-gateway|API Gateway]] for simple tasks better suited for [[lambda]] or other services adds complexity without benefits. Also, avoid using it as a direct replacement for application logic within your backends.

**[[RDS_Instance_Types|3. Security & Governance]]**

Implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] using JSON statements that allow fine-grained control based on user identity, resource type, operation, and [[cloudformation|conditions]]. Example policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "arn:aws:execute-api:us-east-1:*:*/*/GET/resource",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": ["192.0.2.0/24"]
                }
            }
        }
    ]
}
```

Cross-account access requires proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, trust [[policies]], and permissions. Organizational Service Control [[policies]] (SCPs) provide centralized governance across accounts.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling protects against unexpected traffic spikes and potential DoS attacks. Set throttling limits at the API level or per method. Implement exponential backoff strategies for handling throttled requests.

High availability/disaster recovery patterns include deploying [[api-gateway|API Gateway]] REST APIs across multiple regions and utilizing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and DNS failover mechanisms.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls involve setting usage plans, enabling [[api-gateway|caching]], and monitoring costs through AWS [[billing|Cost Explorer]]. Calculate costs using the formula: `(Number of API calls * Call price) + (Number of GB data transferred * Data transfer price)`.

**6. Professional Exam Scenario: Multi-Account Strategy**

Scenario: A large enterprise has several departments, each managing its own AWS account. They want to build a centralized reporting system that aggregates data from all departmental accounts.

Correct Answer: Create a dedicated reporting account and set up an [[api-gateway|API Gateway]] REST API in this account. Configure cross-account access by creating [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles in each departmental account granting necessary permissions to the reporting account's [[api-gateway|API Gateway]] REST API.

Incorrect Answers: Using a single account for everything increases complexity and potential [[appsync|security]] risks. Relying solely on [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering or Direct Connect would limit flexibility and scalability.

**6. Professional Exam Scenario: Global Deployment**

Scenario: A global company wants to serve users worldwide with minimal latency while maintaining a single global URL.

Correct Answer: Deploy [[api-gateway|API Gateway]] REST APIs regionally and configure them to use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] as a reverse proxy. This setup provides a global presence through Amazon's edge network while maintaining regional redundancy.

Incorrect Answers: Serving all traffic from a single region may introduce unacceptable latency for some users. Isolating each region without any interconnectivity prevents load balancing and failover capabilities.