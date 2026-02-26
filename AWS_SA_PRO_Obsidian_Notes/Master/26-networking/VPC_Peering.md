**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering is a networking construct that enables two VPCs to connect at Layer 3, allowing communication using private IP addresses without requiring traffic to traverse the public internet or NAT gateways. It provides a low-latency, secure interconnect between VPCs within the same region or across different regions (Inter-Region [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering) or even across accounts (Multi-Account [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering).

Internally, [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering leverages the AWS infrastructure to exchange routes over multiple connection points via the AWS Global Network. This ensures high bandwidth (up to 50 Gbps in most regions) and low latency (typically less than 1 ms). The under-the-hood mechanics involve configuring route tables, [[appsync|security]] groups, network ACLs, and DNS settings for peered VPCs.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service   | Use Case                                                              |
|-----------|----------------------------------------------------------------------|
| [[Srinivas_Notes/VPC|VPC]] Peering| Securely connect two VPCs in the same or different regions             |
| VPN/Direct Connect| Connect on-premises networks to AWS resources                        |
| [[Transit gateway]]| Interconnect multiple VPCs, [[Git_hub_notes/certified-aws-solutions-architect-professional-main/03-networking/vpn|VPNs]], and Direct Connect endpoints      |
| Public Route Tables| Allow Internet access to specific resources                           |

Anti-pattern: Using [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering when there's a need for overlapping CIDR blocks. In such cases, use VPCs with non-overlapping CIDRs and configure routing accordingly.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be applied to control cross-account access using [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering. For instance, you might create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in Account A with permissions to assume a role in Account B, which then grants access to its resources. Here's a JSON snippet demonstrating this setup:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::ACCOUNT_A:root"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpc": "vpc-xxxxxxxxxxxxx"
                }
            }
        }
    ]
}
```

Additionally, enforce centralized governance through [[organizations|AWS Organizations]] Service Control [[policies]] (SCPs):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyVPCPeeringExceptAllowed",
            "Effect": "Deny",
            "Action": [
                "ec2:CreateVpcPeeringConnection",
                "ec2:AcceptVpcPeeringConnection"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringNotEqualsIfExists": {
                    "aws:PrincipalAccount": "ACCOUNT_ALLOWED_TO_PEER"
                }
            }
        }
    ]
}
```

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits apply to certain API operations related to [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering. To manage these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], implement exponential backoff strategies using SDKs or third-party tools. High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns include Multi-Region [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering and utilizing Transit Gateways for redundancy.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls can be implemented by monitoring data transfer costs associated with [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering. Calculate the total monthly cost as follows:

Total\_cost = Data\_transfer\_cost × Number\_of\_peered\_connections × Number\_of\_regions × Number\_of\_days

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1: Design a solution for connecting three VPCs across two AWS accounts for sharing resources while maintaining strong isolation and governance.

Correct answer: Implement Multi-Account [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering along with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles for cross-account access and SCPs to restrict unauthorized connections.

Incorrect answer: Utilize [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] or Direct Connect since they do not support multi-account configurations.

Scenario 2: Your organization uses [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering extensively. How would you implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] measures?

Correct answer: Monitor data transfer costs associated with [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering and ensure proper tagging of resources for accurate chargeback reporting.

Incorrect answer: Assume that [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering does not impact organizational costs because it doesn't involve external services or egress charges.