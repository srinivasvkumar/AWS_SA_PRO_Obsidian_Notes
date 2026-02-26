## [[RDS_Instance_Types|1. Advanced Architecture]]

[[elb|Cross-Zone Load Balancing]] is a feature of the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Network Load Balancer (NLB)]] in AWS. It allows distributing incoming traffic across all healthy enabled Availability Zones (AZs) rather than within a single zone. This increases the reliability and performance of the load balancer by balancing the load evenly among all AZs.

Under the hood, [[ALB]] and [[NLB]] use a hash-based algorithm to distribute incoming requests to one or more registered targets in multiple AZs. The hash function generates a unique identifier for each request based on various factors such as source IP address, destination port, protocol type, etc. Then, the load balancer uses this identifier to determine which target should receive the request.

When it comes to [[RDS_Instance_Types|global scale considerations]], [[elb|Cross-Zone Load Balancing]] becomes crucial for building highly available applications that can span multiple regions. In this scenario, you can use Global Accelerator to improve the performance of your application by routing traffic to the closest regional load balancer. Global Accelerator also provides additional benefits such as DNS failover, static IP addresses, and enhanced [[appsync|security]] features.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service | [[elb|Cross-Zone Load Balancing]] Supported? | Typical Use Cases |
| --- | --- | --- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] | Yes | HTTP/HTTPS traffic, WebSocket connections, containerized workloads |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Network Load Balancer (NLB)]] | Yes | TCP/UDP traffic, large message transfers, low latency requirements |
| Classic Load Balancer (CLB) | No | Legacy applications, migrating from on-premises to cloud |

Anti-patterns include using CLB instead of [[ALB]] or [[NLB]] when [[elb|cross-zone load balancing]] is required. Another common mistake is not enabling [[elb|Cross-Zone Load Balancing]] for [[ALB]] or [[NLB]] when there are multiple enabled AZs.

## [[RDS_Instance_Types|3. Security & Governance]]

To enable [[elb|Cross-Zone Load Balancing]], you need to ensure proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] permissions are granted. Here's an example [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy allowing a user to create and modify load balancers with [[elb|Cross-Zone Load Balancing]] enabled:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:CreateLoadBalancer",
                "elasticloadbalancing:DescribeLoadBalancers",
                "elasticloadbalancing:ModifyLoadBalancerAttributes"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "elasticloadbalancing:CreateAction": [
                        "CreateLoadBalancer",
                        "AddTags"
                    ]
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:SetLoadBalancerListeners",
                "elasticloadbalancing:SetLoadBalancerOptions",
                "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer",
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:RemoveTags"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalving:DeregisterTargets"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "ArnLike": {
                    "aws:SourceVpc": [
                        "*"
                    ]
                }
            }
        }
    ]
}
```

Additionally, you can enforce [[elb|Cross-Zone Load Balancing]] at the organization level using Service Control [[policies]] (SCPs):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "elasticloadbalancing:CreateLoadBalancer"
            ],
            "Resource": [
                "arn:aws:elasticloadbalancing:*::loadbalancer/*"
            ],
            "Condition": {
                "Null": {
                    "elasticloadbalancing:Scheme": "internet-facing"
                },
                "ForAnyValue:False": {
                    "StringEqualsIfExists": {
                        "elasticloadbalancing:CrossZoneLoadBalancing": [
                            "true"
                        ]
                    }
                }
            }
        }
    ]
}
```

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits depend on the type of load balancer used. For [[ALB]], the maximum number of requests per second is 80,000, while for [[NLB]], it is 1,000,000. To handle higher loads, you can implement exponential backoff strategies to avoid overloading the load balancer.

HA/DR patterns typically involve using multiple load balancers in different regions with [[route53]] [[route53|health checks]] and DNS failover. Additionally, using Global Accelerator can help route traffic to the closest regional load balancer, improving performance and reducing downtime.

## [[RDS_Instance_Types|5. Cost Optimization]]

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost optimization]] involves understanding the usage patterns and applying granular cost controls. For instance, you can use [[ALB]] and [[NLB]] Access Logs only when needed to reduce storage costs. Moreover, you can optimize costs by terminating unused instances, auto-scaling groups, and load balancers.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario 1:

Suppose you need to build a highly available web application serving users worldwide with low latency. What architecture would you choose, and how would you ensure high availability and performance?

#### Correct Answer:

Use Global Accelerator with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] and Auto Scaling Groups (ASGs) in multiple regions. Enable [[elb|Cross-Zone Load Balancing]] for [[ALB]] to distribute traffic evenly across all enabled AZs. Configure [[route53]] [[route53|health checks]] and DNS failover to achieve high availability.

#### Incorrect Answers:

- Using Classic Load Balancer without [[elb|Cross-Zone Load Balancing]] would result in uneven distribution of traffic and potential bottlenecks.
- Not using Global Accelerator could lead to suboptimal routing and higher latency for end-users.

### Scenario 2:

As a Solutions Architect, you have been asked to ensure that no one in your organization can create a new load balancer without [[elb|Cross-Zone Load Balancing]] enabled. How would you accomplish this?

#### Correct Answer:

Implement an [[organizations|AWS Organizations]] Service Control Policy ([[SCP]]) denying the creation of load balancers without [[elb|Cross-Zone Load Balancing]] enabled.

#### Incorrect Answers:

- Implementing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] at the account level would not provide centralized control across the entire organization.
- Modifying the default behavior of load balancers through AWS SDK or CLI calls would not prevent users from creating non-compliant resources.