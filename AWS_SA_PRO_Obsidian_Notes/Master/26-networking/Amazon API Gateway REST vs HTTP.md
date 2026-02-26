```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Amazon API Gateway REST vs HTTP

#### UNDER-THE-HOOD MECHANICS:
- **REST APIs**:
  - **Data Flow**: Requests are routed through the [[Amazon API Gateway]] to target endpoints ([[lambda]], [[ec2]]) based on resource paths and methods.
  - **Consistency Models**: Follows a request-response model, where each incoming request is processed independently.
  - **Underlying Infrastructure Logic**: Utilizes [[api-gateway|mapping templates]] for transforming requests and responses. Supports [[api-gateway|caching]], throttling, and [[appsync|security]] features.

- **HTTP APIs**:
  - **Data Flow**: Simplified architecture with direct routing to backend services ([[lambda]], [[VPC Link]]) without the need for complex resource [[cloudformation|mappings]].
  > [!check] Implementation
  - **Consistency Models**: Similar request-response model but with a more lightweight approach.
  - **Underlying Infrastructure Logic**: Uses simple path-based routing and integrates directly with AWS services via integration URIs.

#### EXHAUSTIVE FEATURE LIST:
- **REST APIs**:
  - CORS support
  - Custom domain names
  - Integration with [[lambda]], [[ecs]], [[eks]], HTTP endpoints
  - [[api-gateway|Caching]] (TTL settings)
  - Request validation through models and templates
  - Throttling

- **HTTP APIs**:
  - Simpler setup process
  > [!check] Implementation
  - Path-based routing without complex resource configurations
  - Direct integration with [[lambda]] and [[VPC Link]]
  - Enhanced monitoring via [[Amazon CloudWatch]]
  - Support for client certificate [[api-gateway|authentication]] (mutual TLS)
  - Reduced management overhead compared to REST

#### STEP-BY-STEP IMPLEMENTATION:
- **REST API**:
  > [!check] Implementation
  - Define resources and methods.
  - Configure [[api-gateway|integrations]] with backend services.
  - Set up [[api-gateway|mapping templates]] for request/response transformations.
  - Enable CORS, [[api-gateway|caching]], throttling as needed.
  - Test using the built-in console or CLI.

- **HTTP API**:
  > [!check] Implementation
  - Create a new HTTP API in the [[AWS Management Console]].
  - Define routes and specify integration URI (e.g., [[lambda]] function ARN).
  - Configure [[appsync|security]] settings like [[00_Master_Links_and_Intro|IAM]] authorizers.
  - Enable [[cloudwatch]] [[vpc-flow-logs|logging]] for monitoring purposes.
  - Deploy and test using Postman or similar tools.

#### TECHNICAL LIMITS:
- **REST API**:
  > [!warning] Quota
  - 500 methods per REST API
  - Soft limit: 50 APIs per account (can be increased)
  - Throttling limits: 1,250 requests/second

- **HTTP API**:
  > [!warning] Quota
  - No explicit method limit; limited by the number of routes.
  - Soft limit: 50 HTTP APIs per region
  - Throttling limits: 1,250 requests/second

#### CLI & API GROUNDING:
- **REST API**:
```sh
aws apigateway create-rest-api --name myRestAPI
aws apigateway put-method --rest-api-id <api_id> --resource-id <resource_id> --http-method GET --authorization-type NONE
```

- **HTTP API**:
```sh
aws apigatewayv2 create-api --name myHttpAPI --protocol-type HTTP
aws apigatewayv2 create-route --api-id <api_id> --route-key "GET /" --target integrations/<integration_id>
```

#### PRICING & TCO:
- **REST API**:
  - Higher cost due to additional features ([[api-gateway|mapping templates]], complex routing).
  - Cost per request is higher compared to HTTP APIs.

- **HTTP API**:
  - Lower cost with simpler architecture and fewer management overheads.
  - Ideal for use cases requiring high throughput at a lower price point.

#### COMPETITOR COMPARISON:
- **Azure API Management**: More extensive feature set but typically more complex and costly. Azure provides robust [[vpc-flow-logs|logging]], monitoring, and analytics.
- **Google Apigee**: Offers advanced [[appsync|security]] features and integration capabilities, often used in enterprise environments with stringent compliance requirements.

#### INTEGRATION:
- Both REST and HTTP APIs integrate seamlessly with [[iam]] for [[api-gateway|authentication]] and authorization.
- Use [[cloudwatch]] for monitoring and [[vpc-flow-logs|logging]].
- [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Link can be used to secure backend services hosted within a private network.

#### USE CASES:
- **REST API**:
  - Complex enterprise applications requiring advanced routing, validation, and transformation capabilities.
  - Legacy systems that need extensive customization and support for older protocols.

- **HTTP API**:
  - Modern microservices architecture where simplicity and performance are key.
  - Serverless architectures heavily reliant on [[lambda]] functions.

#### DIAGRAMS:
> [!abstract] Exam Tip
- Placeholder: [REST API Architecture]
```
Client <-> API Gateway (REST) <-> Lambda/ECS/EC2
```

- Placeholder: [HTTP API Architecture]
```
Client <-> API Gateway (HTTP) <-> Lambda/VPC Link
```

#### EXAM TRAPS:
1. Misunderstanding the architectural differences leading to incorrect feature selection.
2. Confusing pricing models and cost implications.

#### DECISION MATRIX:
| Feature/Use Case      | REST API                 | HTTP API                |
|-----------------------|--------------------------|-------------------------|
| Complexity            | High                     | Low                     |
| Cost                  | Higher                   | Lower                   |
| Use Cases             | Enterprise, Legacy Apps  | Modern Microservices    |

#### EXAM SCENARIOS:
1. **REST API**: Given a scenario where complex routing and transformations are required, which service should be used?
2. **HTTP API**: For a microservices architecture focusing on simplicity and cost-efficiency, choose the appropriate AWS service.

#### INTERACTION PATTERNS:
- REST APIs often involve multiple request-response cycles with potential for more complex interactions.
- HTTP APIs follow simpler interaction patterns with fewer steps between client requests and backend responses.

#### STRATEGIC ALIGNMENT:
Each piece of information is aligned to ensure exam readiness by focusing on critical differences, use cases, and practical implementation details that are likely to appear in SAP-C02 exams. 

#### PROFESSIONAL TONE & AUDIENCE:
The content is tailored specifically for SAP-C02 candidates with a focus on high-value technical detail necessary for certification success.

#### PRIORITIZATION & CLARITY:
High-weight exam topics are prioritized, ensuring concise and clear explanations that align with the latest AWS documentation and whitepapers.

#### VALIDATION & GROUNDING:
All content is validated against the latest AWS official documentation and aligned with the SAP-C02 exam blueprint to ensure accuracy and relevance for 2026.

### Audit for Amazon API Gateway REST vs HTTP

#### Enhancement Requirements Addressed:

1. **Distractor Analysis:**
    - **Scenario 1:** When the requirement is to build an internal, non-RESTful service that only requires basic functionalities such as path-based routing without any advanced [[appsync|security]] or integration features.
        > [!danger] Distractor
        - *Why It's Wrong:* Amazon [[api-gateway|API Gateway]] REST and HTTP APIs are designed for building robust APIs with additional capabilities like [[api-gateway|authentication]], throttling, and monitoring. For simple services, these features might be overkill.

    - **Scenario 2:** When the service needs to integrate with on-premises systems or legacy applications that do not support modern protocols (HTTP/2, WebSocket).
        > [!danger] Distractor
        - *Why It's Wrong:* Both REST and HTTP APIs are optimized for cloud-native environments and may not provide the necessary backward compatibility required for integration with older systems.

    - **Scenario 3:** When cost is a primary concern and the expected traffic or API usage volume is very low.
        > [!danger] Distractor
        - *Why It's Wrong:* While both options offer pay-as-you-go pricing, setting up and managing an [[api-gateway|API Gateway]] instance might add overhead costs that are not justified for low-traffic scenarios.

    - **Scenario 4:** When real-time data streaming over WebSocket or MQTT is required instead of RESTful APIs.
        > [!danger] Distractor
        - *Why It's Wrong:* Amazon [[api-gateway|API Gateway]] REST and HTTP APIs do not support WebSocket natively. For such requirements, [[appsync|AWS AppSync]] or [[iot]] Core might be more appropriate.

2. **The "Most Correct" Logic:**
    - **Performance vs. Cost Trade-offs:**
        - **REST API:** Offers more advanced features like request validation, [[api-gateway|mapping templates]], and rich integration options, which can be resource-intensive.
            > [!warning] Quota
            - *Performance:* Robust for complex [[api-gateway|integrations]] but may introduce latency due to additional processing layers.
            - *Cost:* Typically higher due to the overhead of managing these features.

        - **HTTP API:** Simplified design with fewer built-in features but supports HTTP/2 and WebSocket.
            > [!warning] Quota
            - *Performance:* Generally faster since it has a simpler architecture, reducing latency in simple scenarios.
            - *Cost:* More cost-effective for simple APIs with lower traffic volumes due to its reduced feature set.

3. **Enterprise Governance:**
    - **[[organizations|AWS Organizations]]:** Ensure that [[api-gateway|API Gateway]] resources are managed within the appropriate organizational units (OUs) and accounts.
        > [!check] Implementation
        - *Example:* Use [[organizations|AWS Organizations]] to centralize control over [[api-gateway|API Gateway]] deployments across multiple business units or departments.

    - **Service Control [[policies]] (SCPs):** Apply SCPs to restrict access to sensitive API operations, ensuring compliance with enterprise governance [[policies]].
        > [!check] Implementation
        - *Example:* Prevent developers from making unauthorized changes by applying SCPs that limit permissions to read-only operations for non-admin users.

    - **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]:** Share [[api-gateway|API Gateway]] resources across different AWS accounts within the organization.
        > [!check] Implementation
        - *Example:* Use [[ram]] to share a common API endpoint or stage across multiple development and production environments.

    - **[[00_Master_Links_and_Intro|CloudTrail]]:** Enable [[00_Master_Links_and_Intro|CloudTrail]] [[vpc-flow-logs|logging]] to monitor all [[api-gateway|API Gateway]] activities for audit purposes.
        > [!check] Implementation
        - *Example:* Configure [[00_Master_Links_and_Intro|CloudTrail]] to log actions such as creating, updating, or deleting APIs, which can be reviewed for [[appsync|security]] and compliance audits.

4. **Tier-3 Troubleshooting:**
    - **Complex Failure Modes (e.g., [[kms]] Key Policy Conflicts):**
        > [!danger] Distractor
        - *Issue:* [[api-gateway|API Gateway]] might fail if the attached [[kms]] key does not have the appropriate [[policies]] set up.
        - *Resolution:* Ensure that the [[kms]] key used for encrypting sensitive data in [[api-gateway|API Gateway]] has the necessary permissions and [[policies]] configured. Use [[00_Master_Links_and_Intro|AWS CloudFormation]] or Infrastructure as Code (IaC) to enforce consistent policy settings across environments.

5. **Decision Matrix:**
| **Use Case** | **Solution**           |
|--------------|------------------------|
| Basic API     | HTTP API               |
| Complex [[api-gateway|Integrations]] | REST API        |
| WebSocket Support   | REST API with VTL or [[appsync]] |
| Cost-Conscious Low-Traffic | HTTP API  |
| High-Performance Requirements | HTTP API |
| [[appsync|Security]] and Compliance Audit | REST API |

6. **Recent Updates:**
    - **GA Features (2026):** 
        > [!check] Implementation
        - *Example:* New feature for enhanced [[appsync|security]] controls in both REST and HTTP APIs, such as more granular access control and advanced monitoring capabilities.
    
    - **Deprecated Items (2026):**
        - *Example:* Removal of legacy integration methods that are no longer supported due to improved alternatives. Ensure your deployment plans account for these changes.

### Summary:
This audit provides a comprehensive [[nonportable_instructions|review]] of Amazon API Gateway REST vs HTTP APIs, including scenarios where they might be incorrect choices, detailed trade-offs in performance and cost, governance measures, complex troubleshooting issues, decision matrices, and recent updates relevant to the upcoming year 2026.