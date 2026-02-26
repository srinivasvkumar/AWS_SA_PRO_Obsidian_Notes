**[[RDS_Instance_Types|1. Advanced Architecture]]**

ENIs are virtual network interfaces that you can attach to an instance in a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] to provide network connectivity. They support IPv4 and IPv6 dual stack, multiple [[appsync|security]] groups, and enable network customizations like ClassicLink, ENI tagging, and dedicated tenancy.

ENIs have specific attributes such as a primary private IPv4 address, one or more secondary private IPv4 addresses, one Elastic IP address per private IPv4 address, a MAC address, a source/destination check flag, a description, and a set of [[appsync|security]] groups.

At the global scale, ENIs operate within a single region but can be used to build multi-region solutions using peering connections, [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|VPNs]], or Direct Connect. However, they cannot span regions directly. Under the hood, they rely on Amazon's Neutron infrastructure, which is built on top of Linux Bridge and Open vSwitch technologies.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service          | Use Case                                           |
|------------------|-----------------------------------------------------|
| ENI              | High-performance computing, load balancing         |
| Network Interface| Legacy EC2-Classic setup migration                |
| VM Import/Export | Lift-and-shift migrations from other cloud providers   |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/11-migrations/datasync|AWS DataSync]]     | Data transfer between on-premises storage and AWS    |

Anti-pattern: Using ENIs when another service would better serve your needs, such as Network Interfaces for new applications instead of migrating existing ones.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeNetworkInterfaces",
        "ec2:AttachNetworkInterface"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Team": "DevOps"
        }
      }
    }
  ]
}
```
Cross-account access:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": [
        "ec2:DescribeNetworkInterfaces",
        "ec2:AttachNetworkInterface"
      ],
      "Resource": "arn:aws:ec2:us-west-2:111122223333:network-interface/*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Project": "SharedProject"
        }
      }
    }
  ]
}
```
Organization SCPs:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnapprovedENIAttachment",
      "Effect": "Deny",
      "Action": "ec2:AttachNetworkInterface",
      "Resource": "arn:aws:ec2:*::network-interface/*",
      "Condition": {
        "Null": {
          "ec2:ResourceTag/ApprovedBy": ["support@example.com"]
        }
      }
    }
  ]
}
```

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits: Attachment/detachment operations are limited to five per minute. To manage throttling, implement exponential backoff strategies using SDKs.

HA/DR patterns: Use Auto Scaling Groups with ENIs to ensure high availability. For [[dr]], create standby instances in another region with their own ENIs.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls: Tag ENIs to monitor usage and apply appropriate cost allocation. Calculate costs with the AWS Pricing Calculator.

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1: A company wants to share resources between two accounts while enforcing strict control over ENI attachment permissions. The main account should approve any resource sharing.

Correct answer: Implement cross-account access with resource tags and an Organizational [[SCP]] denying unapproved ENI attachment.

Incorrect answer: Allowing full access to both accounts without [[cloudformation|conditions]]. This violates the principle of least privilege.

Scenario 2: A research organization runs large-scale simulations requiring high-performance computing nodes connected via ENIs. They need to optimize costs by detaching ENIs during off-peak hours.

Correct answer: Schedule Auto Scaling Group termination and launch based on workload demands.

Incorrect answer: Manually detach ENIs during off-peak hours, increasing management overhead and potential human [[api-gateway|errors]].