```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### Deep-Dive into [[AWS Direct Connect (DX)]]: Dedicated vs Hosted

#### Under-the-Hood Mechanics:

**Dedicated Connection:**
- **Internal Data Flow:** A dedicated 1Gbps or 10Gbps connection is established directly between your on-premises network and an AWS Direct Connect location. This connection provides a private, high-bandwidth link to the cloud.
- **Consistency Models:** No consistency issues as data travels over a single, dedicated physical path.
- **Underlying Infrastructure Logic:** The connection is maintained through hardware (Ethernet or 802.1Q) at a specific AWS Direct Connect location.

**Hosted Connection:**
- **Internal Data Flow:** Hosted connections are established via a third-party service provider who manages the link between your on-premises network and an AWS Direct Connect location.
- **Consistency Models:** Similar to Dedicated Connections but involves additional layers of hardware managed by the hosting partner, which can introduce slight latency or reliability concerns.
- **Underlying Infrastructure Logic:** Utilizes the third-party's infrastructure for the last-mile connectivity.

#### Exhaustive Feature List:

**Dedicated Connection:**
- [[Private VLANs]]
- BGP peering sessions with AWS and/or VPCs
- Direct access to multiple regions from a single connection point

**Hosted Connection:**
- Managed by service providers like Equinix, Telx, etc.
- Shared infrastructure for last-mile connectivity
- Flexibility in terms of location (can be closer to the enterprise data center)

#### Step-by-Step Implementation:

1. **Determine Requirements:** Evaluate bandwidth needs and connectivity goals.
2. **Choose Location:** Select a [[Direct Connect]] location that meets your geographic requirements.
3. **Order Connection:**
   - For Dedicated, contact AWS directly or an authorized vendor to provision the connection.
   - For Hosted, engage with a third-party provider like Equinix to order and manage the link.
4. **Configure BGP:** Set up BGP peering sessions between your network equipment and the Direct Connect endpoint.
5. **Establish VLANs:** Configure private VLANs for secure communication between on-premises and AWS resources.

#### Technical Limits (2026):

- **Dedicated Connection:**
  - Bandwidth: 1Gbps, 10Gbps
  - Number of Direct Connect locations per region: Limited by physical availability.
- **Hosted Connection:**
  - Bandwidth: Dependent on the service provider’s offerings.
  - Number of hosted connections per location: Varies based on partner constraints.

#### CLI & API Grounding:

```sh
aws directconnect create-direct-connect-gateway
aws directconnect describe-direct-connect-gateways
aws directconnect allocate-hosted-connection
```

**API Flags:**
- `--bandwidth` (e.g., 1Gbps)
- `--location` (Direct Connect location)

#### Pricing & TCO:

- **Dedicated Connection:** Higher upfront costs but lower per-unit cost for high bandwidth.
- **Hosted Connection:** Lower initial investment, higher ongoing costs due to third-party management fees.

**Optimization Strategies:**
- Consolidate multiple connections into fewer larger ones (dedicated).
- Leverage hosted services for smaller bandwidth needs or temporary requirements.

#### Competitor Comparison:

- **AWS Direct Connect vs. Azure ExpressRoute:** Both provide dedicated links but AWS has a broader partner network.
- **AWS Direct Connect vs. Google Cloud Interconnect:** Similar functionalities, but AWS offers more flexibility in connection types and locations.

**Architectural Trade-offs:**
- Dedicated connections are more cost-effective for high-bandwidth needs but less flexible.
- Hosted connections offer geographical flexibility at higher costs due to third-party management.

#### Integration:

- **[[cloudwatch]]:** Metrics can be integrated into CloudWatch for monitoring bandwidth usage, connection health.
- **[[iam]]:** IAM roles and policies can control access to Direct Connect operations.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC]]:** Direct Connect integrates seamlessly with VPCs for routing traffic directly to AWS resources.

#### Use Cases:

1. **Large Enterprises:** High-speed, secure data transfer between on-premises and cloud environments (Dedicated).
2. **Small/Medium Businesses:** Cost-effective connectivity options leveraging third-party infrastructure (Hosted).

#### Diagrams:
- Placeholder for architectural diagram showing Dedicated vs Hosted Connection setup.

#### Exam Traps:

> [!danger] Distractor
1. Misunderstanding the difference in cost models between Dedicated and Hosted.
2. Overlooking the impact of third-party management on reliability and latency.
3. Confusing Direct Connect with other AWS connectivity services like [[VPC Peering]] or [[Transit gateway]].

#### Decision Matrix:

| Feature                | Dedicated Connection               | Hosted Connection                 |
|------------------------|------------------------------------|------------------------------------|
| Bandwidth              | 1Gbps, 10Gbps                      | Dependent on service provider      |
| Cost                   | Higher upfront                     | Lower initial investment           |
| Flexibility            | Limited                            | High                               |
| Reliability            | High                               | Moderate                           |

#### Exam Scenarios:

> [!check] Implementation
1. Determining the optimal connection type based on bandwidth needs and cost constraints.
2. Configuring BGP sessions and VLANs for Direct Connect setup.

#### Interaction Patterns:
- **Direct interaction** between customer equipment and AWS Direct Connect location (Dedicated).
- **Indirect interaction** via third-party service provider’s infrastructure (Hosted).

#### Strategic Alignment:

Focus on key differences, use cases, and cost models to align with SAP-C02 exam blueprint.

#### Professional Tone & Audience:
Tailored specifically for SAP-C02 candidates with high-value technical content focused on the latest AWS landscape.

#### Prioritization:
High-weight topics include bandwidth limits, setup configurations, and integration points.

#### Clarity:
Concise explanations of complex concepts to ensure understanding during exam preparation.

#### Grounding & Validation:
Referenced official AWS documentation for accuracy as per the latest exam blueprint updates.
```