## Advanced Architecture

At its core, [[ec2|Capacity Reservations]] is a service that allows you to reserve [[ec2]] instance capacity in a specific Availability Zone (AZ) and instance type for any duration. This can be beneficial when you want to ensure you have access to dedicated resources for critical workloads or applications with specific performance requirements. The reservation system operates at the instance level, meaning it reserves the exact number of instances you specify, along with their CPU, memory, and network fabric capabilities.

On a more granular level, [[ec2|Capacity Reservations]] utilize convertible RI (Regional) and standard RI (Zonal) types. Convertible RIs enable [[ec2|capacity reservations]] across multiple instance families, while Standard RIs provide [[ec2|capacity reservations]] within a single instance family. Both options offer flexibility by allowing users to change the attributes of their reservation, such as instance type or AZ, without incurring additional costs.

[[RDS_Instance_Types|Global scale considerations]] come into play through the use of [[organizations|AWS Organizations]]. With [[organizations]], you can create a hierarchy of AWS accounts, apply Service Control [[policies]] (SCPs) to these accounts, and centrally manage [[billing]] and cost management. For example, you could set up an [[AWS Organization]] with a master account and several member accounts. In this setup, you can create [[ec2|Capacity Reservations]] in one member account and share them with other member accounts within the same organization.

## Comparison & Anti-Patterns

| Feature                     | [[ec2|Capacity Reservations]]         | [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]          | On-Demand Instances      |
|-----------------------------|-------------------------------|-------------------------|--------------------------|
| Instance Type Guarantee    | Yes (specific type)           | No                     | Yes (any type)            |
| Cost                        | Lower than On-Demand         | Lowest (but not guaranteed)| Highest                   |
| Duration                    | Can span years                | Short term (minutes)    | As long as needed          |
| Interruption Expectation   | None                         | High (subject to availability) | Never (guaranteed) |

Anti-patterns include using [[ec2|Capacity Reservations]] when [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]] would suffice, leading to unnecessary costs. Another common mistake is assuming [[ec2|Capacity Reservations]] guarantee instance availability during resource shortages or disasters. While they do secure capacity in a specific AZ, they don't protect against regional outages or resource exhaustion due to unforeseen demand surges.

## [[appsync|Security]] & Governance

To implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], you can use JSON statements like the following example, which grants permission to describe and modify [[ec2|Capacity Reservations]] for a specified user:

```json
{
  "Effect": "Allow",
  "Action": [
    "ec2:DescribeCapacityReservation",
    "ec2:ModifyCapacityReservation"
  ],
  "Resource": [
    "*"
  ],
  "Condition": {
    "StringEquals": {
      "aws:PrincipalArn": "arn:aws:iam::123456789012:user/user1"
    }
  }
}
```

Cross-account access can be achieved by attaching appropriate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions to the relevant AWS accounts. Additionally, you can enforce centralized governance using Organization SCPs, ensuring consistent [[appsync|security]] [[policies]] across all accounts.

## Performance & Reliability

[[ec2|Capacity Reservations]] support throttling limits based on the API rate limits defined by AWS. To handle throttling exceptions, implement exponential backoff strategies, allowing your application to automatically retry requests after waiting for increasing intervals.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns, consider deploying [[ec2|Capacity Reservations]] across multiple AZs or regions. By doing so, you can minimize the risk of downtime caused by AZ failures or regional outages.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be implemented using various methods, including:

1. Utilizing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]] (RIs) or Savings Plans to reduce overall compute costs
2. Implementing tagging strategies to track usage and allocate costs to individual teams or projects
3. Periodically reviewing underutilized [[ec2|Capacity Reservations]] and releasing unused resources

Calculating the cost savings from [[ec2|Capacity Reservations]] involves comparing the reserved capacity costs to the equivalent on-demand instance costs over the reservation period.

## Professional Exam Scenarios

### Scenario 1: Multi-Account Strategy

Suppose your company has two departments, Finance and Engineering, each with separate AWS accounts. They both need to run critical workloads on similar [[ec2]] instances but want to avoid unexpected capacity constraints. How would you address this challenge?

*Create a Capacity Reservation in one account and share it with the other.*

Correct answer: Create a Capacity Reservation in either the Finance or Engineering account and then share it with the other account. This ensures both accounts have access to the required instance capacity while minimizing costs.

Incorrect answer: Creating separate [[ec2|Capacity Reservations]] for each account may result in underutilized resources and increased costs.

### Scenario 2: [[RDS_Instance_Types|Global Scale Considerations]]

Assume your organization uses multiple AWS accounts spread across different regions and wants to ensure efficient utilization of [[ec2|Capacity Reservations]]. What steps should you take to achieve this goal?

*Implement [[organizations|AWS Organizations]] and use cross-account sharing of [[ec2|Capacity Reservations]].*

Correct answer: Implement [[organizations|AWS Organizations]] with a master account and member accounts for each region. Then, create [[ec2|Capacity Reservations]] in each member account and share them with the corresponding accounts in other regions. This enables efficient utilization of [[ec2|Capacity Reservations]] and promotes consistent [[appsync|security]] [[policies]] across all accounts.

Incorrect answer: Using a single Capacity Reservation across all regions does not allow for optimal allocation and utilization of reserved resources.