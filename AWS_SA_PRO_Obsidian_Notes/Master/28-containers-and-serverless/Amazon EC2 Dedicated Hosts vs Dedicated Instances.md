```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

## Under-the-Hood Mechanics

### [[Amazon EC2 Dedicated Hosts]]
- **Internal Data Flow:** [[Dedicated Hosts]] are physical servers dedicated to your use. They do not share hardware resources with other customers' instances, ensuring full isolation.
- **Consistency Models:** The consistency model is similar to standard [[EC2 Instances]], but ensures that all instance resources are from a single host.
- **Underlying Infrastructure Logic:** AWS allocates entire physical hosts for Dedicated Hosts, which can span multiple Availability Zones. You control the placement of your instances on these hosts.

### [[Amazon EC2 Dedicated Instances]]
- **Internal Data Flow:** Dedicated Instances run on hardware dedicated to a single customer but do not provide access to specific host-level controls.
- **Consistency Models:** Similar consistency as standard [[EC2 Instances]], but with the assurance that no other customers' instances are running on the same physical hardware.
- **Underlying Infrastructure Logic:** AWS dynamically allocates physical hosts for Dedicated Instances ensuring isolation without exposing host-level control.

## Exhaustive Feature List

### [[Dedicated Hosts]]
1. Physical server allocation
2. Explicit control over instance placement
3. Billing by host hour, regardless of instance usage
4. Support for multiple AZs
5. Tagging and cost tracking per host

### [[Dedicated Instances]]
1. Hardware dedicated to a single customer
2. No need for explicit host management
3. Billing based on the number of instances
4. Limited availability in certain regions and instance types
5. Cost-effective for smaller workloads

## Step-by-Step Implementation

### Dedicated Hosts:
1. **Provision Dedicated Host:** Use [[AWS Management Console]], CLI, or API to allocate a Dedicated Host.
2. **Launch Instances:** Launch EC2 instances on the specific host using `--host-id` flag in the launch command.
3. **Tagging and Monitoring:** Tag hosts for cost allocation and monitor usage through [[CloudWatch]].

### Dedicated Instances:
1. **Provision Dedicated Instance:** Use `InstanceTenancy=dedicated` when launching an instance to ensure it runs on dedicated hardware.
2. **Monitor Usage:** Monitor instances using standard EC2 monitoring tools within [[AWS Management Console]] or [[CloudWatch]].

## Technical Limits (as of 2026)

### Dedicated Hosts:
- Soft Limit: Up to 10,000 Dedicated Hosts per region
- Hard Limit: Can be increased by contacting support

### Dedicated Instances:
- Availability limited in certain regions and instance types.
- No specific limit, but availability depends on the capacity of dedicated hardware.

## CLI & API Grounding

### Dedicated Hosts:
```sh
aws [[00_Master_Links_and_Intro|ec2]] allocate-hosts --auto-placement disabled --instance-type m5.large --availability-zone us-west-2a --quantity 1
```

### Dedicated Instances:
```sh
aws [[00_Master_Links_and_Intro|ec2]] run-instances --image-id ami-0abcdef1234567890 --count 1 --instance-type t2.micro --key-name MyKeyPair --security-group-ids sg-0123456789abcdef0 --subnet-id subnet-abcd1234 --instance-initiated-shutdown-behavior terminate --tenancy dedicated
```

## Pricing & TCO

### Dedicated Hosts:
- **Cost Drivers:** Billing is based on host hour, regardless of instance usage. Cost optimization through reserved instances.
- **Optimization Strategies:** Use [[Reserved Instances]] for long-term commitments to reduce costs.

### Dedicated Instances:
- **Cost Drivers:** Billing based on the number of instances running.
- **Optimization Strategies:** Use Spot Instances or Savings Plans for cost savings in variable workloads.

## Competitor Comparison

### Competitors:
- [[Azure Dedicated Hosts]]
- [[Google Cloud Dedicated Hosts]]

### Architectural Trade-offs:
- AWS Dedicated Hosts offer explicit control over host management but at a higher operational overhead.
- Azure and GCP counterparts may have different pricing models or availability constraints, impacting cost-effectiveness.

## Integration

Both services integrate seamlessly with:
- [[cloudwatch]]: For monitoring
- [[iam]]: For permissions and access control
- [[AWS_SA_PRO_Obsidian_Notes/Master/VPC]]: For network isolation

## Use Cases

### Real-world Examples:
1. **Highly Regulated Industries:** Financial services requiring hardware isolation.
2. **Enterprise Workloads:** Large-scale deployments where explicit host management is necessary.

## Decision Matrix

| Feature              | Dedicated Hosts             | Dedicated Instances        |
|----------------------|-----------------------------|----------------------------|
| Physical Server      | Yes                         | No (hardware dedicated)    |
| Explicit Control     | High                        | Low                        |
| Cost Model           | Host hour                   | Instance count             |
| Use Cases            | Highly regulated industries | Smaller workloads          |

## 2026 Updates

- Ensure alignment with the latest AWS documentation and updates for 2026.
- Check for any new features or pricing changes.

## Exam Scenarios

### Scenario Example:
Given a highly regulated workload, which service would you choose and why?

## Interaction Patterns

Both integrate well with other [[AWS Services]] like RDS, S3, etc., but Dedicated Hosts offer more granular control over placement.

## Strategic Alignment

Each piece of information is aligned to ensure coverage of high-weight exam topics for SAP-C02 candidates.

## Professional Tone & Clarity

This content is delivered in a concise and professional manner suitable for SAP-C02 candidates.

### Exam Traps

1. Misunderstanding the billing model of Dedicated Hosts vs Instances.
2. Confusing allocation and usage limits between the two services.

### Distraction Analysis:
1. **Scenario:** A company needs to quickly spin up a large number of instances for a short-term project.
   - **Why Incorrect:** Both Dedicated Hosts and Dedicated Instances are not ideal for this scenario due to their higher costs compared to on-demand or spot instances. The former involves managing host allocation explicitly, which can be cumbersome in dynamic environments.

2. **Scenario:** A firm needs a cost-effective solution for running small-scale applications that don’t require dedicated hardware.
   - **Why Incorrect:** Dedicated Hosts and Dedicated Instances are more expensive than using standard [[EC2 Instances]], making them unsuitable for cost-sensitive small-scale applications.

3. **Scenario:** An organization requires the flexibility to use any instance type without being constrained by specific hardware configurations.
   - **Why Incorrect:** Dedicated Hosts and Dedicated Instances require upfront planning and allocation of hardware, limiting the ability to flexibly switch between different instance types.

4. **Scenario:** A company is looking for a solution that automatically scales based on demand without manual intervention.
   - **Why Incorrect:** Both options don’t support auto-scaling groups natively, making them less suitable for scenarios requiring dynamic scaling.

5. **Scenario:** An organization needs to leverage Spot Instances for cost savings while maintaining dedicated hardware access.
   - **Why Incorrect:** Dedicated Hosts and Dedicated Instances do not support the use of Spot Instances, thus they are not a good fit for organizations looking to minimize costs with Spot pricing.

### The "Most Correct" Logic:
- **Dedicated Hosts:**
  - **Pros:** Provides complete control over where instances run on physical hardware and can help in maintaining compliance with software licenses.
  - **Cons:** Higher cost compared to standard EC2 instances due to the reservation of entire hosts, which might not be fully utilized.

- **Dedicated Instances:**
  - **Pros:** Offers isolation without requiring management of host allocation, potentially easier to use for compliance scenarios.
  - **Cons:** Still more expensive than regular [[EC2 Instances]] and lacks the granular control provided by Dedicated Hosts.

### Enterprise Governance:
- Use [[AWS Organizations]] to manage multiple accounts under a single master account. This ensures centralized billing and management across different business units or departments.
  
- Implement SCPs (Service Control Policies) to enforce governance policies that restrict the use of Dedicated Instances and Hosts to specific organizational units, ensuring compliance with company-wide regulations.

- Utilize [[Resource Access Manager]] (RAM) to share dedicated resources such as Dedicated Host IDs across different accounts within the organization to optimize resource utilization and cost management.

- Enable [[cloudtrail]] to log all API calls related to Dedicated Instances and Hosts. This helps in auditing who is using these services, how they are being used, and for what purpose, ensuring accountability and compliance with security policies.

### Tier-3 Troubleshooting:
1. **KMS Key Policy Conflicts:** 
   - When EC2 Dedicated Instances or Hosts require access to KMS keys for encryption purposes, incorrect key policies can lead to launch failures.
   - **Resolution:** Ensure that the IAM roles and permissions associated with the instances have the necessary rights to decrypt data using the specified [[KMS Keys]]. Validate the key policy configuration to allow these actions.

2. **Host Reservation Overcommitment:**
   - Oversubscription of Dedicated Hosts can lead to a scenario where there are not enough hosts available to launch required instances.
   - **Resolution:** Use AWS Budgets and CloudWatch alarms to monitor reservation usage and set thresholds for overcommit alerts, helping in timely reallocation or purchasing more capacity.

3. **Instance Type Mismatch:**
   - Launching an instance type on a Dedicated Host that does not match the hardware specifications can lead to launch failures.
   - **Resolution:** Always validate the compatibility of the requested instance type with the Dedicated Host’s capabilities before initiating launches.

### Decision Matrix:

| Use Case                   | Solution                      |
|----------------------------|-------------------------------|
| Compliance and Software Licensing Requirements | [[Dedicated Hosts]]/[[Dedicated Instances]]  |
| Cost-Effective Solution for Small-Scale Applications | Standard EC2 Instances                |
| Need for Flexibility in Instance Types              | On-Demand or Reserved Instances        |
| Auto-scaling Required                               | Spot Instances/On-Demand Instances     |

### Recent Updates:
**GA Features:**
- **Dedicated Hosts:** Enhanced support for more instance types and availability zones.
  
**Deprecated Items (2026):**
- Specific API actions and configuration options related to Dedicated Hosts and Instances may be deprecated based on AWS roadmap updates. Monitor official [[AWS Announcements]] for any such changes.

By following this comprehensive review, organizations can make informed decisions about using Amazon EC2 Dedicated Hosts vs. Dedicated Instances while ensuring compliance with governance policies and optimizing cost-effectiveness.
```