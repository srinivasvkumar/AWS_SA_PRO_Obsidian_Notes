## Advanced Architecture
At its core, an [[ec2]] Dedicated Host is a physical server that's dedicated for you to run your [[ec2]] instances on. This allows for workload control, visibility, and flexibility. The host has its own underlying hardware specifications such as socket count, core count, and memory configuration.

When it comes to [[RDS_Instance_Types|global scale considerations]], Dedicated Hosts are available in Regions only, not across multiple Regions or Availability Zones (AZs) due to their physical nature. However, you can launch instances from the same Dedicated Host in any AZ within the region.

Under the hood, when you purchase a Dedicated Host, you're essentially buying the right to place one or more [[ec2]] instances on it. Each host supports one or more instance types, allowing you to mix and match instance types on the same host. This is particularly useful if your application requires different types of instances to handle various workloads.

## Comparison & Anti-Patterns
Here are some scenarios where Instances might be more suitable than Dedicated Hosts:

| Use Case | Instance | Dedicated Host |
|---|---|---|
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost optimization]] | If your primary concern is minimizing costs, using [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] or [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]] can provide significant savings compared to Dedicated Hosts. | Dedicated Hosts may offer better price predictability but could end up being more expensive depending on the exact use case. |
| Workload variability | Instances can easily scale up or down based on demand, making them ideal for variable workloads. | Dedicated Hosts have a fixed capacity, so scaling would require adding additional hosts. |

Common anti-patterns include using Dedicated Hosts when they aren't necessary (e.g., due to licensing requirements) and not taking advantage of the ability to place multiple instance types on the same host.

## [[appsync|Security]] & Governance
From a [[appsync|security]] perspective, Dedicated Hosts allow you to apply host-based [[appsync|security]] rules like Windows Firewall settings or Linux iptables rules. Additionally, you can bring your own server-side encryption solutions.

For cross-account access, you need to set up [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering and ensure proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions are configured. Here's an example JSON policy snippet granting access:

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
                "ec2:DescribeInstances",
                "ec2:DescribeHosts"
            ],
            "Resource": "*"
        }
    ]
}
```

To enforce organization-wide restrictions, Service Control [[policies]] (SCPs) can be used at the Organization level to limit who can create and manage Dedicated Hosts.

## Performance & Reliability
There are no throttling limits associated with Dedicated Hosts beyond those imposed by the instances themselves. Therefore, exponential backoff strategies typically applied to other services don't come into play here.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns depend on the application architecture built on top of the Dedicated Hosts. For example, placing instances from the same host in different AZs provides regional failover capabilities.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls with Dedicated Hosts primarily revolve around host usage. You pay for each active host per hour, regardless of whether there are any instances running on it. To calculate costs, consider factors such as the number of hosts, instance types, and region.

## Professional Exam Scenarios
### Scenario 1
Your company runs a mission-critical application on [[ec2]] instances. Due to regulatory compliance, all instances must reside on physically isolated hardware. Which two actions would best meet these requirements?

Correct Answers:
1. Purchase [[ec2]] Dedicated Hosts to ensure physically isolated hardware.
2. Ensure instances launched on Dedicated Hosts comply with regulatory requirements.

Incorrect Answer:
3. Launch instances in a separate [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] since this does not guarantee physically isolated hardware.

Justification: Physically isolating instances through Dedicated Hosts ensures compliance while maintaining the benefits of [[ec2]]'s scalability and flexibility.

### Scenario 2
An e-commerce website needs to minimize costs during off-peak hours but still maintain consistent performance. What combination of services should be used?

Correct Answer:
1. Run production workloads on Dedicated Hosts for consistent performance.
2. Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] during off-peak hours to reduce costs.

Incorrect Answer:
3. Use On-Demand Instances since they do not provide cost savings compared to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]].