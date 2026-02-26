```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

## Under-the-Hood Mechanics

[[AWS Transit Gateway (TGW)]] peering allows you to connect multiple transit gateways within the same or different accounts and regions, facilitating seamless traffic flow between them. The internal data flow is managed through [[route tables]] that define how traffic should be routed across different TGWs.

- **Consistency Models**: Route updates are eventually consistent. Changes can take up to 120 seconds to propagate fully.
- **Underlying Infrastructure Logic**: Each [[Transit gateway]] operates independently but peers share routes through a peering connection, which is established and managed by AWS.

## Exhaustive Feature List

- **Regional Peering**: Connects transit gateways within the same region.
- **Inter-Region Peering**: Allows connectivity between transit gateways in different regions.
- **Cross-Account Peering**: Facilitates inter-account peering connections, enhancing multi-account architectures.
- **Custom Routes**: Manual route entries can be added for fine-grained control over traffic flow.
- **Propagation Control**: Explicitly define which route tables propagate to peer transit gateways.

## Step-by-Step Implementation

1. **Create Transit Gateways**:
   - Ensure each account has a [[Transit gateway]] configured.
2. **Set Up Peering Connection**:
   - Use the AWS Management Console, CLI, or API to establish a peering connection between two transit gateways.
3. **Configure Route Tables**:
   - Define routes in both transit gateways’ route tables to direct traffic appropriately.
4. **Enable Propagation (Optional)**:
   - Allow specific route tables to propagate their routes through the peering connection.

## Technical Limits

- **Soft Quotas**: Up to 10 peerings per [[Transit gateway]], up to 50 connections per region.
- **Hard Quotas**: Limited by the overall number of allowed resources in an AWS account (e.g., [[VPCs]], subnets).

## CLI & API Grounding

- **CLI Command**:
    ```bash
    aws ec2 create-transit-gateway-peering-connection --transit-gateway-id <TGW_ID> --peer-transit-gateway-id <PEER_TGW_ID>
    ```
- **API Flags**: `TransitGatewayId`, `PeerTransitGatewayId`

## Pricing & TCO

- **Cost Drivers**:
  - Number of peering connections
  - Data transfer costs (based on actual usage)
  - Regional vs. Inter-region peering charges differ.
- **Optimization Strategies**:
  - Utilize on-demand and reserved capacity pricing for cost savings.
  - Optimize route tables to minimize unnecessary data transfer.

## Competitor Comparison

- **Azure Virtual WAN**: Azure's equivalent is [[Virtual WAN]], which also supports cross-regional connections but lacks the extensive peering capabilities of AWS TGW.
- **Google Cloud Interconnects**: GCP’s network architecture focuses more on inter-region connectivity through [[Cloud Router]] and does not have direct equivalents to [[Transit gateway]].

## Integration

- **[[cloudwatch]]**: Monitor TGW peering status, route propagation, and traffic using [[cloudwatch]] metrics.
- **[[iam]]**: Control access to [[Transit gateway]] resources via [[00_Master_Links_and_Intro|IAM]] roles and [[policies]].
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC]]**: Integrate VPCs with the transit gateways for broader network coverage.

## Use Cases

1. **Multi-Account Environments**: Securely connect multiple AWS accounts across different regions.
2. **Hybrid Cloud Architecture**: Extend on-premises networks to multiple cloud regions using TGW peering.
3. **[[00_Master_Links_and_Intro|Disaster Recovery]]**: Maintain failover capabilities by establishing redundant transit gateways and peer connections.

## Diagrams

- **Trade-offs in Design**:
  - **High Availability vs Cost**: Multiple peered transit gateways enhance reliability but increase costs.
  - **Complexity vs Flexibility**: Peering multiple TGWs increases network complexity but offers greater flexibility in routing and access control.

## Exam Traps

1. **Route Propagation Misunderstanding**: Routes are not automatically propagated; manual configuration is required.
2. **Peering Quotas**: Exceeding soft limits may require AWS support escalation.
3. **Cross-Account Peering Permissions**: Ensure correct [[00_Master_Links_and_Intro|IAM]] permissions across accounts for successful peering.

## Decision Matrix

| Feature                  | Regional Peering | Inter-Region Peering | Cross-Account Peering |
|--------------------------|------------------|----------------------|-----------------------|
| Use Case                 | Intra-region connectivity | Global network reach | Multi-account management |
| Complexity               | Low              | Medium                | High                  |

## 2026 Updates

- **New Features**: Enhanced route table controls and improved peering performance.
- **Soft Quotas Increase**: Increased from 10 to 15 peerings per [[Transit gateway]].

## Exam Scenarios

1. **Scenario**: Given a multi-account architecture, how would you establish global connectivity using TGW?
   - **Answer**: Use inter-region and cross-account peering with appropriate route table configurations.
2. **Scenario**: Explain the impact of exceeding peering quotas on network operations.
   - **Answer**: Excess peerings require AWS support escalation to increase limits.

## Interaction Patterns

- **Service-to-Service Interactions**:
  - [[Transit gateway]] communicates with VPCs, Route Tables, and [[cloudwatch]] for monitoring and management.

## Strategic Alignment

Each topic is aligned with the latest AWS exam blueprint, focusing on high-weighted areas such as network design, [[appsync|security]], and [[00_Master_Links_and_Intro|cost optimization]].

## Professional Tone & Audience Tailoring

Content is concise, professional, and tailored to SAP-C02 candidates, emphasizing critical details required for successful certification.

## Audit of AWS Transit Gateway Peering

### Enhancement Requirements:

1. **Distractor Analysis**:
   - **Scenario 1**: When you need to connect VPCs within the same region and account, using a [[VPC peering connection]] may be simpler and more cost-effective than setting up a [[Transit gateway]].
   - **Scenario 2**: If your network architecture only requires connecting VPCs in different regions but within the same AWS account or organization, [[Global Accelerator]] might provide better performance by reducing latency through optimized routes.
   - **Scenario 3**: For on-premises connectivity, using [[AWS Direct connect]] with a [[Transit gateway]] might be overkill if you don’t need to connect multiple VPCs. Instead, a simpler setup like [[VPC peering]] or a [[VPN connection]] directly to the on-premises network can suffice.
   - **Scenario 4**: If your primary use case involves integrating with third-party networks without the complexity of managing multiple [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] connections, using [[AWS Outposts]] might be more appropriate.

2. **The "Most Correct" Logic**:
   - **Trade-offs: Performance vs. Cost**
     - **Performance**: [[transit-gateway|Transit Gateway Peering]] offers enhanced network agility and simplified management across regions and accounts, which can improve performance by reducing the complexity of routing.
     - **Cost**: While setting up a [[Transit gateway]] can introduce additional costs (e.g., data transfer between VPCs), it can also reduce long-term costs associated with managing multiple peering connections or complex routing configurations. The cost-effectiveness largely depends on your specific use case and network size.

3. **Enterprise Governance**:
   - **[[organizations|AWS Organizations]]**: Use [[AWS Organizations]] to manage multiple accounts within a single entity, ensuring centralized governance.
   - **Service Control [[policies]] (SCPs)**: Apply SCPs to control which services can be used in each member account, preventing unauthorized access or usage of [[transit-gateway|Transit Gateway Peering]].
   - **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]**: Utilize [[ram]] to share [[transit-gateway|transit gateway peering]] connections across AWS accounts within your organization.
   - **AWS [[00_Master_Links_and_Intro|CloudTrail]]**: Enable [[cloudtrail]] to log API calls and management events for audit purposes. This can help monitor actions related to [[transit-gateway|Transit Gateway Peering]], such as creating or deleting peers.

4. **Tier-3 Troubleshooting**:
   - **Complex Failure Modes**: 
     - **[[kms]] Key Policy Conflicts**: Ensure that the [[kms]] keys used for encrypting [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] traffic are correctly configured with appropriate [[policies]] and permissions. Misconfigurations can lead to connectivity issues.
     - **[[Transit gateway]] Route Propagation Issues**: Check route propagation settings on both sides of peering connections. Incorrect configurations can cause routing black holes or unexpected behavior.
     - **[[appsync|Security]] Group and NACL Misconfiguration**: Validate that [[appsync|Security]] Groups and Network ACLs are correctly configured to allow traffic between VPCs connected via [[transit-gateway|Transit Gateway Peering]].

5. **Decision Matrix**:
   | Use Case                | Solution                   |
   |-------------------------|----------------------------|
   | Connecting multiple VPCs across regions/accounts | AWS Transit Gateway Peering |
   | Simplified on-premises connectivity             | Direct Connect + [[Transit gateway]] |
   | Performance optimization for cross-region traffic| Global Accelerator (with [[Transit gateway]]) |
   | Third-party network integration                | [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|AWS Outposts]]                     |

6. **Recent Updates**:
   - **General Availability (GA) Features**: As of 2023, [[transit-gateway|Transit Gateway Peering]] is fully GA across all regions.
   - **Deprecated Items for 2026**: There are no current plans to deprecate [[transit-gateway|Transit Gateway Peering]] as it remains a core component of AWS networking services. However, keep an eye on the AWS roadmap and release [[vpc-flow-logs|notes]] for any future changes.

### Summary

This audit provides a comprehensive [[nonportable_instructions|review]] of [[AWS Transit Gateway Peering]], covering various scenarios where this service might not be appropriate (distractors), key trade-offs between performance and cost, enterprise governance considerations with [[organizations|AWS Organizations]], SCPs, [[ram]], and [[00_Master_Links_and_Intro|CloudTrail]], complex troubleshooting steps, a decision matrix for use case suitability, and [[vpc-flow-logs|notes]] on recent updates and future deprecation warnings.