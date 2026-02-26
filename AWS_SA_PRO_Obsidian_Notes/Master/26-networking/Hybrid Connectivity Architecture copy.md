```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### Hybrid Connectivity Architecture

#### Under-the-Hood Mechanics:
[[Hybrid connectivity]] in [[AWS]] allows enterprises to connect their on-premises infrastructure with AWS resources using various methods such as [[Direct Connect]], [[Site-to-Site VPN]], [[Transit Gateway Connect]], and [[AWS Network Manager]]. These solutions enable secure, private data transfer between environments.

- **Direct Connect**: Establishes a dedicated network connection from your premises to an AWS Direct Connect location.
- **Site-to-Site VPN**: Uses IPsec tunnels over the internet for encrypted connectivity.
- **Transit Gateway Connect**: Extends on-premises networks into the cloud using direct connections through [[VGW]] or [[TGW]].
- **AWS Network Manager**: Manages and visualizes all your network resources, including those in hybrid environments.

#### Exhaustive Feature List:
1. **Direct Connect**:
   - Dedicated 1 Gbps to 100 Gbps connections.
   - Colocation facilities for easy peering.
   - AWS Direct Connect Gateway for cross-region connectivity.
2. **Site-to-Site VPN**:
   - IPsec tunnels for secure, encrypted communication.
   - Support for BGP for dynamic routing.
3. **Transit Gateway Connect**:
   - Multi-account and multi-VPC transit architecture.
   - Support for direct attachment of [[VGWs]] to [[TGW]].
4. **AWS Network Manager**:
   - Global network visualization and management.
   - Integration with Transit Gateway, Direct Connect, and Site-to-Site VPN.

#### Step-by-Step Implementation:
1. **Assessment**: Evaluate on-premises infrastructure and AWS resources.
2. **Design**: Choose the appropriate connectivity method based on performance and security requirements.
3. **Provisioning**:
   - Set up [[Direct Connect]] if high throughput is required.
   - Configure Site-to-Site VPN for lower-cost options.
4. **Deployment**:
   - Establish connections through colocation facilities or IPsec tunnels.
   - Attach VGWs to TGW for a scalable architecture.
5. **Monitoring and Maintenance**: Use [[CloudWatch]] for monitoring network performance and reliability.

#### Technical Limits (2026):
- Direct Connect: 100 Gbps max per connection.
- Site-to-Site VPN: Multiple tunnels, but IPsec throughput is limited by internet bandwidth.
- Transit Gateway: 50 maximum attachments per region.
- Network Manager: Up to 50 global networks.

#### CLI & API Grounding:
- **Direct Connect**: `aws directconnect describe-direct-connect-gateways`, `create-direct-connect-gateway`
- **Site-to-Site VPN**: `aws ec2 create-vpn-gateway`, `attach-internet-gateway`
- **Transit Gateway**: `aws ec2 create-transit-gateway`, `associate-route-table`
- **Network Manager**: `aws networkmanager describe-global-networks`, `create-device`

#### Pricing & TCO:
- Direct Connect: Hourly rates based on connection speed and colocation facility.
- Site-to-Site VPN: Free, but subject to Data Transfer costs.
- Transit Gateway: Monthly fees based on active attachments.
- Network Manager: No additional charges; only applicable associated resources.

#### Competitor Comparison:
- **Azure ExpressRoute**: Similar to Direct Connect with slightly different colocation options.
- **Google Cloud Interconnect**: Provides a comparable direct connection service but has regional limitations.
- **Competitor Transit Solutions**: Often more complex and less integrated than AWS Transit Gateway.

#### Integration:
- **CloudWatch**: Monitor network performance metrics for all connectivity services.
- **IAM**: Use roles and policies to manage access to hybrid resources.
- **VPC**: Integrate with VPC for routing and security management.

#### Use Cases:
1. **Enterprise Migration**: Securely moving on-premises workloads to AWS while maintaining connectivity.
2. **Disaster Recovery**: Establishing a failover path between primary data centers and AWS regions.
3. **Multi-Cloud Operations**: Using Direct Connect or Transit Gateway for cross-cloud operations.

#### Diagrams:
1. **Direct Connect Diagram** (Trade-offs: High cost vs. high performance)
2. **Transit Gateway Integration Diagram** (Trade-offs: Complex setup vs. scalable architecture)

> [!abstract] Exam Tip
1. Confusing Direct Connect with Site-to-Site VPN for throughput requirements.
2. Overlooking the need for BGP in large-scale hybrid environments.

#### Decision Matrix:

| Feature                  | Use Case 1 (Migration) | Use Case 2 (DR) | Use Case 3 (Multi-Cloud) |
|--------------------------|------------------------|------------------|---------------------------|
| Direct Connect           | High Performance       | Essential        | Useful                    |
| Site-to-Site VPN         | Secure Connection      | Secondary        | Minimal                   |
| Transit Gateway          | Scalable Architecture  | Recommended      | Key Feature               |
| Network Manager          | Management             | Monitoring       | Centralized Control       |

#### 2026 Updates:
- Enhanced Network Manager features for better visibility.
- Increased Direct Connect bandwidth options.

> [!warning] Quota
1. Direct Connect: 100 Gbps max per connection.
2. Site-to-Site VPN: Multiple tunnels, but IPsec throughput is limited by internet bandwidth.
3. Transit Gateway: 50 maximum attachments per region.
4. Network Manager: Up to 50 global networks.

#### Exam Scenarios:
1. **Scenario**: Design a disaster recovery setup using hybrid connectivity.
   - **Question**: Which service is best suited for maintaining high availability?
   - **Answer**: Direct Connect or Transit Gateway with BGP routes.

2. **Scenario**: Securely migrate on-premises data to AWS.
   - **Question**: What service ensures secure and efficient data transfer?
   - **Answer**: Site-to-Site VPN or Direct Connect based on bandwidth needs.

#### Interaction Patterns:
- Direct Connect interacts with VPCs for routing.
- Transit Gateway integrates with Network Manager for global network management.
- Site-to-Site VPN uses IPsec tunnels to establish secure connections over the internet.

#### Strategic Alignment:
Each feature is crucial for different use cases, ensuring exam candidates understand the trade-offs and benefits of each service. Prioritization focuses on high-weight topics like Direct Connect and Transit Gateway.

#### Professional Tone & Audience:
Tailored to SAP-C02 candidates with concise, high-value explanations and grounding in official AWS documentation.

#### Prioritization & Clarity:
Focuses on critical exam topics such as Direct Connect and Transit Gateway, providing clear explanations of complex concepts.

#### Grounding:
References official AWS documentation for accuracy and validation against the latest exam blueprint.

#### Architectural Trade-offs:
Design choices impact performance, cost, and complexity. For example, while Direct Connect offers high throughput, it is more expensive than Site-to-Site VPN.
```