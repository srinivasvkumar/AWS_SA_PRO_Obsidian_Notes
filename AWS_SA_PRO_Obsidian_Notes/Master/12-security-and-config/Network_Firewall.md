## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, Network Firewall is a stateful firewall service that operates at Layer 3 and L4 of the OSI model, providing packet inspection, filtering, and mitigation capabilities. It is built upon AWS's highly available and distributed infrastructure, offering low-latency connections and high throughput.

### [[RDS_Instance_Types|Global Scale Considerations]]

Network Firewall can be deployed in multiple Regions and accounts, allowing you to create centralized [[appsync|security]] [[policies]] that span your entire organization. This enables consistent enforcement of rules across workloads without requiring manual intervention or custom scripts. To achieve this, you should use [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs) to enforce the use of Network Firewall in specified Organizational Units (OUs) or accounts.

### Under the Hood Mechanics

Network Firewall uses three main components: Firewall Manager, Firewall Endpoints, and Stateful Engine.

- **Firewall Manager**: A service that coordinates the deployment and management of Firewall rules across accounts and applications. It allows you to define rules centrally and apply them consistently throughout your environment.
- **Firewall Endpoints**: Logical entities representing one or more subnets within an Availability Zone (AZ). They act as the entry point for traffic inspected by the Network Firewall.
- **Stateful Engine**: The underlying technology responsible for inspecting packets and enforcing rules based on predefined criteria. It maintains context about the connection state, enabling advanced features like session tracking and flow normalization.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

The following table compares Network Firewall with alternative services and highlights common anti-patterns:

| Service               | Use Cases                                                         | Anti-Patterns                                                   |
| --------------------- | --------------------------------------------------------------- | -------------------------------------------------------------- |
| Network Firewall      | Centralized network [[appsync|security]] for workloads in VPCs              | Implementing perimeter-based [[appsync|security]] instead of zero trust   |
| [[appsync|Security]] Groups       | Stateless, fine-grained access control for [[ec2]] instances        | Overuse of [[appsync|security]] groups leading to complexity and misconfiguration |
| Network Access Control Lists (NACLs) | Basic [[appsync|security]] layer for VPCs                                     | Inadequate maintenance and documentation                          |
| Application Load Balancer [[appsync|Security]] Groups | Securing ingress traffic at the load balancer level            | Ignoring egress traffic restrictions                             |
| AWS WAF               | Protecting HTTP/HTTPS resources from web exploits              | Applying WAF rules directly to all traffic sources (e.g., IP addresses) |

## [[RDS_Instance_Types|3. Security & Governance]]

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for Network Firewall involve granting permissions to manage and modify rules, endpoints, and other components. Here's an example JSON policy snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "network-firewall:DescribeFirewall",
              "network-firewall:DescribeFirewallPolicy",
              "network-firewall:CreateFirewall",
              "network-firewall:DeleteFirewall",
              "network-firewall:UpdateFirewall"
            ],
            "Resource": "*"
        }
    ]
}
```

Cross-account access can be established using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[sts]] AssumeRole calls. For example, a central [[appsync|security]] team could have a role with permissions to manage Network Firewall settings, while individual application teams would have read-only access.

Organization SCPs can enforce the usage of Network Firewall by setting a deny rule for creating [[appsync|Security]] Groups or NACLs without specific tags indicating approval from the central [[appsync|security]] team.

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits for Network Firewall depend on the type and number of rules applied. For instance, stateless rules have a limit of 2000 rules per Firewall endpoint, while Suricata-based rules have a limit of 500 rules per Firewall endpoint.

Exponential backoff strategies should be employed when encountering throttled requests. An example implementation might include increasing the wait time between retries exponentially up to a maximum value before giving up or notifying an administrator.

HA/DR patterns involve deploying Firewall endpoints across multiple AZs and distributing rules evenly among them. Additionally, maintaining standby Firewalls in separate Regions ensures minimal downtime during failover scenarios.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls for Network Firewall involve monitoring usage metrics such as number of rules, data processed, and throughput. By analyzing these costs over time, you can optimize them by removing unnecessary rules, adjusting throughput caps, or utilizing reserved capacity pricing.

For instance, if you notice that a particular set of rules is consuming most of your budget, it may be worth evaluating whether those rules are still relevant or if they can be rewritten more efficiently.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario 1: Multi-Account [[appsync|Security]]

Suppose you need to implement a centralized [[appsync|security]] strategy for a large enterprise spanning multiple AWS accounts. How would you leverage Network Firewall to meet this requirement?

Correct Answer: Use Firewall Manager to create a Firewall policy and associate it with one or more Firewall endpoints in each account. Then, utilize [[organizations|AWS Organizations]] and SCPs to enforce the use of Firewall Manager across designated organizational units (OUs) or accounts.

Incorrect Answer: Create a single Firewall endpoint shared across all VPCs in different accounts. This approach lacks scalability, maintainability, and isolation between accounts.

### Scenario 2: Cost Management

Imagine your organization has strict cost constraints but requires advanced network [[appsync|security]] features provided by Network Firewall. What steps can you take to minimize costs while ensuring adequate protection?

Correct Answer: Monitor usage metrics regularly, remove unnecessary rules, and adjust throughput caps as needed. Utilize reserved capacity pricing when possible and ensure efficient rule writing practices.

Incorrect Answer: Disable Network Firewall whenever it's not needed. While this might reduce costs temporarily, it also introduces significant [[appsync|security]] risks by leaving your network exposed.