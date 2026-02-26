**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] Advanced Architecture**

[[ALB]] is a Layer 7 load balancer that supports HTTP and HTTPS protocols. It can handle multiple requests in a single connection, using HTTP/2 or WebSocket protocol. [[ALB]] offers advanced features like content-based routing, header-based routing, host-based routing, and path-based routing.

[[ALB]] has a scalable architecture that uses a distributed set of servers called *listeners* to route incoming traffic to one or more target groups based on rules defined in listener rules. Listeners receive and process incoming connections, and forward them to registered targets in target groups.

[[ALB]] supports [[elb|cross-zone load balancing]] by default, allowing it to distribute incoming application traffic across multiple Availability Zones within a region. This improves application availability and fault tolerance.

[[ALB]] operates at the request level, allowing it to balance HTTP(S) requests even when the client only opens a single TCP connection. It also enables advanced request processing capabilities such as request manipulation, tagging, and [[vpc-flow-logs|logging]].

[[ALB]] supports global acceleration through Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] Endpoints, which allows you to create a globally available endpoint for your application. The endpoint automatically routes end user requests to the nearest regional [[ALB]], reducing latency and improving performance.

**Comparison & Anti-Patterns**

| Service | When to Use |
| --- | --- |
| [[ALB]] | Supports HTTP/HTTPS, content-based routing, header-based routing, host-based routing, and path-based routing. Suitable for web applications and microservices. |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Network Load Balancer (NLB)]] | Supports TCP, UDP, and TLS protocols. Suitable for load balancing TCP traffic such as HTTP/2 and gRPC traffic. |
| Classic Load Balancer (CLB) | Legacy load balancer. Not recommended for new applications. |

Common anti-patterns include using [[ALB]] for non-HTTP(S) workloads, using CLB instead of [[ALB]], and not configuring [[elb|cross-zone load balancing]].

**[[appsync|Security]] & Governance**

To enable cross-account access, you need to configure an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account and attach an appropriate policy that allows necessary actions. Here's an example JSON policy:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:DescribeTargetGroups",
                "elasticloadbalancing:DescribeListeners",
                "elasticloadbalancing:DescribeRules",
                "elasticloadbalancing:ModifyListener",
                "elasticloadbalancing:RegisterTargets"
            ],
            "Resource": "*"
        }
    ]
}
```
You can enforce [[appsync|security]] and governance [[policies]] using [[organizations|AWS Organizations]] Service Control [[policies]] (SCPs). For example, you can limit the number of ALBs per account using the following [[SCP]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": "elasticloadbalancing:CreateLoadBalancer",
            "Resource": "arn:aws:elasticloadbalancing:*::loadbalancer/*",
            "Condition": {
                "NumericLessThan": {
                    "aws:RequestCount": 5
                }
            }
        }
    ]
}
```
This [[SCP]] denies creating a new [[ALB]] if the account already has five ALBs.

**Performance & Reliability**

[[ALB]] has throttling limits that control the rate of incoming requests. If the rate exceeds the limit, [[ALB]] returns a 429 Too Many Requests error. To avoid this issue, implement exponential backoff strategies that increase the wait time between retries.

HA/DR patterns involve using multiple ALBs in different regions and configuring DNS failover using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]].

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include setting up [[Budgets]] and alarms, monitoring usage trends, and enabling cost-optimized features such as data compression, connection idle timeout, and IP address preservation.

Here's an example calculation for [[ALB]] costs:

Assume you have an [[ALB]] that receives 1 million requests per month, each request is 1 KB, and you pay $0.008 per GB-month for data processing.

Total data processed = 1,000,000 requests \* 1 KB/request \* (1024 \* 1024)/(1024 \* 1024 \* 1024) GB = 1 GB

Monthly cost = 1 GB \* $0.008/GB = $0.008

**Professional Exam Scenarios**

Scenario 1: A company wants to deploy a multi-region web application with low latency and high availability. They want to use [[ALB]] for load balancing and automatic failover. How would you design this architecture?

Correct answer: Create multiple ALBs in different regions, configure DNS failover using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]], and use Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|geolocation routing]] to direct traffic to the closest [[ALB]].

Incorrect answer: Configure a single [[ALB]] and use Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|latency-based routing]] to direct traffic to the closest endpoint. This solution does not provide automatic failover capabilities.

Scenario 2: A company wants to restrict access to their [[ALB]] to specific IP addresses. How would you implement this requirement?

Correct answer: Create a [[appsync|security]] group associated with the [[ALB]] and allow incoming traffic from specified IP addresses. Alternatively, you can create a WAF rule that allows traffic only from specific IP addresses.

Incorrect answer: Implement network ACLs in the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] that allow traffic from specified IP addresses. This solution may block traffic unintentionally due to implicit deny rules.