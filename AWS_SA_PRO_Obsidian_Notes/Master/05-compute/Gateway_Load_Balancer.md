## Advanced Architecture

A Gateway Load Balancer (GLB) is a regional, Layer 7 load balancer that can route incoming traffic to applications based on custom routing rules. It operates at the edge of the Amazon Web Services (AWS) network and supports transparent origin-request handling, advanced request and response manipulations, and content-based routing.

GLB's [[RDS_Instance_Types|internals]] consist of three main components: listeners, rules, and connection pools. Listeners receive incoming requests from clients and apply the appropriate rule. Rules define how to match and route requests to specific target groups. Connection pools maintain connections to targets and handle [[route53|health checks]].

Global scale can be achieved using GLB in conjunction with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] Resolver and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudMap]]. This allows for cross-region load balancing, automatic discovery of services, and low-latency DNS responses. For example, imagine a scenario where you have two GLBs, one in US East (N. Virginia) and another in Europe (Ireland). A [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] record set can be configured to respond with the IP addresses of both GLBs as an anycast set. Clients connecting to the domain will automatically connect to the closest GLB, providing lower latency and better performance.

## Comparison & Anti-Patterns

| Service | Use Case |
| --- | --- |
| [[ALB]] | Internal load balancing, HTTP/HTTPS protocol support, layer 7 routing |
| [[NLB]] | Low-level TCP/UDP protocol support, high performance, low latency |
| GLB | Global load balancing, multi-region failover, automatic service discovery |

Anti-patterns include using GLB when application-layer processing is not necessary or when dealing with non-HTTP/HTTPS protocols. In these cases, [[ALB]] or [[NLB]] would be more appropriate.

## [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for GLB can be created using JSON. Here's an example policy allowing a user to manage GLB resources within their account:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:AddListenerCertificates",
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:CreateListener",
                "elasticloadbalancing:CreateRule",
                "elasticloadbalancing:DeleteListener",
                "elasticloadbalancing:DeleteRule",
                "elasticloadbalancing:DeregisterTargets",
                "elasticloadbalancing:DescribeListeners",
                "elasticloadbalancing:DescribeRules",
                "elasticloadbalancing:ModifyListener",
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:RemoveListenerCertificates",
                "elasticloadbalancing:SetRuleOrder",
                "elasticloadbalancing:TagResource"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpc": "${aws_vpc.main.id}"
                }
            }
        }
    ]
}
```

Cross-account access can be granted by creating a role in the source account with the necessary permissions and assuming it in the destination account. Alternatively, [[organizations]] can use Service Control [[policies]] (SCPs) to restrict certain actions on GLB resources across all accounts in the organization.

## Performance & Reliability

Throttling limits for GLB are enforced at the account level. The maximum number of GLBs per region is 10, and each GLB can have up to 100 rules. To mitigate throttling issues, implement exponential backoff strategies in your client code.

HA/DR patterns for GLB involve deploying multiple GLBs across different regions and using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] to distribute traffic between them. By default, GLBs perform [[route53|health checks]] on registered targets and remove unhealthy ones from rotation. However, manual monitoring and fallback mechanisms should also be implemented as a best practice.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be applied using AWS [[billing|Cost Explorer]]. Tagging GLB resources and analyzing usage patterns can help identify areas for optimization. For example, if a GLB has underutilized listener certificates, they can be removed to reduce costs.

Calculating the cost of a GLB involves several factors such as data processing fees, data transfer charges, and certificate fees. As a rough estimate, consider allocating $0.025 per GB processed and $0.09 per GB transferred. Keep in mind that these values may vary depending on your specific use case.

## Professional Exam Scenarios

Scenario 1: You need to provide global load balancing for a web application with low-latency requirements. The application consists of two components: a static website hosted on Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and a RESTful API hosted on [[ec2]] instances. Describe the architecture and justification for using GLB in this scenario.

Correct answer: Use GLB with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudMap]] to distribute traffic between a static website hosted on Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and RESTful API hosted on [[ec2]] instances across multiple regions. Justification: GLB provides global load balancing capabilities, allowing clients to connect to the closest endpoint while ensuring low-latency requirements are met.

Incorrect answer: Use [[ALB]] with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudMap]] to distribute traffic between a static website hosted on Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and RESTful API hosted on [[ec2]] instances across multiple regions. Justification: [[ALB]] does not operate at the edge of the AWS network, resulting in higher latency compared to GLB.

Scenario 2: You are tasked with implementing a secure multi-account strategy for managing GLB resources. Describe the steps required to achieve this goal.

Correct answer:

1. Create a centralized [[organizations|AWS Organizations]] account.
2. Define a Service Control Policy ([[SCP]]) to restrict actions on GLB resources.
3. Create roles in member accounts with the necessary GLB permissions.
4. Assume the roles in the centralized account to manage GLB resources.

Justification: Implementing a multi-account strategy for GLB resources ensures fine-grained control over resource management, reduces the risk of misconfiguration, and enables consistent [[appsync|security]] [[policies]] across all accounts.

### Architecture Diagram
![[GWLC gateway load balancer 2.png]]


### Architecture Diagram
![[GWLB gateway load balancer 1.png]]


### Architecture Diagram
![[GWLB gateway load balancer 6.png]]


### Architecture Diagram
![[GWLB gateway load balancer 5.png]]


### Architecture Diagram
![[GWLB gateway load balancer work.png]]
