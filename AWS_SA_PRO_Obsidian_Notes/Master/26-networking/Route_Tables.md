### Advanced Architecture

At its core, Route Tables are central components of Amazon's Virtual Private Cloud ([[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]) services that determine how network traffic is directed within and in/out of VPCs. They consist of rules, known as routes, which specify destination CIDR blocks and corresponding targets such as NAT Gateways, Internet Gateways, or peered VPCs.

On a global scale, AWS uses a technique called "global acceleration" to improve connectivity between users and resources spread across different regions. This works by routing traffic through edge locations closer to end-users, thereby reducing latency and improving performance. Global Accelerator creates two static IP addresses that map to regional endpoints, while route tables are programmed to direct traffic optimally based on factors like geographical proximity and application [[route53|health checks]].

![Mermaid diagram illustrating Global Accelerator integration with Route Tables](https://mermaid-js.github.io/mermaid-live-editor/#/bYXqzgEBCmAgJ7AxhAbCDAKANwBpBAWAZxgBoUlGUAxigCmqap1FjAmlAIWQDciBuQGvgxsouCjqgGhoQaTKwsM7mDyvExrDFtgAjaWlgC5nCcGUKoUIpq6hDa1shCAkgawpCBfKCFtKCukuLi86IjJLJygnXVsnQKwnXVs9QHNzZCg0REFSQVQrAC0sSSoriS5fiSvpCS0oiS0FnTkMpIC0tLSNOLS0tLDAsMAkgbgRoCH1BLAkrCq4uLi87JTshCnJwiCjqbiAhCqyr)

### Comparison & Anti-Patterns

| Service          | Use Case                                                              |
| ---------------- | -------------------------------------------------------------------- |
| Route Tables    | Managing inter-subnet communication within a single [[Srinivas_Notes/VPC|VPC]]            |
| [[Srinivas_Notes/VPC|VPC]] Peering      | Connecting multiple VPCs together for resource sharing             |
| [[Transit gateway]] | Interconnecting multiple VPCs and on-premises networks at scale   |
| Direct Connect  | Dedicated high-bandwidth connections from enterprise data centers     |

Anti-pattern: Using Route Tables when you should be using [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering or [[Transit gateway]] can lead to complexity, poor [[appsync|security]] management, and potential performance bottlenecks.

### [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can leverage route table associations to control access to specific subnets. For instance, deny certain [[ec2]] instances from communicating with each other by denying INTERNET_GATEWAY access.

```json
{
    "Effect": "Deny",
    "Action": [
        "route:ModifyRouteTable"
    ],
    "Resource": "arn:aws:route:*::route-table/*",
    "Condition": {
        "StringEqualsIgnoreCase": {
            "aws:SourceVpc": "vpc-xxxxxxxxxxxxx"
        },
        "ArnLike": {
            "aws:PrincipalARN": "*EC2*"
        },
        "Bool": {
            "aws:VpcPeeringConnectionExists": "false"
        }
    }
}
```

Cross-account access can also be established via Route Table manipulation. However, it requires manual intervention due to AWS [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] on automating these operations.

Organization Service Control [[policies]] (SCPs) can enforce restrictions on route table configurations, ensuring consistent compliance across accounts.

### Performance & Reliability

Throttling limits exist for modify route table operations (~100 requests per second). Batch modifications help mitigate this issue but may still cause issues if not managed properly. Implementing exponential backoff strategies during throttled [[cloudformation|conditions]] helps maintain system availability.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns often involve multiple route tables and associated subnets spanning various Availability Zones. This setup allows automatic failover and load balancing of network traffic during outages.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls include tagging route tables and their associated resources to monitor usage accurately. Furthermore, rightsizing your resources (e.g., choosing smaller instance types) reduces egress costs associated with data transfer.

Calculation example:
Suppose you have two Route Tables—one serving as a primary connection to an Internet Gateway and another connected to a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]] for private instances. If your monthly data transfer cost is $0.10/GB for internet data and $0.01/GB for AWS data transfer, then understanding the traffic distribution between these two route tables will allow you to calculate the exact cost.

### Professional Exam Scenario

Scenario 1: A company has three VPCs distributed globally—two in US East (N. Virginia) and one in Europe (Ireland). They want to establish secure communication between them without exposing resources publicly. Which services should they use?

Correct Answer: [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering or [[Transit gateway]]
Justification: Both services enable private connectivity between VPCs without exposing resources publicly. The choice depends on the number of VPCs involved and scalability requirements.

Incorrect Answer: Route Tables
Justification: Route Tables cannot directly connect different VPCs; they manage routing within a single [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

Scenario 2: An organization wants to restrict developers from modifying route tables associated with production environments. How would you implement this using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and SCPs?

Correct Answer: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy denying modify route table actions on specific route tables and associate it with relevant user groups/roles. Additionally, apply an [[SCP]] at the organizational level disallowing modify route table actions.

Justification: While the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy prevents unauthorized changes by individual users, the [[SCP]] ensures no other means of bypassing the restriction exists within the specified AWS account(s).