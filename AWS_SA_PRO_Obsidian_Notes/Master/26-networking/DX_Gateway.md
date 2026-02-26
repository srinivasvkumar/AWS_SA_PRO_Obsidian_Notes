**Advanced Architecture**

DX Gateway is a fully managed service that connects on-premises networks to the AWS global network through private virtual interfaces. It supports up to 50 VPCs and enables reusing existing public IP addresses. The DX Gateway [[RDS_Instance_Types|internals]] include three main components:

- **Data Plane (DP)**: Processes customer traffic.
- **Control Plane (CP)**: Manages configuration and monitors DP health.
- **Management Plane (MP)**: Facilitates API calls, UI interactions, and [[vpc-flow-logs|logging]].

For [[RDS_Instance_Types|global scale considerations]], DX Gateways support multiple active connections in different regions, allowing low-latency access from local VPCs. Under the hood, they use [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] behind the scenes to route traffic efficiently.

**Comparison & Anti-Patterns**

| Issue | [[Srinivas_Notes/VPN|VPN]] | Direct Connect (DX) | [[Transit gateway]] (TGW) | DX Gateway |
|---|---|---|---|---|
| Throughput | Lower (~1.2 Gbps) | Higher (up to 10 Gbps) | Up to 50 Gbps | Up to 50 Gbps |
| Latency | Higher than DX | Lower than [[Srinivas_Notes/VPN|VPN]] | Similar to [[Srinivas_Notes/VPN|VPN]] | Similar to DX |
| Scalability | Limited | Highly scalable | Very high (for TGW) | Highly scalable |
| Multi-region? | Single region | Multiple regions | Single region | Multiple regions |
| Cost | Lower | Higher | Moderate | Highest |

Anti-patterns:

- Using DX Gateway as an alternative to [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] or Direct Connect when lower costs and simpler setup are desired.
- Configuring DX Gateway without considering throttling limits and exponential backoff strategies during heavy load periods.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "dx-gateway:Create"
            ],
            "Resource": "*"
        }
    ]
}
```
Cross-account access:

DX Gateway allows creating private virtual interfaces in other accounts by configuring them as "associated" accounts. This grants permissions to manage resources but not modify their configurations.

Organization Service Control [[policies]] (SCPs):

To enforce centralized control over DX Gateway usage, create SCPs at the organization level. For example, deny specific actions like `dx-gateway:Delete*`, or restrict resource creation to specific tags.

**Performance & Reliability**

Throttling limits:

DX Gateway has a limit of 500 API requests per minute. To handle higher loads, implement exponential backoff strategies using the AWS SDKs.

HA/DR patterns:

Implement redundancy by deploying two DX Gateways in separate availability zones within the same region. Configure backup targets such as backup VPCs or [[dr]] sites.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls:

Monitor DX Gateway usage by enabling [[cloudwatch]] metrics and setting alarms based on bandwidth utilization or API request rates. Implement tagging strategies to track costs across different teams or projects.

Calculation examples:

Assuming a 10 Gbps DX Gateway with 95% utilization for half a month (15 days) and a flat rate of $0.25 per hour:

Total cost = (10 GBPS × 15 days × 24 hours × 0.25) / 1024^3 × 4 (for both DP and CP)
Total cost ≈ $27.34

**Professional Exam Scenarios**

Scenario 1:

An e-commerce company requires a highly available solution with low latency between its on-premises data center and AWS workloads. They have strict compliance requirements and need to ensure proper separation of duties.

Correct answer:

Deploy a DX Gateway with private virtual interfaces in multiple VPCs and enable cross-account access for management purposes. Implement Service Control [[policies]] (SCPs) to enforce compliance requirements and separate duties.

Incorrect answer:

Use a single [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connection since it provides similar latency and lower costs. This approach does not address the compliance requirements and lacks proper separation of duties.

Scenario 2:

A media company needs to transfer large files between its data centers and Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. They require high throughput and reliable data transfers while minimizing costs.

Correct answer:

Implement a Direct Connect (DX) connection instead of DX Gateway due to its lower costs and higher throughput. Use [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] Transfer Acceleration if low latency is still required.

Incorrect answer:

Use DX Gateway because it offers global scale capabilities. However, DX Gateway is not designed for high-throughput file transfers and will be more expensive compared to Direct Connect.