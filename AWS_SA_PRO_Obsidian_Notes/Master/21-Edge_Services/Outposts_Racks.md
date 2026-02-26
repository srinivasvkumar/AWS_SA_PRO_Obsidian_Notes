### Advanced Architecture
At its core, Outposts racks consist of two primary components: the infrastructure and the Outpost-enabled services. The infrastructure includes one or more Outposts racks that can be connected to your AWS account and region. Each rack contains one or more instances of supported Amazon [[ec2]] instance types, as well as optional storage ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]], [[fsx]]) and networking (ENIs, [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]) resources.

The global nature of Outposts racks requires careful planning and consideration. For example, you must determine how many racks you need in each location, whether to enable specific services, and what network connectivity options to use. Additionally, you should plan for sufficient capacity to accommodate future growth while minimizing underutilized resources.

### Comparison & Anti-Patterns
Here is a comparison table for selecting between Outposts racks and other edge services like [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|AWS Wavelength]], Local Zones, and [[snow|Snowball Edge]].

| Service         | Suitable Use Cases                               | Anti-Patterns                            |
|-----------------|---------------------------------------------------|--------------------------------------------|
| Outposts Racks   | Running applications close to on-premises systems  | Requiring low latency < 10 ms            |
| [[Collab_Notes_detailed/Core_Compute/Wavelength|Wavelength]]      | Ultra-low latency mobile edge computing           | Not using 5G networks                    |
| Local Zones     | High-performance workloads requiring locality    | Needing full AWS service availability        |
| [[snow|Snowball Edge]]   | Data transfer, processing, and migration          | Occasional data transfers without permanence|

Some common anti-patterns include attempting to achieve ultra-low latency (< 10ms) with Outposts racks instead of [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] or assuming all services are available on Outposts racks when some may not be.

### [[appsync|Security]] & Governance
[[appsync|Security]] [[policies]] for Outposts racks require careful planning, especially when working across multiple accounts. To implement secure access, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy allowing users to manage their Outposts racks within their respective AWS accounts. Here's an example JSON policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "outposts:*"
            ],
            "Resource": [
              "*"
            ],
            "Condition": {
              "StringEquals": {
                "aws:PrincipalAccount": ["ACCOUNT_ID"]
              }
            }
        }
    ]
}
```

For cross-account access, use [[sts]] AssumeRole to grant temporary permissions from one account to another. In addition, apply Service Control [[policies]] (SCPs) at the organization level to enforce [[control-tower|guardrails]] and restrict unwanted actions.

### Performance & Reliability
Throttling limits vary depending on the Outposts rack size and type. Monitor API calls and operations to avoid hitting these limits. Implement exponential backoff strategies to handle throttled requests gracefully.

To increase high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities, deploy multiple Outposts racks across different Availability Zones or regions. This setup allows automatic failover and manual switchovers during outages.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls are essential for optimizing costs associated with Outposts racks. Firstly, monitor usage metrics such as instance hours, data transferred, and storage consumption. Secondly, enable [[billing]] alerts to track spending thresholds. Lastly, calculate total cost by adding up the following factors:

* Instance costs (On-Demand, Reserved, or Spot)
* Storage costs ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]], [[fsx]])
* Networking costs (ENI, [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]])
* Other AWS service costs used with Outposts racks

### Professional Exam Scenarios
#### Scenario 1
You're designing a hybrid cloud solution for a manufacturing company connecting their on-premises system to AWS. They have strict compliance requirements and need low-latency connections between their facilities and AWS. Which solution would best suit their needs?

Correct Answer: Outposts racks provide the desired low-latency connection to on-premises systems while meeting compliance requirements.

Incorrect Answers:

* [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] does not offer direct connections to on-premises systems.
* Local Zones lack support for running applications close to on-premises systems.
* [[snow|Snowball Edge]] is designed for occasional data transfers rather than permanent connections.

#### Scenario 2
Your company operates an [[iot]] platform with strict data privacy regulations. You want to process and analyze data closer to the source while maintaining regulatory compliance. Which AWS edge service meets these requirements?

Correct Answer: Outposts racks offer the ability to process data near the source while adhering to regulatory requirements.

Incorrect Answers:

* [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] focuses on 5G mobile edge computing, which doesn't align with [[iot]] devices.
* Local Zones lack support for running applications close to [[iot]] devices.
* [[snow|Snowball Edge]] is designed for occasional data transfers rather than continuous processing.