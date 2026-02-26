### Advanced Architecture

At its core, an Auto Scaling Group ([[asg]]) in AWS provides the ability to automatically adjust the number of [[ec2]] instances in response to triggers like CPU usage or latency. This is achieved through the use of a Launch Configuration or Launch Template, which defines the instance type(s) and other settings.

However, [[asg|scaling policies]] within ASGs offer more advanced capabilities. These [[policies]] can be based on [[cloudwatch|CloudWatch alarms]], allowing for custom metrics and thresholds. Additionally, ASGs support coordinated scaling across multiple services using the Service Auto Scaling API, as well as integration with Step [[asg|Scaling Policies]] for more nuanced responses to changing [[cloudformation|conditions]].

Global scale ASGs require careful consideration. To distribute traffic globally, you might use Application Load Balancers (ALBs) with Network Load Balancers (NLBs) in each region, connected via [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]]. Each [[ALB]] would then have its own associated [[asg]], allowing for localized scaling while maintaining high availability.

### Comparison & Anti-Patterns

| Service          | Use Case                                                              |
|-----------------|----------------------------------------------------------------------|
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]  | Cost-sensitive workloads that can tolerate interruptions.            |
| On-Demand Instances   | Immediate availability, predictable performance, short term workloads. |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Reserved Instances]] | Steady state workloads where capacity forecasting is possible.       |

Anti-patterns include using ASGs when [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]] aren't appropriate due to potential interruptions, or relying solely on manual scaling for unpredictable workloads.

### [[appsync|Security]] & Governance

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] may involve permissions for creating or updating ASGs, along with controlling access to underlying resources. For example:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:PutScalingPolicy",
                "autoscaling:PutWarmPool",
                "autoscaling:UpdateAutoScalingGroup"
            ],
            "Resource": "*"
        }
    ]
}
```

Cross-account access can be granted using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles, assuming the necessary trust relationships are established. Organizational Service Control [[policies]] (SCPs) can enforce [[control-tower|guardrails]] around tagging, naming conventions, and other aspects of [[asg]] management.

### Performance & Reliability

Throttling limits exist for certain APIs related to ASGs. To avoid hitting these limits, implement exponential backoff strategies when making repeated calls. High availability/disaster recovery patterns often involve multiple ASGs spread across different regions or availability zones.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls can be implemented using tags at various levels (e.g., [[asg]], instance, etc.). Calculating costs involves understanding the relationship between instance types, pricing models (on-demand, reserved, spot), and any additional data transfer charges.

### Professional Exam Scenario 1

Your organization uses AWS ASGs extensively for web applications. Due to recent growth, they want to ensure their infrastructure remains performant without exceeding budget constraints. How would you address this challenge?

* Implement granular cost controls using tags and [[billing]] reports.
* Utilize [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]] where possible to reduce costs.
* Implement [[cloudwatch|CloudWatch alarms]] to trigger scaling actions based on predefined thresholds.

Incorrect answer(s):

* Manually scale instances based on demand. This doesn't allow for automatic adjustments during peak times.
* Rely solely on On-Demand Instances. While providing immediate availability, this approach could lead to unnecessary expenses.

### Professional Exam Scenario 2

Your company operates globally and needs to serve users from multiple geographical locations. They currently have ALBs set up in each region but lack a cohesive scaling strategy. What solution would you propose?

* Set up ASGs for each [[ALB]] in every region.
* Coordinate scaling across all ALBs using the Service Auto Scaling API.
* Integrate ASGs with Step [[asg|Scaling Policies]] for fine-grained control over scaling decisions.

Incorrect answer(s):

* Create one centralized [[asg]]. This wouldn't provide optimal performance since user requests would still need to traverse long distances.
* Limit scaling to specific regions only. This doesn't account for varying demand patterns in different regions.