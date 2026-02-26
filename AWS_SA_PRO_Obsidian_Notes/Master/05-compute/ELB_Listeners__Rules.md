## Advanced Architecture

At its core, an Elastic Load Balancer ([[elb]]) Listener is a service that checks for connection requests from clients and then routes incoming traffic to one or more registered targets. An [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] listener supports both HTTP and HTTPS protocols, and can route requests to multiple target groups based on rules.

The [[RDS_Instance_Types|internals]] of an [[elb]] listener involve several components such as the load balancer node, the Availability Zone (AZ) coordinator, and the canonical hostname record. The load balancer node receives incoming requests and distributes them across registered targets in different AZs. The AZ coordinator manages the registration and deregistration of targets in each AZ. Finally, the canonical hostname record is used to distribute traffic evenly across all healthy targets within a target group.

[[RDS_Instance_Types|Global scale considerations]] include using Network Load Balancers (NLBs) with Application Load Balancers (ALBs) to provide low latency connections between clients and applications running in multiple VPCs across different regions. This configuration uses Global Accelerator to improve performance by routing traffic to the closest [[ALB]] endpoint.

## Comparison & Anti-Patterns

When not to use [[elb]] listeners and rules:

| Service | Use Case |
|---|---|
| Amazon [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] | For content delivery at the edge, e.g., static web content, images, videos, etc. |
| AWS [[Fargate]] | For serverless computing without managing servers, e.g., containerized workloads. |
| Amazon [[api-gateway|API Gateway]] | For building, publishing, maintaining, monitoring, and securing RESTful APIs. |

Common design mistakes include:

* Not configuring [[elb|session stickiness]] for certain use cases, leading to poor user experience due to lost sessions.
* Incorrectly configuring [[appsync|security]] groups for the load balancer nodes, exposing them to unauthorized access.
* Overlooking the importance of [[elb|cross-zone load balancing]], which can lead to uneven distribution of traffic.

## [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for [[elb]] listeners and rules may include:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:CreateListener",
                "elasticloadbalancing:DeleteListener",
                "elasticloadbalancing:DeregisterTargets",
                "elasticloadbalancing:DescribeListeners",
                "elasticloadbalancing:ModifyListener",
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:DescribeRules",
                "elasticloadbalancing:DescribeTargetGroups"
            ],
            "Resource": "arn:aws:elasticloadbalancing:*:*:loadbalancer/*",
            "Condition": {
                "StringEquals": {
                    "elasticloadbalancing:LoadBalancerType": "application"
                }
            }
        }
    ]
}
```
Cross-account access can be achieved through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and resource [[policies]]. An example organization [[SCP]] would be:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": "elasticloadbalancing:*",
            "Resource": "arn:aws:elasticloadbalancing:*::loadbalancer/*",
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:SourceVpc": "${aws_vpc.this.arn}"
                }
            }
        }
    ]
}
```
## Performance & Reliability

Throttling limits for [[elb]] listeners and rules depend on the type of load balancer. For example, an Application Load Balancer has a limit of 1000 rules per listener. To handle throttling limits, exponential backoff strategies can be implemented using AWS SDKs.

HA/DR patterns for [[elb]] listeners and rules involve deploying NLBs and ALBs across multiple AZs and regions. This ensures high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls for [[elb]] listeners and rules involve understanding the pricing model for each load balancer type. For example, [[ALB]] charges are based on the number of active [[ALB]] connections, while [[NLB]] charges are based on the number of new flows. Calculation examples include:

* [[ALB]]: $0.0225 x Number of active connections per hour
* [[NLB]]: $0.008 x Number of new flows per hour

## Professional Exam Scenarios

Scenario 1: A company wants to implement a highly available application architecture that spans two regions. They want to ensure that their load balancer can handle large amounts of traffic. Which load balancer type should they choose?

Correct Answer: NLBs and ALBs should be deployed in both regions to ensure high availability and low latency connections.

Incorrect Answer: Using only ALBs in a single region would result in higher latency and lower availability.

Scenario 2: A developer needs to configure fine-grained access control for their [[elb]] listener and rules. They want to restrict access to specific IP addresses. How can they achieve this?

Correct Answer: By implementing a [[appsync|security]] group rule that allows traffic only from specific IP addresses.

Incorrect Answer: Configuring an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy with a deny statement for all IP addresses except the allowed ones would not work since [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] do not apply to network resources like ELBs.