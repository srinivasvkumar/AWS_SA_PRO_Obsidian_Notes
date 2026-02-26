**[[RDS_Instance_Types|1. Advanced Architecture]]**

Global Accelerator (GA) is a networking service that uses Amazon’s vast global infrastructure to improve application availability and performance. GA provides static anycast IP addresses that distribute traffic across multiple edge locations, ensuring optimal application responsiveness.

Internally, Global Accelerator uses Network Address Translation (NAT) technology to route traffic from clients to the closest available endpoint. This process involves three main components: accelerators, listeners, and endpoints.

- Accelerators: Logical entities that represent one or more static IP addresses. They serve as entry points for client traffic.
- Listeners: Define the port and protocol used by an accelerator. A listener can have one or many endpoints associated with it.
- Endpoints: Point to resources within your own network (Application Load Balancer, Network Load Balancer, [[ec2]] instances, or REST API endpoints).

Global Accelerator operates at Layer 4 (TCP and UDP) of the OSI model, enabling efficient load balancing while preserving client source IP addresses. Moreover, GA automatically detects the health of its endpoints and routes traffic only to healthy ones.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service                      | Use Case                                                              |
| ---------------------------- | -------------------------------------------------------------------- |
| Global Accelerator           | Improve latency, enable active-active deployments                   |
| Amazon [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]            | Content delivery, [[api-gateway|caching]], origin shielding                          |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]]           | Dedicated connectivity between customer networks and AWS             |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/03-networking/vpn|AWS Site-to-Site VPN]]         | Secure communication between remote networks and AWS                    |
| AWS [[Git_hub_notes/certified-aws-solutions-architect-professional-main/03-networking/vpn|Client VPN]]              | Remote user [[Srinivas_Notes/VPN|VPN]] access to AWS and on-premises resources               |
| [[transfer-family|AWS Transfer Family]]          | Managed file transfer services                                       |

Anti-patterns include using GA when [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]], Direct Connect, or [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] would be more appropriate based on specific requirements. For example, if content delivery optimization is needed, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] should be considered instead.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve granting permissions to create, modify, and delete GA resources. Here's an example JSON policy allowing access to Global Accelerator APIs:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "globalaccelerator:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Cross-account access relies on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, delegating necessary permissions to users in different accounts. Organizational Service Control [[policies]] (SCPs) further restrict actions allowed within member accounts.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits depend on the region and the action being performed. For instance, creating an accelerator has a limit of 1 request per second. To handle such [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], implement exponential backoff strategies in your automation code.

High Availability (HA)/Disaster Recovery ([[dr]]) patterns benefit from Global Accelerator's active-active architecture, which ensures continuous availability even when individual endpoints fail.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls involve understanding how GA charges customers. Charges consist of data transfer fees, accelerator fees, and endpoint fees. By monitoring these costs and adjusting configurations accordingly, you can optimize spending. Calculating potential costs requires knowing your application's expected traffic patterns and geographical distribution.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
You need to design a solution that allows real-time collaboration on documents stored in Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] across multiple regions while minimizing latency. Users are located worldwide. Which service(s) should you use?

Correct answer: Global Accelerator with [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] as the origin server.

Incorrect answer: Using Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] might not preserve client IP addresses due to [[api-gateway|caching]].

Scenario 2:
Your company has two AWS accounts: Account A contains production workloads, and Account B hosts development environments. You want developers in Account B to manage Global Accelerator resources in Account A without sharing full access. How can you achieve this?

Correct answer: Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles in Account A to delegate required GA permissions to developers in Account B.

Incorrect answer: Allowing unrestricted access to all GA resources in Account A is too permissive.