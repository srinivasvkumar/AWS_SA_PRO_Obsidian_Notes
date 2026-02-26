## Advanced Architecture

At its core, an Instance Store is a virtual machine's temporary block-storage device that uses the local physical disks of the host server. It provides very low latency and high input/output operations per second (IOPS) suitable for applications requiring fast, ephemeral storage.

The Instance Store architecture consists of two main components: the Instance Store control plane and data plane. The control plane manages metadata about the store, while the data plane handles actual data blocks. Each instance type has a specific instance store capacity, which can vary from a few GBs to multiple TBs.

Global scale is achieved by replicating AMIs (Amazon Machine Images) across regions, allowing users to launch instances with identical [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Instance Stores]] in different geographic locations. However, there’s no built-in mechanism for automatic data replication between instances or regions. Users must implement their custom solutions if they need synchronous replication.

## Comparison & Anti-Patterns

| Service               | Use Case                                                              |
| --------------------- | -------------------------------------------------------------------- |
| Instance Store        | Demanding IOPS workloads, short-term data processing                |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] (General Purpose) | Persistent data, flexible performance options                       |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] (Network File System)| Shared file systems, POSIX compliance                             |
| [[Srinivas_Notes/S3|S3]] (Object Storage)    | Long-term archival, web-scale data distribution                      |

Anti-patterns include storing long-term stateful data on Instance Store, as it doesn’t persist beyond the life of the instance. Also, using it for shared storage scenarios without proper locking mechanisms may lead to inconsistencies and data corruption.

## [[appsync|Security]] & Governance

For cross-account access, you can create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account and attach an instance profile to your [[ec2]] instances. In the target account, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy granting appropriate permissions to the ARN of the resource(s) and attach it to a relevant user, group, or role.

Here's an example JSON policy allowing read-only access to an [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket in another account:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::source-account-id:role/instance-role"
      },
      "Action": [
        "s3:GetBucketLocation",
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::target-bucket-name"
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::source-account-id:role/instance-role"
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::target-bucket-name/*"
    }
  ]
}
```

To enforce [[appsync|security]] via Organization Service Control [[policies]] (SCPs), you could set up a deny-all strategy except explicitly allowed services. Here's an example [[SCP]] denying all actions except `ec2:AttachVolume` and `ec2:DetachVolume`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "NotAction": [
        "ec2:AttachVolume",
        "ec2:DetachVolume"
      ],
      "Resource": "*"
    }
  ]
}
```

## Performance & Reliability

Throttling limits depend on the instance type and size. To manage these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], use exponential backoff strategies when making repeated calls, especially during heavy load or [[api-gateway|errors]].

High availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] ([[dr]]) patterns involve creating standby instances in separate Availability Zones (AZs) or regions. For AZ redundancy, ensure that your application writes to multiple instances simultaneously. For regional [[dr]], maintain copies of critical data in both production and secondary regions.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls include monitoring usage metrics like volume size, snapshot size, and number of I/O requests. Calculate costs using the AWS Pricing Calculator or the [[billing|Cost Explorer]] tool.

For example, let's say you have an m5.large instance with one 40GB gp2 volume running for 730 hours per month. Using the pricing calculator, we find:

- Compute cost: $0.092 per hour × 730 hours = $67.76
- Storage cost: $0.10 per GB-month × 40 GB = $4.00
- I/O requests cost: 3.2 million I/O requests × $0.005 per 1M requests = $0.016

Total monthly cost: $67.76 + $4.00 + $0.016 = $71.776

## Professional Exam Scenarios

**Scenario 1:**
You are designing a genomics research platform that requires extremely high throughput storage for temporary data processing. The solution must be cost-effective and scalable across multiple regions. What would you recommend?

*Recommendation*: Use Instance Store-backed [[ec2]] instances for high-throughput storage requirements due to their low latency and high IOPS capabilities. Leverage Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for long-term archival needs and inter-region data replication. Implement auto-scaling groups across multiple regions to address scalability concerns.

**Answer:** Correct. This approach utilizes the strengths of both services—fast, ephemeral Instance Store for temporary data processing and durable [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for archival purposes.

**Incorrect:** Relying solely on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes would not provide the necessary IOPS performance needed for genomics research. Similarly, using only [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] might introduce unnecessary complexity and cost when faster storage is available via Instance Store.

**Scenario 2:**
Your organization wants to restrict the creation of [[ec2]] instances with [[ebs|Instance Store volumes]] to specific teams within the company. How would you achieve this using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and Service Control [[policies]] (SCPs)?

*Solution*: Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy allowing the specified teams to launch Instance Store-enabled instances. Attach this policy to relevant group(s)/user(s). Additionally, apply an [[SCP]] at the organizational level denying the [[ec2]]:RunInstances action unless instances have an associated Instance Store volume.

**Answer:** Correct. By combining [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and SCPs, you effectively limit the creation of Instance Store-enabled instances to specific teams while preventing unwanted launches outside those teams.

**Incorrect:** Implementing this restriction solely through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] may not suffice since users outside the designated teams could still launch non-Instance Store instances. Likewise, relying only on SCPs might be too restrictive, potentially blocking legitimate instance creations.