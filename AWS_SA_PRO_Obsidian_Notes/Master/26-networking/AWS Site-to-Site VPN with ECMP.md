```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### Deep-Dive into [[AWS Site-to-Site VPN]] with [[ECMP]]

#### UNDER-THE-HOOD MECHANICS:
**Internal Data Flow:**
- **[[Site-to-Site VPN]]**: Establishes a secure connection between an on-premises network and VPC.
- **Data Path:** Packets are encrypted using [[IPsec]] or OpenVPN protocols before being sent over the internet to AWS, then decrypted in the VPC.

**Consistency Models:**
- Data consistency is maintained through tunnel endpoints and gateways ensuring that packets follow a consistent route. ECMP (Equal-Cost Multi-Path routing) allows for load balancing across multiple paths with equal cost metrics.

**Underlying Infrastructure Logic:**
- **[[Customer Gateway]]**: Represents the on-premises end of the connection.
- **[[Virtual Private Gateway]]**: Attaches to the VPC and acts as the entry/exit point for traffic.
- **Tunnel Endpoints**: Establishes the IPsec tunnels between customer gateway and virtual private gateway.

#### EXHAUSTIVE FEATURE LIST:
1. **IPSec Tunneling:** Supports IKEv2, AES-256 encryption with SHA-2 authentication.
2. **BGP Integration:** Supports dynamic routing over BGP for better scalability.
3. **Multi-Factor Authentication (MFA):** Adds an extra layer of security to the connection setup.
4. **Health Checks:** Monitors the health and availability of tunnels.
5. **ECMP Support:** Load balances traffic across multiple routes with equal cost metrics, improving reliability and performance.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Create a [[Virtual Private Gateway]]** within your VPC using AWS Management Console or CLI.
2. **Configure a Customer Gateway**: Define the public IP address of your on-premises router and specify the routing protocol (BGP).
3. **Establish Tunnel Connections**: Create tunnels between the virtual private gateway and customer gateway, specifying encryption details.
4. **Enable ECMP**: Ensure both ends support ECMP for load balancing traffic across multiple routes.
5. **Configure Routing Tables**: Update VPC route tables to direct traffic through the tunnels.

#### TECHNICAL LIMITS (2026):
- Maximum of 100 customer gateways per virtual private gateway.
- Up to 3 IPsec tunnels per pair of customer gateway and virtual private gateway.
- ECMP support requires compatible hardware on both ends.

#### CLI & API GROUNDING:
**CLI Commands:**
```sh
aws [[00_Master_Links_and_Intro|ec2]] create-vpn-gateway --type ipsec.1
aws [[00_Master_Links_and_Intro|ec2]] create-customer-gateway --ip-address <IP> --bgp-asn <ASN> --type ipsec.1
aws [[00_Master_Links_and_Intro|ec2]] create-vpn-connection --customer-gateway-id <ID> --vpn-gateway-id <GW_ID>
aws [[00_Master_Links_and_Intro|ec2]] modify-vpn-tunnel-options --vpn-connection-id <VPN_ID> --inside-ip-address-type static
```

**API References:**
- `CreateVpnGateway`
- `CreateCustomerGateway`
- `CreateVpnConnection`
- `ModifyVpnTunnelOptions`

#### PRICING & TCO:
- **Pricing Model**: Usage-based pricing, typically $0.05 per gigabyte of data transferred.
- **Optimization Strategies**:
  - Use ECMP to distribute traffic and avoid single point failures.
  - Enable health checks for proactive monitoring and failover.

#### COMPETITOR COMPARISON:
- **[[Azure Virtual WAN]] vs [[AWS Site-to-Site VPN]]**: Azure offers more centralized management but lacks the extensive ECMP support found in AWS.
- **Google Cloud Interconnect**: Requires dedicated hardware, whereas AWS is software-defined with flexible routing options.

#### INTEGRATION:
- **CloudWatch Integration**: Use [[cloudwatch]] to monitor health checks and set alarms for tunnel issues.
- **IAM Integration**: Use IAM roles to manage access to the Site-to-Site VPN resources.
- **VPC Integration**: Route tables within VPCs can direct traffic through the tunnels.

#### USE CASES:
1. **Hybrid Cloud Environments**: Connect on-premises systems with AWS VPC for seamless application integration.
2. **Disaster Recovery**: Use multiple ECMP routes for high availability and failover during outages.
3. **Data Synchronization**: Maintain real-time data sync between on-premises databases and RDS instances in AWS.

#### DIAGRAMS:
- Placeholder: Diagram showing the setup of a [[Site-to-Site VPN]] with multiple tunnels using ECMP load balancing.

#### EXAM TRAPS:
1. Common misconception about number of allowed customer gateways per VPC.
2. Understanding the difference between BGP and static routing.
3. Overlooking the need for compatible hardware to support ECMP.

> [!abstract] Exam Tip
1. **Scenario:** Setting up a Site-to-Site VPN with BGP routing.
   - **Question:** What are the necessary CLI commands to create and configure this setup?
2. **Scenario:** Implementing ECMP for load balancing in a multi-tunnel configuration.
   - **Question:** How do you ensure that all routes have equal cost metrics?

#### DECISION MATRIX:
| Feature            | Use Case 1 (Hybrid Cloud)    | Use Case 2 (Disaster Recovery)   |
|--------------------|-------------------------------|----------------------------------|
| IPSec Tunneling    | Critical                     | Essential                        |
| BGP Integration    | Recommended                  | Required                         |
| ECMP Support       | Optional but beneficial      | Mandatory for high availability  |

#### 2026 UPDATES:
- Expect further enhancements in ECMP support and increased quotas.
- More integrations with AWS security services like [[Security Hub]].

> [!warning] Quota
- **GA Features (2026):** Look out for any new general availability features related to ECMP or improved support for specific router models.
- **Deprecated Items:** Check the AWS release notes and deprecation schedules to identify any components of Site-to-Site VPN that may be deprecated in 2026. This could include older IPsec protocols or certain tunnel configurations.

#### AUDIT OF AWS SITE-TO-SITE VPN WITH ECMP

> [!danger] Distractor
1. **Scenario 1: Network Latency Sensitive Applications:** While [[AWS Site-to-Site VPN]] supports ECMP (Equal Cost Multi-Path) routing, it may not be the best choice for applications that require ultra-low latency and high performance. Services like [[Direct Connect]] might offer more deterministic low-latency connections.
2. **Scenario 2: Large Volume of Traffic with Complex Routing:** For environments where traffic volume is extremely high and requires complex routing decisions beyond what ECMP offers, dedicated hardware-based solutions or more sophisticated SD-WAN services may be a better fit.

#### DECISION MATRIX
| Use Case                | Solution                      |
|-------------------------|-------------------------------|
| Low Latency Requirements| [[AWS Direct connect]]        |
| High Volume Routing     | SD-WAN Services               |
| Enhanced Encryption      | [[AWS PrivateConnect]]        |
| Multi-Region Redundancy  | [[AWS Transit Gateway]] with DX   |
| Standard Connectivity    | [[AWS Site-to-Site VPN]] with ECMP|
```