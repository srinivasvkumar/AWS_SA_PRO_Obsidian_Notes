## Advanced Architecture

At its core, an Internet Gateway (IGW) is a simple service that enables connectivity between Amazon VPCs and the internet. However, there are several advanced architecture concepts to consider:

* **Global Accelerator Integration**: IGW can be integrated with Global Accelerator to improve application performance by directing user traffic to the closest regional endpoint. This reduces latency and improps load times.
* **Internet Gateway Metadata**: Each IGW has a unique IPv4 address CIDR block (e.g., 11.0.0.0/8) associated with it. While not directly useful in most scenarios, understanding the internal workings of IGWs can help when troubleshooting or designing complex systems.
* **IPv6 Support**: IGW supports IPv6 addressing without additional configuration required. Simply enable IPv6 in your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] and create an IPv6 egress-only internet gateway.

## Comparison & Anti-Patterns

| Service          | Use Case                                                              |
| --------------- | -------------------------------------------------------------------- |
| Internet Gateway | Connect [[Srinivas_Notes/VPC|VPC]] to the internet                                         |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]]     | Enable instances in private subnets to communicate with the internet |
| Egress-only IGW   | Restrict outbound internet access for specific resources            |

An anti-pattern would be using an IGW as a default gateway for instances within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. Instead, use a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]] or Egress-only IGW for controlled internet connectivity.

## [[appsync|Security]] & Governance

To manage cross-account access, you need to set up proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and Service Control [[policies]] (SCPs). Here's an example JSON policy allowing access to an IGW from another account:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInternetGateways"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpc": "arn:aws:ec2:region::vpc/vpc-abcdefgh"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateInternetGateway",
                "ec2:AttachInternetGateway",
                "ec2:DeleteInternetGateway"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:SourceAccount": "123456789012"
                },
                "StringEqualsIgnoreCase": {
                    "ec2:ResourceTag/Environment": "production"
                }
            }
        }
    ]
}
```

For organization-wide governance, use Service Control [[policies]] (SCPs) to enforce tagging and naming standards. For instance, restrict IGW creation to users who have added specific tags like `Team:Network`.

## Performance & Reliability

There are no throttling limits for IGW operations. However, if you encounter [[api-gateway|errors]] related to creating, attaching, or deleting IGWs, implement exponential backoff strategies in your automation scripts.

High availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] ([[dr]]) patterns include placing multiple IGWs across different Availability Zones within a single region or using Global Accelerator to distribute traffic globally.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

There are no granular cost controls for IGWs. The only [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] strategy is to remove unused IGWs. To calculate costs, use the AWS Pricing Calculator or the [[billing|Cost Explorer]] tool.

## Professional Exam Scenario 1

You are tasked with implementing a solution for a company with two AWS accounts: one production account and one development account. The development team requires limited access to the production environment's IGW. Which actions should you take?

Correct answer: Implement cross-account [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] allowing developers to describe and attach IGWs in the production account while enforcing tag restrictions through SCPs.

Incorrect answer: Create a separate IGW in the development account and configure a peering connection between the two VPCs. This approach introduces unnecessary complexity and does not provide fine-grained control over IGW access.

## Professional Exam Scenario 2

Your organization wants to minimize the impact of potential DDoS attacks targeting their public-facing web applications hosted in [[ec2]] instances behind an IGW. Which steps should you follow to increase [[appsync|security]] and reliability?

Correct answer: Implement [[shield|AWS Shield]] Standard protection, enable route tables to send traffic through Network Access Control Lists (NACLs), and configure WAF rules to filter malicious requests. Consider adding [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] in front of the IGW to distribute traffic globally and reduce the likelihood of facing DDoS threats.

Incorrect answer: Adding more IGWs will not mitigate DDoS risks. Increasing the number of IGWs may lead to higher costs but does not add any [[appsync|security]] benefits.