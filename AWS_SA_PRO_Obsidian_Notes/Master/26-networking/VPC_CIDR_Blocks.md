## [[RDS_Instance_Types|1. Advanced Architecture]]: [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] CIDR Blocks [[RDS_Instance_Types|Internals]]

At its core, a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] is a network infrastructure in Amazon's cloud that you define and control. It enables you to launch AWS resources into a virtual network that you've defined. VPCs are associated with a unique CIDR block, ranging from /16 to /28 netblocks, which you can divide into subnets.

Global scale is achieved by peering connections between VPCs within the same region or across different regions. This allows for interconnecting resources while maintaining isolation and separate administration. Transitive peering relationships are not supported, so you need to set up multiple peerings if you want full connectivity between all networks involved.

Under the hood, VPCs utilize AWS-managed hardware VTEPs (Virtual Tunnel Endpoints) connected to AWS backbone via Elastic Network Adapter (ENA) drivers. These components enable communication between instances in the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]].

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service               | Use Case                                                              |
| --------------------- | -------------------------------------------------------------------- |
| **[[Srinivas_Notes/VPC|VPC]] Peering**       | Interconnecting VPCs directly without transit via the public internet |
| **VPN/Direct Connect** | Securely connecting on-premises data centers to AWS VPCs             |
| **[[Transit gateway]]**   | Central hub for managing multiple VPCs, [[Git_hub_notes/certified-aws-solutions-architect-professional-main/03-networking/vpn|VPNs]], and Direct Connects    |
| **Interface Endpoint** | Accessing AWS Services (like [[Srinivas_Notes/S3|S3]], [[dynamodb]]) using private IPs         |

Anti-pattern: Using public subnets for DB servers. This exposes them to unnecessary risks since they can be accessed from the internet. Instead, use private subnets and secure access with Network Address Translation (NAT) gateways or egress-only internet gateways.

## [[RDS_Instance_Types|3. Security & Governance]]

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] example:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances"
      ],
      "Resource": [
        "*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:RebootInstances"
      ],
      "Resource": [
        "arn:aws:ec2:*::instance/*"
      ],
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Team": "Eng"
        }
      }
    }
  ]
}
```
Cross-account access:
- Policy grants permissions to specific principals (accounts or users):
```json
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "111122223333"
  },
  "Action": [
    "ec2:DescribeInstances"
  ],
  "Resource": [
    "*"
  ]
}
```
Organization Service Control [[policies]] (SCPs):
- Prevent creation of resources outside specific regions:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "ec2:RunInstances",
        "ec2:CreateHadoopCluster",
        ...
      ],
      "Resource": [
        "*"
      ],
      "Condition": {
        "StringNotEqualsIgnoreCase": {
          "aws:RequestedRegion": [
            "us-east-1",
            "us-west-2"
          ]
        }
      }
    }
  ]
}
```

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits depend on the action and API rate limits per second. For example, DescribeInstances has a limit of 50 calls per second. If these limits are exceeded, AWS returns a `403 ThrottlingException`. To mitigate such issues, implement exponential backoff strategies with jitter.

HA/DR patterns include running instances in multiple AZs within a single region and utilizing Multi-Region [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] scenarios.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls involve monitoring usage through [[cloudwatch|CloudWatch Alarms]] and setting up [[billing]] Alerts based on custom thresholds. Additionally, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]] provide cost savings when committing to instance usage for a term (1 or 3 years).

## 6. Professional Exam Scenario

Scenario 1: A company wants to extend their on-premises network to AWS but doesn't want traffic routed over the public internet. Which two solutions meet this requirement?

Correct Answer: [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] or Direct Connect.

Explanation: Both [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] and Direct Connect offer secure connectivity between an organization's data center and AWS resources. While [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] uses encryption for [[appsync|security]], Direct Connect provides a dedicated network connection. Neither routes traffic over the public internet.

Incorrect Answers: [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering, Interface Endpoint.

Explanation: [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering only works within AWS regions, not between on-premises networks and AWS. [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|Interface Endpoints]] allow private access to AWS services, not general connectivity.

---

Scenario 2: A research organization manages several VPCs containing sensitive scientific data. They must ensure that no unauthorized user or service can access these resources. What strategy would you recommend?

Correct Answer: Implement strict [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], restrict [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket access, and enforce resource tagging.

Explanation: By implementing tight [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], [[organizations]] can control who has access to specific resources. Restricting [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket access prevents accidental leaks of data stored there. Enforcing resource tagging helps manage costs and maintain visibility over used resources.