## [[RDS_Instance_Types|1. Advanced Architecture]]

Local Zones are part of [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|AWS Outposts]] family, but unlike Outposts they don't bring AWS infrastructure to your data center. Instead, Local Zones deploy a subset of AWS services within an existing region, providing single-digit millisecond latency to local applications. Under the hood, each Local Zone has its own set of Availability Zones, which are physically separate from the main region.

Local Zone architecture consists of three main components:

- **Local Zone Infrastructure**: A collection of [[ec2]] instances, ENIs, subnets, route tables, and [[appsync|security]] groups that function similarly to their counterparts in the main region. These resources are isolated from the main region and have different IP addresses.
- **Regional Connectivity**: High-bandwidth, low-latency connections between the Local Zone and the main region using AWS’ private network infrastructure. This connectivity enables inter-region [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering, allowing communication between resources in the Local Zone and the main region.
- **Local Zone Management**: Tools such as Service Quotas, Trusted Advisor, and [[billing|Cost Explorer]] to manage and monitor Local Zone resources separately from the main region.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service | Latency | Data Transfer | Scaling | Global Footprint |
|---|---|---|---|---|
| **Local Zone** | Single-digit ms | Within Local Zone: $0.01/GB; outside: standard rates | Per-service scaling | Select regions only |
| **Outposts** | ~100 ms | Free within Outpost; standard rates for external transfer | Per-rack scaling | Customer-specific locations |
| **Global Accelerator** | Sub-10 ms | Free within accelerator; standard rates for external transfer | Stateless load balancing only | ~100 cities worldwide |

Anti-patterns include:

- Using Local Zones for global footprint when Global Accelerator is more suitable.
- Expecting full Outposts functionality from Local Zones.
- Ignoring the additional costs associated with managing Local Zone resources compared to equivalent resources in the main region.

## [[RDS_Instance_Types|3. Security & Governance]]

### Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "ec2:DescribeInstances",
              "ec2:StartInstances",
              "ec2:StopInstances"
            ],
            "Resource": [
              "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": ["us-west-2", "local-zone-name"]
                }
            }
        }
    ]
}
```

### Cross-Account Access

Use [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering or [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC Endpoints]] to enable cross-account access between the main region and the Local Zone.

### Organization SCPs

Prevent unwanted resource creation by setting SCPs at the organization level. For example, deny `ec2:RunInstances` for all accounts except specific ones allowed to create instances in the Local Zone.

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits depend on the individual service and can be increased through Service Quotas. Implement exponential backoff strategies for throttled API requests.

HA/DR patterns involve running mission-critical workloads in both the main region and Local Zone, with synchronous replication between them.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls include per-second [[billing]], reserved capacity, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]]. Calculate costs based on the number of instances, instance types, storage configurations, and data transfer requirements.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario 1: Multi-account Strategy

Your company uses multiple AWS accounts for production, staging, and testing environments. They want to leverage Local Zones for lower latency while keeping the same account structure.

Correct answer: Create Local Zones in select regions and implement cross-account [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering between the main region and the Local Zone. Ensure proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] allow necessary actions within the Local Zone.

Incorrect answer: Assume Local Zones are available in every region, or merge all accounts into one to simplify management.

### Scenario 2: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Your team needs to run a GPU-intensive ML workload in a Local Zone due to latency requirements. The solution must minimize costs.

Correct answer: Utilize [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]] for the ML training jobs, and switch to on-demand instances for short periods during hyperparameter tuning. Enable per-second [[billing]] and optimize storage usage.

Incorrect answer: Run all instances as on-demand, or ignore Local Zone costs and assume they will be similar to the main region.

### Architecture Diagram
![[AWS_Local_Zones.png]]
