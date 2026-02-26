## Advanced Architecture

At the core of an [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|AWS Site-to-Site VPN]] is the Virtual Private Gateway (VGW) that interfaces with Customer Gateways (CGW) at each site. The VGW supports both Border Gateway Protocol ([[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]]) and static routing configurations. In [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] mode, the VGW can act as the [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] peer and advertise routes to the CGW. This allows automatic route propagation based on network changes in your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] or [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]].

In a multi-account environment, you should centralize VGW management using [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs). To achieve this, create a dedicated management account and share the VGW with other accounts using AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]]. This way, you maintain a single point of control for networking settings and [[appsync|security]] [[policies]].

When designing global networks, consider the following aspects:

- **Global Accelerator (GA) integration**: GA enables faster internet connectivity by directing traffic to the optimal AWS edge location. By combining GA with Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]], you can build fast and secure hybrid connections.
- **[[transit-gateway|AWS Transit Gateway]] (TGW)**: TGW simplifies network management by acting as a hub for attaching multiple VPCs and [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections. It also supports peering with other TGW instances, allowing you to create highly scalable and efficient network topologies.

## Comparison & Anti-Patterns

| Service | Use cases | Anti-patterns |
| --- | --- | --- |
| Site-to-Site [[Srinivas_Notes/VPN|VPN]] | Global sites, small data transfers, low-cost option | Large data transfers, high-frequency real-time workloads |
| Direct Connect (DX) | High-bandwidth requirements, large data transfers, low-latency applications | Occasional users, non-enterprise environments |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/03-networking/vpn|Client VPN]] | Remote user access, bring-your-own-device scenarios | Server-side encryption, databases, and stateful firewalls |

Common design mistakes include:

- Not accounting for throttling limits when sizing [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections
- Ignoring anti-patterns and choosing the wrong service for specific use cases
- Neglecting performance optimization techniques such as route prioritization and connection failover

## [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] involve granting permissions to manage the VGW, CGW, and related resources. Here's an example JSON policy snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateVpnConnection",
                "ec2:DescribeVpnConnections"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DeleteVpnConnection",
                "ec2:UpdateVpnConnection"
            ],
            "Resource": [
                "arn:aws:ec2:*::vpn-connection/*"
            ]
        }
    ]
}
```

Cross-account access requires proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, trust [[policies]], and permissions to interact with resources from different accounts. For instance, sharing the VGW with another account involves creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the destination account with a trust relationship to the source account.

Organization SCPs enable enforcing centralized [[appsync|security]] [[policies]] across all member accounts. They can be used to restrict or allow actions like managing [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections, configuring routing tables, or modifying [[appsync|security]] groups.

## Performance & Reliability

Throttling limits for Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] include maximum concurrent [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections per VGW (10) and maximum active tunnels per [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connection (3). To mitigate these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], implement exponential backoff strategies in case of connection failures. Additionally, monitor API call rates and usage metrics to prevent potential throttling issues.

HA/DR patterns involve deploying redundant VGWs in different Availability Zones (AZs) and setting up multiple [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections between the primary and secondary sites. This ensures continuous network availability even if one site fails.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls for Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] involve monitoring and adjusting the following factors:

- Data transfer costs: Depending on the amount of data transferred, alternative services like DX may provide more cost-effective options.
- Number of [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections: Ensure you have only the necessary number of connections open at any given time.
- Idle timeout settings: Configure shorter idle timeout values to minimize charges during periods of inactivity.

Calculating the total cost includes estimating the number of [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections, data transfer fees, and any additional charges for associated resources like [[ec2]] instances or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] databases.

## Professional Exam Scenario 1

Suppose a company has two offices connected via a Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]. They want to optimize their network for better performance while maintaining high availability. Which solution best meets their needs?

Correct answer: Implement [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] integrated with the existing Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]. This will improve latency by directing traffic through the nearest AWS edge locations while keeping the current [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] setup intact.

Incorrect answer: Migrate to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]]. While DX offers higher bandwidth and lower latency, it does not provide the same level of flexibility and cost savings as Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] with Global Accelerator.

## Professional Exam Scenario 2

An organization uses Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] to connect its headquarters and remote branch offices. Due to rapid expansion, they need to add new branches frequently. How would you ensure their current architecture remains secure and scalable?

Correct answer: Centralize VGW management using [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs). Create a dedicated management account and share the VGW with other accounts using AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]]. Then, integrate the VGW with [[transit-gateway|AWS Transit Gateway]] (TGW) to support attaching multiple VPCs and [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections.

Incorrect answer: Manage each VGW individually in each account without centralized oversight. This approach lacks scalability and increases the risk of misconfigurations and [[appsync|security]] vulnerabilities.