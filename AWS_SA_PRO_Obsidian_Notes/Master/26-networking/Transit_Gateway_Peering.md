### Advanced Architecture

At its core, [[transit-gateway|Transit Gateway Peering]] enables the connection of two Amazon Web Services (AWS) Transit Gateways within the same region. This is achieved by creating a peering connection between them, allowing traffic to flow directly from one [[Transit gateway]] to another without using public internet connectivity or [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections.

Internally, [[transit-gateway|Transit Gateway Peering]] leverages the following components:

- **Peer Table**: A table maintained by each [[Transit gateway]] that stores information about active peerings.
- **Association State**: The current state of the peering connection, which can be either 'associating', 'associated', 'disassociating', or 'disassociated'.
- **Propagation State**: Indicates whether routes have been propagated to the [[Transit gateway]] route tables.

When operating at a global scale, [[transit-gateway|Transit Gateway Peering]] supports up to 125 transit gateways per region. However, it does not support inter-region peering or cross-region connectivity. For such requirements, alternative solutions like Global Accelerator, [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering, or Direct Connect should be considered.

### Comparison & Anti-Patterns

| Service                   | [[transit-gateway|Transit Gateway Peering]]          | [[Srinivas_Notes/VPC|VPC]] Peering               | [[Srinivas_Notes/VPN|VPN]] Connection         | Direct Connect       |
|----------------------------|----------------------------------|---------------------------|-----------------------|---------------------|
| Maximum Number of Peers    | 125 per region                   | Point-to-point            | Star or Spoke         | N/A                |
| Region Limit              | Within region only                | Within region only         | Supports multiple      | Supports multiple    |
| Performance               | Fastest path routing             | Limited bandwidth        | Depends on [[Srinivas_Notes/VPN|VPN]] device  | High throughput     |
| [[appsync|Security]]                  | IPSec encryption                | Isolated VPCs             | Encryption required   | Dedicated physical   |
| Cost                      | Low                             | Low                       | Moderate              | Highest             |

Common anti-patterns include attempting to use [[transit-gateway|Transit Gateway Peering]] as an inter-region solution, using it in place of [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering when maximum bandwidth is required, and relying on it for Direct Connect offloads.

### [[appsync|Security]] & Governance

To ensure proper [[appsync|security]] and governance, you can implement the following complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "tgw-gateway:CreatePeeringConnection"
      ],
      "Resource": [
        "*"
      ]
    },
    {
      "Effect": "Deny",
      "Action": [
        "tgw-gateway:CreatePeeringConnection"
      ],
      "Resource": [
        "*"
      ],
      "Condition": {
        "StringNotEqualsIfExists": {
          "aws:PrincipalArn": [
            "arn:aws:iam::*:role/MyTrustedRole"
          ]
        }
      }
    }
  ]
}
```

Cross-account access can be managed using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions. Additionally, Organization Service Control [[policies]] (SCPs) can enforce restrictions on [[Transit gateway]] creation and management across accounts.

### Performance & Reliability

Throttling limits vary based on the action performed. For example, CreatePeeringConnection has a limit of 5 requests per second. To mitigate throttling issues, implement exponential backoff strategies with jitter.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve deploying redundant Transit Gateways in separate Availability Zones and using Route Tables to control traffic flows. In case of failure, traffic will automatically switch over to the redundant [[Transit gateway]].

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be achieved by analyzing usage patterns and applying appropriate [[Transit gateway]] pricing models, such as pay-per-hour or pay-per-connection.

For example, if you require a [[Transit gateway]] for short periods, you can apply the On-Demand pricing model. Alternatively, if your usage remains constant, you may opt for the Pay-per-hour pricing model.

### Professional Exam Scenarios

#### Scenario 1: Multi-Account Strategy

Suppose you need to create a network infrastructure connecting multiple AWS accounts within the same region. Each account contains several VPCs, and there is no requirement for inter-region connectivity. Which solution would best meet these requirements?

*Correct Answer*: [[transit-gateway|Transit Gateway Peering]]

Justification: [[transit-gateway|Transit Gateway Peering]] allows for connecting multiple Transit Gateways within the same region, providing a scalable and efficient networking solution. It ensures high performance and low latency while simplifying management compared to other options.

Incorrect Answers:

* [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering: While capable of connecting individual VPCs, [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering lacks the ability to manage multiple VPCs efficiently as the number of required connections grows exponentially.
* [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] Connections: Although suitable for remote users and branch offices, [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] Connections do not provide the necessary efficiency and ease of management for connecting multiple VPCs within the same region.

#### Scenario 2: [[appsync|Security]] & Governance

As part of a company's compliance policy, all [[Transit gateway]] creations must be approved by the central IT team. Design a solution that meets this requirement while minimizing manual intervention.

*Correct Answer*: Implement an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role (e.g., MyTrustedRole) with the required permissions and restrict the creation of Transit Gateways to specific resources via Deny statements in [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]].

Justification: By creating a trusted [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role, the central IT team maintains control over [[Transit gateway]] creation while reducing manual intervention. Using Deny statements in [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] enforces the restriction of [[Transit gateway]] creation to specific resources.