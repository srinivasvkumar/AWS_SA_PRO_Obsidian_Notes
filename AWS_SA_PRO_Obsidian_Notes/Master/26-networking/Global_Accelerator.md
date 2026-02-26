**Advanced Architecture**

Global Accelerator uses a network of Amazon Web Services (AWS) edge locations to improve the performance of internet-facing applications by directing user traffic to the optimal regional endpoint based on application performance, user location, and real-time network [[cloudformation|conditions]]. The service operates at layer 4 (TCP, UDP) and supports both IPv4 and IPv6.

Accelerators, Endpoints, and Listeners form the core components of Global Accelerator architecture. An accelerator is a set of static IP addresses (anycast) that distribute traffic to one or more endpoints in different regions. Endpoints can be Application Load Balancers ([[ALB]]), Network Load Balancers ([[NLB]]), Amazon [[ec2]] instances, or AWS resources with public IP addresses. A listener is an accelerator component that checks for incoming traffic on a specific port and protocol. It can have multiple endpoint groups, allowing traffic distribution between them.

Global Accelerator's "under the hood" mechanics include:

- **Endpoint traffic routing**: Based on [[route53|health checks]], traffic is directed to healthy endpoints within a region. If all endpoints fail in a region, traffic is redirected to another region.
- **Traffic management methods**: Two traffic management methods are available: geolocation (route traffic based on user location) and [[route53|latency-based routing]] (route traffic based on lowest latency).
- **Port mapping**: Map multiple listeners to the same port on different endpoint groups.

**Comparison & Anti-Patterns**

| Service                   | Use Case                                                                                       |
| ------------------------- | --------------------------------------------------------------------------------------------- |
| Global Accelerator        | Improve performance for globally distributed users, support multi-region applications.     |
| Amazon [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]        | Serve static and dynamic content, [[api-gateway|caching]], origin shielding, [[appsync|security]] features.             |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]]       | Dedicated network connectivity between customer infrastructure and AWS services.            |
| [[Srinivas_Notes/VPC|VPC]] Peering, [[Srinivas_Notes/VPN|VPN]], Transit | Hybrid cloud, data center migrations, [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], cross-region connectivity.      |
| AWS [[Git_hub_notes/certified-aws-solutions-architect-professional-main/03-networking/vpn|Client VPN]]           | Secure remote access to resources in your AWS environment.                                |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/03-networking/vpn|AWS Site-to-Site VPN]]      | Establish private connectivity between your on-premises networks and AWS VPCs.          |

Anti-patterns involve using Global Accelerator when other services are better suited. For example, it would not be ideal to use Global Accelerator solely for load balancing purposes when [[ALB]] or [[NLB]] could handle the task efficiently. Also, avoid using Global Accelerator as a replacement for Direct Connect or [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections for hybrid cloud scenarios.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be implemented using JSON statements such as:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "globalaccelerator:CreateAccelerator",
              "globalaccelerator:DescribeAccelerator*, 
               globalaccelerator:DeleteAccelerator"
            ],
            "Resource": "*"
        }
    ]
}
```

Cross-account access requires proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions. To enable Global Accelerator to work across accounts, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account with necessary permissions and assume the role from the destination account.

Organization Service Control [[policies]] (SCPs) can limit the usage of Global Accelerator to prevent unwanted resource creation or modification.

**Performance & Reliability**

Throttling limits depend on the number of accelerators per region, which is currently set at 50. Exponential backoff strategies should be employed during throttled events to mitigate impact.

HA/DR patterns benefit from Global Accelerator due to its ability to route traffic to healthy endpoints in different regions. Maintain redundancy by setting up endpoints in various regions and enabling [[route53|latency-based routing]].

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls include selecting specific [[api-gateway|endpoint types]] and monitoring traffic flow through each accelerator. Calculate costs using the pricing calculator and monitor usage via [[billing|Cost Explorer]].

**Professional Exam Scenarios**

Scenario 1:
You need to implement a highly available, low-latency web application with two identical replicas located in the US East (N. Virginia) and Europe (Ireland) regions. Users from both regions must experience minimal latency while connecting to the application.

Correct answer: Implement Global Accelerator with two endpoints (one for each region) and configure [[route53|latency-based routing]]. This setup ensures that users will be directed to the nearest endpoint based on their location.

Incorrect answer: Using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] alone does not provide multi-region support without additional customizations. While Direct Connect offers reduced latency, it is designed for hybrid cloud scenarios rather than multi-region deployments.

Scenario 2:
Your company has strict [[appsync|security]] requirements, including controlling access to Global Accelerator resources. You want to ensure only authorized personnel can manage these resources.

Correct answer: Create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] granting permissions to Global Accelerator actions and attach those [[policies]] to relevant [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users, groups, or roles. Additionally, utilize Service Control [[policies]] (SCPs) if you wish to enforce restrictions at the organization level.

Incorrect answer: Relying on resource-based [[policies]] or Access Control Lists (ACLs) provides insufficient granularity and control over who can manage Global Accelerator resources.