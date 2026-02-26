**[[RDS_Instance_Types|1. Advanced Architecture]]**

AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]] (RIs) provide significant discounts compared to On-Demand instances by reserving a capacity in a specific Availability Zone (AZ) or within a region. RIs can be purchased for a 1-year or 3-year term, and offer three payment options: All Upfront, Partial Upfront, or No Upfront.

**Global Considerations:**
- Convertible RIs allow changing instance families, instance types, and [[eb|platforms]], but not regions or Availability Zones.
- Regional RIs enable using reserved capacity across instances in multiple AZs within a single region.
- Scheduled RIs let you reserve instances that launch within a recurring schedule over a time period of up to a year.

**Under The Hood Mechanics:**
- Instance Size Flexibility enables running different sizes of instances under the same RI.
- RI Marketplace allows selling unused RIs from one account to another.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Cases |
| --- | --- |
| **[[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]** | Short-term workloads, fault-tolerant applications, background processing tasks, [[cicd|CI/CD]] pipelines |
| **On-Demand Instances** | Workloads with short-term variability, unpredictable usage patterns, new application testing, development environments |
| **[[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]]** | Steady-state production workloads, databases, predetermined long-term usage requirements |

*Anti-pattern:* Using RIs as a standalone solution without considering other services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] or Auto Scaling Groups (ASGs).

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] example:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "ec2:DescribeInstances",
              "ec2:DescribeReservedInstances"
            ],
            "Resource": "*"
        }
    ]
}
```
Cross-account access:
- Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in Account A with permissions to manage RIs.
- Grant permissions to Account B users via trust policy.
- Access RIs from Account B using assumed roles.

Organization SCPs:
- Deny creating RIs at the organization level if not following your company's tagging strategy.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling Limits:
- Maximum number of RIs per region: 5,000 (default quota).
- Increase the limit through the Support Center.

Exponential Backoff Strategies:
- Implement retry mechanisms when requesting RI changes or modifications.

HA/DR Patterns:
- Use RIs for steady-state production workloads in both primary and secondary regions.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular Cost Controls:
- Apply tags to RIs based on resource type, owner, team, or environment.
- Monitor costs using AWS [[billing|Cost Explorer]], Trusted Advisor, and third-party tools.

Calculation Examples:
- Assume a t2.micro instance costs $0.013 per hour on-demand.
- Purchasing a 1-year All Upfront RI would result in a 75% discount ($0.013 * 0.25 = $0.00325 per hour).

**6. Professional Exam Scenario: Vignette-Style Questions**

Scenario 1:
Your organization has a large number of [[ec2]] instances distributed among multiple accounts. You need to ensure efficient utilization of resources while minimizing costs. Which AWS services should be considered?

Correct Answer: Combine Reservation Modifications (to maximize RI coverage) with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] (for non-critical workloads) and ASGs (for autoscaling and load distribution).

Incorrect Answer: Only purchase RIs without considering other services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] or ASGs.

Scenario 2:
You have been tasked with implementing a centralized governance model for managing RIs across multiple AWS accounts. What measures should be taken?

Correct Answer: Implement Organizational Service Control [[policies]] (SCPs) to enforce tagging standards, and grant cross-account access to shared RIs using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles.

Incorrect Answer: Allow each individual account to manage their own RIs without any central oversight or guidelines.

### Architecture Diagram
![[EC2 Reserved Instances Savings Plan.png]]


### Architecture Diagram
![[EC2 Reserved Instances Capacy reservation.png]]
