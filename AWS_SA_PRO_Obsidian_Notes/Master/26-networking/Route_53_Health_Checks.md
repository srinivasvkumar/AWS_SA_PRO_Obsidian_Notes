### Advanced Architecture

At its core, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] is a highly available and scalable service that performs active and passive [[route53|health checks]] of your resources, such as website endpoints, IP addresses, or even other [[route53|health checks]]. It operates at a global level, allowing you to create and manage [[route53|health checks]] from multiple geographically distributed locations.

Internally, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] utilize a variety of techniques to ensure high availability and accuracy. For instance, it uses DNS query load balancing and automatic failover to distribute traffic to healthy resources while rerouting traffic from failed resources. Moreover, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] support both HTTP and TCP [[route53|health checks]], enabling you to monitor the health of web servers and non-web services like databases or load balancers.

[[RDS_Instance_Types|Global scale considerations]] involve configuring [[route53|health checks]] across multiple AWS regions and linking them to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]]'s [[route53|hosted zones]]. This ensures that if one region goes down, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] can still direct traffic to healthy resources in another region. In addition, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] utilize Amazon's extensive private network infrastructure for low latency and reliable connectivity between checkers and monitored resources.

### Comparison & Anti-Patterns

| Service | Ideal Use Case |
| --- | --- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] | Monitoring the health of web applications, APIs, and other resources using HTTP and TCP checks. |
| [[cloudwatch|CloudWatch Alarms]] | Triggering alerts based on metric thresholds, such as CPU utilization or request count. |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] [[route53|health checks]] | Monitoring the health of targets registered with an [[ALB]], ensuring they can receive and process requests. |

Anti-patterns include using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] for monitoring application metrics, which should be handled by [[cloudwatch|CloudWatch Alarms]]. Another common mistake is relying solely on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] for critical applications without implementing alternative backup systems.

### [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] revolve around granting permissions to various actions like creating, updating, and deleting [[route53|health checks]]. Here's an example JSON policy snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:CreateHealthCheck",
                "route53:UpdateHealthCheck",
                "route53:DeleteHealthCheck"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Cross-account access is possible through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust relationships, allowing users from one account to perform actions within another. Additionally, [[organizations]] can enforce centralized control over [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] via Service Control [[policies]] (SCPs), restricting or allowing specific actions and resources.

### Performance & Reliability

Throttling limits for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] depend on the type of [[route53|health checks]] used. For instance, the default limit for HTTP and HTTPS [[route53|health checks]] is 500 checks per account, but this can be increased upon request. To avoid exceeding these limits, implement exponential backoff strategies when retrying failed [[route53|health checks]].

HA/DR patterns using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] typically involve configuring multiple [[route53|health checks]] across different AWS regions and linking them to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]]'s routing [[policies]]. By doing so, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] can automatically reroute traffic to healthy resources in alternate regions during outages.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] come in the form of pay-per-use pricing. There are no upfront costs or minimum fees, making it easy to optimize expenses. To calculate costs, consider factors like the number of [[route53|health checks]], frequency of checks, and the regions being utilized.

### Professional Exam Scenarios

**Scenario 1:** A company has two data centers located in New York and London. They want to set up [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] to monitor their internal applications and automatically redirect traffic to the available data center. Which routing policy should they use? Explain your reasoning.

Correct Answer: The company should use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]]'s [[route53|Geolocation routing]] policy, which allows them to route traffic based on the location of the user making the request. Specifically, they should create two [[route53|health checks]]—one for each data center—and then configure the [[route53|Geolocation routing]] policy to direct traffic to the nearest healthy data center.

Incorrect Answers:

* [[route53|Weighted routing]] policy: This policy doesn't take into account the user's location and would not provide the desired automatic failover functionality.
* [[route53|Latency-based routing]] policy: While this policy considers latency, it does not guarantee traffic will be directed to the nearest data center since it only measures latencies between [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] and the resources.

**Scenario 2:** A consulting firm manages multiple clients' AWS accounts and wants to centrally monitor all [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] across those accounts. What combination of AWS services could achieve this goal? Explain your reasoning.

Correct Answer: The consulting firm can leverage [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs) to enable cross-account access for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]]. First, they must create a new organization and invite their client accounts to join. Then, they can create an [[SCP]] that grants the necessary [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]] actions (create, update, delete) and apply it to all accounts within the organization.

Incorrect Answers:

* Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust relationships alone: Although [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust relationships allow cross-account access, they do not provide centralized management capabilities required for monitoring purposes.
* Utilizing AWS Resource Groups: Resource Groups focus on organizing resources based on tags and do not offer the desired monitoring capabilities for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Health Checks]].