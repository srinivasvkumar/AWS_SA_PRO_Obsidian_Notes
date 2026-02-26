### Advanced Architecture
At its core, [[network-and-dns-firewall|AWS Network Firewall]] is a stateful firewall that provides centralized management and enforcement of network [[appsync|security]] [[policies]]. It operates at layer 3 and 4, allowing you to allow or deny traffic based on source/destination IP, port, and protocol. Additionally, it supports Suricata-based rules for intrusion detection and prevention (IDPS) at layers 3-7.

Internally, Network Firewall uses dedicated hardware acceleration within the AWS Nitro system for high performance packet filtering. This enables it to handle high throughput requirements while minimizing latency. Moreover, it natively integrates with AWS services such as [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], [[Transit gateway]], and Route Tables, providing seamless connectivity across various architectures.

[[RDS_Instance_Types|Global scale considerations]] include support for Amazon Web Services (AWS) Global Accelerator, which allows you to create a Network Firewall endpoint in multiple regions and distribute incoming traffic among them. The Accelerator automatically selects the optimal region based on factors like latency and capacity, ensuring efficient routing and high availability.

### Comparison & Anti-Patterns

| Service          | Use Case                                                              |
|------------------|----------------------------------------------------------------------|
| Network Firewall | Centralized control over network [[appsync|security]] [[policies]]                |
| [[appsync|Security]] Groups   | Stateful inspection for [[ec2]] instances within a single [[Srinivas_Notes/VPC|VPC]]            |
| Network ACL      | Stateless inspection for entire subnets within a single [[Srinivas_Notes/VPC|VPC]]           |
| ALB/NLB [[appsync|Security]] Groups | Layer 7 load balancer [[appsync|security]] [[policies]]                         |

Anti-patterns often occur when combining different types of firewalls without clear responsibilities. For instance, using both Network Firewall and [[appsync|Security]] Groups concurrently can lead to confusion about rule evaluation order and potential misconfiguration.

### [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve granting permissions to manage Network Firewall resources. Here's an example JSON policy snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "network-firewall:DescribeFirewall",
                "network-firewall:DescribeFirewallPolicy",
                "network-firewall:ListFirewallPolicies",
                "network-firewall:CreateFirewall",
                "network-firewall:DeleteFirewall"
            ],
            "Resource": "*"
        }
    ]
}
```

Cross-account access is achievable by creating resource shares between member accounts in an [[AWS Organization]]. Alternatively, you could use [[organizations|AWS Organizations]] Service Control [[policies]] (SCPs) to enforce specific actions at the organization level.

### Performance & Reliability

Throttling limits depend on the number of rules per firewall policy. As of now, there's no official documentation stating throttling limits or exponential backoff strategies for Network Firewall. However, implementing these strategies would follow standard [[iam|best practices]] for AWS APIs.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns typically involve distributing Network Firewall endpoints across multiple regions via Global Accelerator. In case of a regional outage, traffic will be redirected to another available region.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be implemented by utilizing separate firewall [[policies]] for different environments (dev, test, prod) and applying appropriate tags. This way, you can monitor and optimize costs according to your business needs.

Calculating costs involves understanding the pricing model: $0.10 per hour for each running firewall, plus data processing charges ($0.05 per GB ingress and $0.10 per GB egress). To illustrate, if you run two firewalls for 24 hours in a month with 1TB ingress and 2TB egress traffic, the total cost would be:

(2 * $0.10 * 24 * 30) + (0.05 * 1024^3) + (0.10 * 2048^3) = $14.4 + $500 + $1600 ≈ $2114.4

### Professional Exam Scenarios

**Scenario 1:** Your company has a multi-account setup with strict compliance requirements. They want to centrally manage all their firewall rules but also maintain individual account [[billing]]. How would you achieve this?

*Correct Answer*: Create a central master account holding the Network Firewall resources and share them with other accounts using Resource Shares. Each linked account should have its own [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[policies]] to manage only its relevant resources. Tagging can help track costs separately.

Incorrect Answers:
- Using [[appsync|Security]] Groups instead of Network Firewall: While [[appsync|Security]] Groups provide similar functionality, they lack the advanced features offered by Network Firewall such as IDPS and integration with Global Accelerator.
- Placing all resources in a single account: This approach violates the principle of least privilege and does not offer any isolation benefits.

**Scenario 2:** Your application requires high availability and low latency worldwide. It receives heavy traffic from Europe and Asia, but less from North America. Which solution would meet these requirements while keeping costs minimal?

*Correct Answer*: Implement [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] along with Network Firewall. Place Network Firewall endpoints in Europe and Asia, leveraging Global Accelerator's intelligent traffic routing feature to direct most requests to the closest endpoint.

Incorrect Answers:
- Using only one region: While this reduces complexity, it increases latency for users outside the chosen region and doesn't optimize cost efficiency.
- Implementing Network Firewall in every region: Although this ensures lower latencies globally, it significantly raises operational costs due to unnecessary redundancy.