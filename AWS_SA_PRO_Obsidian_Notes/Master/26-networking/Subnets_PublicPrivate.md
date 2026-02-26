### Advanced Architecture
At its core, a subnet is a range of IP addresses within a single [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. Subnets can be categorized as Public or Private based on their accessibility from the internet. A primary determinant of this classification is the presence or absence of a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]] or an Internet Gateway in the subnet's route tables.

#### [[RDS_Instance_Types|Global Scale Considerations]]
When designing a global infrastructure, it's essential to understand that [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering, [[transit-gateway|AWS Transit Gateway]], and [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] operate at the global level while other networking components like Subnets, Route Tables, and [[appsync|Security]] Groups function within a specific region. Therefore, direct communication between instances in different regions requires traversing the public internet or using [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connectivity options.

To achieve true global scale, you might need to implement multi-region active-active setups using services like [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] DNS routing [[policies]]. This allows you to serve users from multiple locations with lower latency and higher availability.

#### Under the Hood Mechanics
Each subnet has a default network access control group (NACL) associated with it, which acts as a stateless firewall. While these are optional, they provide additional [[appsync|security]] by controlling ingress and egress traffic at the subnet level. 

For managing routing rules, we rely on Route Tables, which determine how data flows inside and outside the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. By associating specific routes with Subnets, we can create customized communication pathways based on our requirements.

### Comparison & Anti-Patterns

| Service          | Use Case                                                              |
|-----------------|----------------------------------------------------------------------|
| Subnets         | Logical segmentation of resources within a [[Srinivas_Notes/VPC|VPC]]                       |
| [[transit-gateway|AWS Transit Gateway]]    | Connecting multiple VPCs, data centers, or remote networks   |
| [[Srinivas_Notes/VPC|VPC]] Peering     | Inter-region / inter-VPC connectivity without touching the internet |
| Direct Connect   | High-bandwidth, dedicated connections to your data center           |

**Anti-pattern:** Using Public Subnets for sensitive workloads. This exposes them to potential attacks from the internet. Instead, leverage Private Subnets and secure bastion hosts or jump boxes for controlled access.

### [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
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
    }
  ]
}
```
Cross-Account Access:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "ec2:*",
      "Resource": "arn:aws:ec2:us-west-2:111122223333:instance/*"
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
      "Sid": "DenyEc2RunInstanceInProductionVpc",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEqualsIgnoreCase": {
          "ec2:vpc": "poc-vpc"
        },
        "ArnLike": {
          "ec2:vpc-security-group-egress-policy": "arn:aws:ec2:us-west-2:111122223333:egress-policy/*"
        }
      }
    }
  ]
}
```

### Performance & Reliability

Throttling Limits:
- ENIs per [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]: 500
- Network Interfaces per Instance: 3
- Elastic IPs per Region: 5

Exponential Backoff Strategies:
1. Wait twice as long before each retrial.
2. Limit the total number of retries.
3. Define a maximum waiting time.

HA/DR Patterns:
- Multi-AZ deployments for critical applications.
- Active-passive architectures with Auto Scaling groups and Application Load Balancers.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls include using Amazon [[vpc-flow-logs|VPC Flow Logs]] only when necessary, monitoring and adjusting [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volume sizes, and terminating unused instances. For example, if you have a private subnet with no internet-facing components, there would be little reason to enable flow logs.

Calculation Examples:
- Public Subnet (w/ [[elb]]): $0.024 per hour (Linux t2.micro instance) + ~$0.008 per GB data processed.
- Private Subnet (w/o [[elb]]): $0.024 per hour (Linux t2.micro instance).

### Professional Exam Scenario Questions

**Scenario 1:** Your company wants to launch a new product in multiple regions simultaneously while maintaining strict [[appsync|security]] [[policies]]. They require high availability and low-latency access to the application. How would you design this architecture?

Correct Answer: Implement a multi-region active-active setup using [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] along with private subnets in each region. Attach instances or Auto Scaling groups to these subnets. Ensure proper routing via Route Tables and NACLs. Utilize [[shield|AWS Shield]] for DDoS protection and AWS WAF for web application firewall capabilities.

Incorrect Answer: Create public subnets in each region and place instances directly within those subnets. This approach does not follow [[iam|best practices]] regarding [[appsync|security]] and separation of concerns.

**Scenario 2:** Due to regulatory compliance, your organization needs to limit [[ec2]] instance creation only to certain VPCs across all AWS accounts. How would you enforce this restriction?

Correct Answer: Apply an Organization Service Control Policy ([[SCP]]) at the root level to deny [[ec2]]:RunInstances action within specific VPCs.

Incorrect Answer: Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] restricting [[ec2]]:RunInstances actions on individual AWS accounts. This solution lacks central management and may lead to inconsistencies across various accounts.