**Advanced Architecture**

At the core of Global Networks is Amazon's global infrastructure, consisting of Regions, Availability Zones (AZs), and Edge Locations. The primary services involved in Global Networking are Amazon Virtual Private Cloud ([[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]), Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]], [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]], and [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] solutions.

Architecture components include:

- *Transit Gateways*: A network transit hub that interconnects VPCs and non-AWS networks. Transit Gateways can span multiple accounts using AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]] and support peering to other Transit Gateways.
  Mermaid Diagram:
  ```mermaid
  graph LR
    subgraph Account_A
      A[VPC_A] --1. Transit Gateway-- TGW[(TGW)]
    end
    subgraph Account_B
      B[VPC_B] --2. Transit Gateway Peering-- TGW
    end
    subgraph Account_C
      C[VPC_C] --3. RAM Shared Transit Gateway-- TGW
    end
  ```

- *[[direct-connect|Direct Connect Gateways]]*: Allow users to establish private connectivity between their data centers, colocation environments, or edge locations to AWS. Direct Connect supports link aggregation groups (LAGs) for higher bandwidth and resiliency.

- *Global Accelerator*: Improves application performance by leveraging AWS' global network infrastructure. It provides static IP addresses that map to regional endpoints, thus routing user traffic to the nearest optimal location.

- *[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] Resolver*: Enables recursive DNS queries for your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] resources. With Resolver Endpoints, you can forward queries to authoritative DNS servers, including those in your own network.

**Comparison & Anti-Patterns**

| Service               | Use Case                                                              |
| --------------------- | -------------------------------------------------------------------- |
| Global Accelerator   | Improve performance for real-time, interactive applications         |
| [[Transit gateway]]       | Centralized interconnect for VPCs and on-premises networks        |
| [[Srinivas_Notes/VPC|VPC]] Peering           | Interconnect VPCs directly without requiring [[Transit gateway]]     |
| Direct Connect       | High-bandwidth, low-latency connectivity from on-premises networks |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]]             | Scalable, highly available Domain Name System (DNS)                |

Anti-patterns involve misconfigured DNS settings, such as not utilizing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|latency-based routing]] [[policies]], which could result in suboptimal traffic routing. Another anti-pattern is relying solely on [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering when dealing with numerous VPCs, leading to complexity and potential scalability issues.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] may include resource-level permissions for Global Accelerator, enabling granular control over who can manage specific accelerators. Here's an example JSON policy:

```json
{
    "Effect": "Allow",
    "Action": [
        "globalaccelerator:CreateAccelerator",
        "globalaccelerator:DeleteAccelerator"
    ],
    "Resource": "arn:aws:globalaccelerator:::*",
    "Condition": {
        "StringEquals": {
            "globalaccelerator:Owner": "${aws:username}"
        }
    }
}
```

Cross-account access is possible through [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering, [[Transit gateway]] attachments, and [[direct-connect|Direct Connect gateways]]. Additionally, organization Service Control [[policies]] (SCPs) can enforce restrictions on Global Network services for better governance.

**Performance & Reliability**

Throttling limits are enforced at different levels depending on the service. For instance, Global Accelerator has a limit of 100 accelerators per account, while [[Transit gateway]] offers a maximum of 5000 associated customer networks. To handle throttling exceptions, implement exponential backoff strategies, ensuring appropriate retry intervals and avoiding overwhelming APIs.

HA/DR patterns include deploying dual Global Accelerator endpoints across separate Regions, utilizing Multi-Region [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering, and maintaining redundant Direct Connect connections.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls are achievable by applying tags to Global Network resources like Global Accelerator endpoints and [[direct-connect|Direct Connect gateways]]. Calculate costs using the [AWS Pricing Calculator](https://calculator.aws/) and monitor spending via [[billing|Cost Explorer]].

**Professional Exam Scenarios**

Scenario 1:
You are tasked with designing a hybrid cloud architecture for a company with 3 on-premises datacenters and 2 existing VPCs within the same AWS Region. They require high-bandwidth, low-latency connectivity between their data centers and VPCs. What solution should be implemented?

Correct Answer: Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]] with links to each datacenter and VPCs configured with [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] endpoint services to enable secure, private connectivity.

Incorrect Answers:
- Using Global Accelerator would not offer the desired low-latency connectivity since it operates at the application level.
- Establishing [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering would only work if both VPCs were in the same Region, but it does not provide direct connectivity to on-premises networks.

Scenario 2:
A media company requires minimizing latency for its streaming platform accessed by users worldwide. How can they optimize performance?

Correct Answer: Utilize [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] to distribute incoming traffic across multiple edge locations closer to end-users.

Incorrect Answers:
- Relying on [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering alone would not suffice because it only interconnects VPCs and does not route user traffic.
- Implementing Direct Connect would improve connectivity between on-premises networks and AWS, but it does not optimize application performance for global users.