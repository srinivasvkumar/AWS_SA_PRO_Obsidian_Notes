## Advanced Architecture

A [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Network Load Balancer (NLB)]] operates at Layer 4 (TCP, UDP) of the OSI model, enabling it to handle millions of requests per second while maintaining minimal latency. It supports clients using both IPv4 and IPv6 protocols. An [[NLB]] can distribute incoming application traffic across multiple [[ec2]] instances, containers, and IP addresses.

### [[RDS_Instance_Types|Internals]]

An [[NLB]] uses **AWS-owned** IP addresses for load balancing, not client IP addresses. This approach ensures that the backend instances see only the [[NLB]]'s IP address as the source of requests. The following Mermaid syntax diagram illustrates an [[NLB]]'s internal workings:

```mermaid
graph LR
A[Client] -- TCP/UDP --> B((NLB))
B -- TCP/UDP --> C[Target Group]
C -- IP --> D[EC2 Instance]
```

### [[RDS_Instance_Types|Global Scale Considerations]]

While an [[NLB]] itself is zonal, you can create Global Accelerator resources in front of your [[NLB]](s) to provide a single anycast endpoint for your users. This setup improves performance by directing user traffic to the nearest regional edge location. Furthermore, Global Accelerator enables automatic failover to healthy endpoints in alternate regions if disruptions occur.

## Comparison & Anti-Patterns

| Feature               | [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]]                   | [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Network Load Balancer (NLB)]]          | Gateway Load Balancer ([[gwlb]])         |
|-----------------------|-----------------------------------------------------|--------------------------------------|--------------------------------------|
| Listener Protocol     | HTTP/2, WebSocket                                  | TCP, UDP                             | Only GENEVE                           |
| Use Case              | L7 Routing, Sticky Sessions, HTTP Headers            | High Throughput, Low Latency        | Genev-enabled [[Srinivas_Notes/VPC|VPC]] Endpoint Services |
| Connection Persistence | Cookie-based, Path-based                             | Client IP-based                      | Not Supported                       |
| Internal or Internet Facing | Both                                              | Both                                 | Only Internet Facing                 |

Anti-patterns include:

- Using an [[NLB]] when Layer 7 functionality like URL path–based routing, header manipulation, or SSL termination is required.
- Expecting [[elb|session stickiness]] using cookies or headers.

## [[appsync|Security]] & Governance

To secure access to your [[NLB]], follow these guidelines:

1. Implement strict [[appsync|security]] group rules allowing only necessary ingress/egress traffic.
2. Enable AWS WAF and [[shield]] advanced protection on ALBs to filter unwanted traffic.
3. Create dedicated [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles for each environment, restricting their permissions to specific actions and resources. Example policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:Describe*",
                "elasticloadbalancing:CreateListener",
                "elasticloadbalancing:DeleteListener",
                "elasticloadbalancing:CreateRules",
                "elasticloadbalancing:DeleteRule",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:RevokeSecurityGroupIngress"
            ],
            "Resource": "*"
        }
    ]
}
```

4. Use Service Control [[policies]] (SCPs) within [[organizations|AWS Organizations]] to enforce centralized control over account-level activities. For instance, deny deletion of certain types of load balancers:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyDeleteNLB",
            "Effect": "Deny",
            "Action": "elasticloadbalancing:DeleteLoadBalancer",
            "Resource": "arn:aws:elasticloadbalancing:*::loadbalancer/*",
            "Condition": {
                "StringEqualsIgnoreCase": {
                    "elasticloadbalancing:Type": [
                        "network",
                        "network-ip"
                    ]
                }
            }
        }
    ]
}
```

## Performance & Reliability

NLBs have no specified throttling limits but may experience increased latencies under extreme load due to AWS infrastructure [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]]. To mitigate such issues, implement exponential backoff strategies during transient [[api-gateway|errors]].

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes, deploy NLBs across different Availability Zones and ensure proper target registration and deregistration procedures.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls for NLBs include:

- Tagging resources to monitor costs by project, team, or environment.
- Setting up [[billing]] alarms to receive notifications when usage exceeds predefined thresholds.
- Enforcing resource deletion [[policies]] via Service Quotas or [[config|AWS Config]] to avoid unnecessary charges.

## Professional Exam Scenarios

### Question 1

Suppose you need to build a solution that handles 10 million requests per second with sub-10 ms latency requirements. Which load balancer type would be most appropriate? Why?

Correct Answer: A [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Network Load Balancer (NLB)]] should be used because it operates at Layer 4 (TCP, UDP) and can handle millions of requests per second with minimal latency.

Incorrect Answer: An [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] is not suitable since it operates at Layers 7 (HTTP, HTTPS) and introduces additional overhead due to its support for features like URL path–based routing, header manipulation, and SSL termination.

### Question 2

You are responsible for securing an existing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Network Load Balancer (NLB)]] in your organization. Currently, there are no restrictions on who can delete load balancers. How do you prevent unauthorized deletion of the [[NLB]] using SCPs?

Correct Answer: By implementing an [[SCP]] that denies the `elasticloadbalancing:DeleteLoadBalancer` action for Network Load Balancers, as shown in the example provided earlier.

Incorrect Answer: Allowing unrestricted deletion of load balancers might lead to unexpected outages and potential [[appsync|security]] risks. Therefore, applying proper SCPs is essential to preventing unauthorized deletion of critical resources like NLBs.