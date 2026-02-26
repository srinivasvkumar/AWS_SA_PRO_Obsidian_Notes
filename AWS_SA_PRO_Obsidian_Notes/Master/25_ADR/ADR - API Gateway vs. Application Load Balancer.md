```yaml
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
priority: high
aliases: [API Gateway vs. Application Load Balancer]
---

> [!abstract] Exam Tip  
Remember that API Gateway is ideal for microservices/API-driven architectures with features like caching and throttling, while ALB is better suited for load balancing web traffic across multiple targets.

## Enhanced Technical Notes: Amazon API Gateway vs. Application Load Balancer

### UNDER-THE-HOOD MECHANICS:

#### **Amazon API Gateway:**
- **Internal Data Flow:** API Gateway acts as a front door for applications to access data, business logic, or functionality from back-end services.
- **Consistency Models:** Designed to handle HTTP/HTTPS requests and route them to appropriate backend services like Lambda functions, VPC Link, HTTP Proxy integrations, etc.
- **Underlying Infrastructure Logic:** Requests are processed through stages (e.g., test, prod) with each stage having its own caching, throttling, and other configurations.

#### **Application Load Balancer (ALB):**
- **Internal Data Flow:** ALB distributes incoming application traffic across multiple targets such as EC2 instances, ECS/EKS clusters.
- **Consistency Models:** Ensures consistent routing based on client IP address or source security group.
- **Underlying Infrastructure Logic:** Uses HTTP/HTTPS and WebSocket protocols to distribute traffic.

### EXHAUSTIVE FEATURE LIST:

#### **Amazon API Gateway:**
- Stages (test, prod)
- Caching
- Throttling and Rate Limiting
- Integration Types (Lambda, VPC Link, HTTP Proxy)
- Custom Domain Names
- CORS Support
- WebSocket APIs
- API Keys for authentication
- Mutual TLS Authentication
- VTL (Velocity Template Language) for request/response transformation
- AWS Managed Service for Lambda authorizers

#### **Application Load Balancer:**
- Path-based Routing
- Content-Based Routing
- Sticky Sessions
- Cross-Zone Load Balancing
- Health Checks
- Multiple Availability Zones
- SSL/TLS Termination
- Target Groups (EC2, ECS, Lambda)
- Integrated with AWS WAF for enhanced security

### STEP-BY-STEP IMPLEMENTATION:

#### **Amazon API Gateway:**
1. **Initial Configuration:**
   - Create an API in API Gateway.
   - Define resources and methods using RESTful principles.
   - Integrate backend services (Lambda, HTTP endpoint).

2. **Deployment & Stages:**
   - Deploy the API to a stage (e.g., test).
   - Configure caching for better performance.

3. **Security Configuration:**
   - Implement security measures like CORS, API keys, and Lambda authorizers.
   - Enable mutual TLS if required.

4. **Multi-Region Rollout:**
   - Use CloudFormation or CDK to deploy the same configuration across regions.
   - Set up Route 53 for global traffic management.

#### **Application Load Balancer:**
1. **Initial Configuration:**
   - Create an ALB in a specific VPC.
   - Define target groups (EC2, ECS, Lambda).

2. **Routing Rules & Security:**
   - Configure path-based routing rules.
   - Enable SSL/TLS termination and configure security policies.

3. **Health Checks:**
   - Set up health checks for the backend targets.
   - Configure cross-zone load balancing if needed.

4. **Multi-Region Rollout:**
   - Create ALBs in each region’s VPC.
   - Use Route 53 weighted or latency-based routing to distribute traffic.

### TECHNICAL LIMITS (2026):

#### **Amazon API Gateway:**
- 10,000 APIs per AWS account
- 20 stages per API
- Adjustable limits for throttling and caching

#### **Application Load Balancer:**
- 5 ALBs per region per VPC
- 3 target groups per load balancer
- Adjustable limits for concurrent connections

### CLI & API GROUNDING:

#### **Amazon API Gateway (CLI):**
```sh
aws apigateway create-rest-api --name "MyAPI"
aws apigateway put-method --rest-api-id <id> --resource-id <rid> --http-method GET
```

#### **Application Load Balancer (CLI):**
```sh
aws elbv2 create-load-balancer --name MyALB --subnets subnet-1234567890abcdef0 subnet-0987654321fedcba0
aws elbv2 register-targets --target-group-arn <arn> --targets Id=i-01234567890abcdef0,Id=i-0987654321fedcba0
```

### PRICING & TCO:

#### **Amazon [[api-gateway|API Gateway]]:**
- Pay per request and data transfer.
- Use savings plans for [[00_Master_Links_and_Intro|cost optimization]].

#### **Application Load Balancer:**
- Charged based on hours used and data processed.
- Savings plans and [[00_Master_Links_and_Intro|reserved instances]] can reduce costs.

### COMPETITOR COMPARISON:

#### **[[api-gateway|API Gateway]] vs. [[ALB]]:**
- **Use Case:** [[api-gateway|API Gateway]] is best for microservices/API-driven architectures, while [[ALB]] is suited for load balancing web traffic to EC2/ECS targets.
- **[[appsync|Security]]:** Both support SSL/TLS termination but [[api-gateway|API Gateway]] offers more granular [[appsync|security]] options like [[lambda]] authorizers.

### CONTEXTUALIZATION WITHIN THE AWS ECOYSTEM:

#### **[[api-gateway|API Gateway]]:**
- Integrates with [[cloudwatch]] for [[vpc-flow-logs|logging]] and monitoring.
- Can trigger [[00_Master_Links_and_Intro|Lambda functions]] directly from API requests.
- Often used in serverless architectures alongside [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]], [[dynamodb]], etc.

#### **Application Load Balancer:**
- Works seamlessly within VPCs.
- Commonly paired with Auto Scaling Groups to manage [[00_Master_Links_and_Intro|EC2]] instances.
- Integrates with [[cloudwatch]] for detailed metrics and monitoring.

### REAL-WORLD USE CASES:

#### **[[api-gateway|API Gateway]]:**
- Microservices architecture in a serverless environment using [[00_Master_Links_and_Intro|Lambda functions]].
- Secure API endpoints with custom authorizers and mutual TLS.

#### **Application Load Balancer:**
- Load balancing web traffic across multiple [[00_Master_Links_and_Intro|EC2]] instances or [[ecs]] clusters.
- Implementing A/B testing by routing traffic to different target groups based on path patterns.

### ARCHITECTURAL DIAGRAMS:

1. **[[api-gateway|API Gateway]] + [[lambda]] + [[dynamodb]]**:
   - [[api-gateway|API Gateway]] handles HTTP requests and forwards them to a [[lambda]] function.
   - The [[lambda]] function interacts with [[dynamodb]] for data storage and retrieval.

2. **[[ALB]] + [[00_Master_Links_and_Intro|EC2]] Instances**:
   - [[ALB]] distributes traffic across multiple [[00_Master_Links_and_Intro|EC2]] instances in different availability zones.

### EXAM TRAPS AND COMMON MISCONCEPTIONS:

- Misunderstanding the difference between [[api-gateway|API Gateway]]'s [[api-gateway|caching]] vs. [[ALB]]’s [[elb|cross-zone load balancing]].
- Overlooking [[appsync|security]] features specific to each service (e.g., [[lambda]] authorizers for [[api-gateway|API Gateway]], SSL/TLS termination for [[ALB]]).

### DISTRACTOR ANALYSIS:
1. **Scenario:** High RPO/RTO requirements where [[api-gateway|API Gateway]] is chosen over [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and [[lambda]] due to its [[api-gateway|caching]] capabilities.
   - **Explanation:** While [[api-gateway|caching]] can improve performance, it may not address high RPO/RTO needs effectively.

2. **Scenario:** Using [[ALB]] for a microservices architecture in a serverless environment without understanding the need for [[api-gateway|API Gateway]]'s integration features.
   - **Explanation:** Microservices require fine-grained control over routing and [[appsync|security]] which [[api-gateway|API Gateway]] provides better than [[ALB]].

3. **Scenario:** Choosing [[api-gateway|API Gateway]] for load balancing HTTP traffic to multiple [[00_Master_Links_and_Intro|EC2]] instances due to its path-based routing capabilities.
   - **Explanation:** While both services can handle routing, [[ALB]] is designed specifically for distributing traffic across multiple targets efficiently.

4. **Scenario:** Deploying an application with strict [[appsync|security]] [[policies]] using only [[ALB]] without considering the need for [[api-gateway|API Gateway]]'s integration features.
   - **Explanation:** For applications requiring detailed [[appsync|security]] measures like custom authorizers or mutual TLS, [[api-gateway|API Gateway]] offers better capabilities.

5. **Scenario:** Using [[api-gateway|API Gateway]] for load balancing traffic to [[ecs]] clusters due to its integration types support.
   - **Explanation:** While it can be done, [[ALB]] is more efficient in handling complex routing and scaling requirements for [[ecs]] clusters.

### THE "MOST CORRECT" LOGIC:

- **Operational Excellence vs. [[00_Master_Links_and_Intro|Cost Optimization]]:**
  - [[api-gateway|API Gateway]] excels in operational excellence with features like [[lambda]] authorizers and [[api-gateway|caching]].
  - [[ALB]] is more cost-effective for load balancing web traffic across multiple targets, reducing the need for additional [[appsync|security]] layers.

### ENTERPRISE GOVERNANCE:

1. **[[organizations|AWS Organizations]]:** Use SCPs to restrict access to specific services ([[api-gateway|API Gateway]] or [[ALB]]) based on organizational [[policies]].
2. **Service Control [[policies]] (SCPs):** Define [[policies]] that prevent certain actions like creating new APIs in [[api-gateway|API Gateway]] without approval.
3. **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]:** Share resources across accounts securely using [[ram]] for cross-account access to APIs and load balancers.
4. **[[00_Master_Links_and_Intro|CloudTrail]] Auditing:** Monitor all activities related to [[api-gateway|API Gateway]] and [[ALB]] through [[00_Master_Links_and_Intro|CloudTrail]] logs.

### TIER-3 TROUBLESHOOTING:

1. **[[kms]] Key Policy Conflicts in Cross-Account Migrations:**
   - Ensure [[kms]] keys have the correct permissions across accounts for secure data transfer.
2. **Complex Failure Modes:** Troubleshoot issues like unexpected timeouts, 504 [[api-gateway|errors]] by checking [[route53|health checks]] and [[elb|cross-zone load balancing]] settings.

### DECISION MATRIX FORMAT:

| Feature                          | Use Case                                | Best Suited For                       |
|----------------------------------|----------------------------------------|--------------------------------------|
| [[api-gateway|Stages]]                           | Development lifecycle management        | Microservices/API-driven apps         |
| [[api-gateway|Caching]]                         | Improving API performance               | High-traffic APIs                     |
| Throttling and Rate Limiting     | Controlling access                      | Securing high-value APIs              |
| Path-based Routing               | Distributing traffic based on paths     | Web applications with multiple backends|
| Sticky Sessions                  | Session persistence                     | Applications requiring session state  |

### LATEST AWS SERVICE UPDATES (2026):

- **[[api-gateway|API Gateway]]:** Enhanced WebSocket support and monitoring features.
- **[[ALB]]:** Improved SSL/TLS integration, added newer [[appsync|security]] [[policies]].

### EXAM SCENARIOS:

1. **Deploying a microservices architecture with [[api-gateway|API Gateway]] and [[00_Master_Links_and_Intro|Lambda functions]]:**
   - Questions may involve setting up [[api-gateway|stages]], integrating with [[lambda]], and configuring [[api-gateway|caching]].
   
2. **Setting up load balancing across [[00_Master_Links_and_Intro|EC2]] instances using [[ALB]]:**
   - Focus on creating target groups, routing rules, and enabling [[route53|health checks]].

### INTEGRATION POINTS WITH [[00_Master_Links_and_Intro|OTHER AWS SERVICES]]:

- [[api-gateway|API Gateway]] integrates well with [[cloudwatch]] for monitoring and [[vpc-flow-logs|logging]].
- [[ALB]] works seamlessly within [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] environments and is often paired with Auto Scaling Groups for dynamic scaling.

## Related [[vpc-flow-logs|Notes]]
- [[AWS API Gateway]]
- [[Application Load Balancer (ALB)]]
- [[Lambda Functions]]
- [[VPC (Virtual Private Cloud)]]