**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[eks]] Nodegroups are a component of Amazon [[eks]] that simplifies the deployment and management of Kubernetes worker nodes. They are essentially managed node groups that launch and terminate [[ec2]] instances in an Auto Scaling group. The following advanced architecture concepts apply to [[eks]] Nodegroups:

* **Managed Nodegroups**: Managed nodegroups provide automatic patching, maintenance, and scaling capabilities. They support [[ec2]] instance types from the latest four generations, enabling customers to take advantage of the latest hardware innovations.
* **[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]]**: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot instances]] can be used in [[eks]] nodegroups to reduce costs significantly. However, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]] can be terminated by AWS at any time if there is insufficient capacity or if the spot price exceeds the maximum price set by the user.
* **Multi-Account Strategies**: [[eks]] nodegroups can span multiple accounts using AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Resource Access Manager (RAM)]] to share resources across accounts. This enables centralized management of [[eks]] clusters while maintaining separate [[billing]] and [[appsync|security]] [[policies]].
* **Quotas**: [[eks]] nodegroups have default quotas for the number of nodegroups, nodes per nodegroup, and total nodes per region. These quotas can be increased by submitting a request to AWS Support.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

The following table compares [[eks]] nodegroups to alternative services:

| Service | Use Cases | Advantages | Disadvantages |
| --- | --- | --- | --- |
| [[eks]] Nodegroups | Simplified deployment and management of worker nodes | Automatic patching, maintenance, and scaling | Limited customization options |
| [[Fargate]] | Serverless computing without managing servers | Pay-per-use pricing model | Not suitable for long-running applications |
| Self-managed [[ec2]] instances | Complete control over worker nodes | Customizable configurations | Requires manual patching, maintenance, and scaling |

Common anti-patterns include using self-managed [[ec2]] instances when managed nodegroups would suffice and using [[Fargate]] for long-running applications.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for [[eks]] nodegroups involve granting permissions to specific actions such as creating, updating, and deleting nodegroups. Here is an example JSON policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "eks:CreateNodegroup",
                "eks:DescribeNodegroup",
                "eks:UpdateNodegroup",
                "eks:DeleteNodegroup"
            ],
            "Resource": "arn:aws:eks:us-west-2:123456789012:nodegroup/my-nodegroup"
        }
    ]
}
```

Cross-account access can be granted using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]]. For example, an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in account A can be created with a trust policy allowing account B to assume the role. Once assumed, the role can perform actions on resources in account A.

Organization Service Control [[policies]] (SCPs) can enforce [[appsync|security]] [[policies]] at the organization level. For example, an [[SCP]] can restrict the creation of [[eks]] nodegroups to specific regions or limit the number of nodes per nodegroup.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits for [[eks]] nodegroups include a maximum of five nodegroups per region and 1000 nodes per nodegroup. To avoid throttling [[api-gateway|errors]], an exponential backoff strategy should be implemented. Here is an example code snippet in Python:

```python
import time
import random

def create_nodegroup(client, max_attempts=5):
    attempts = 0
    success = False
    while not success and attempts < max_attempts:
        try:
            response = client.create_nodegroup(
                clusterName='my-cluster',
                nodegroupName='my-nodegroup',
                ...
            )
            success = True
        except Exception as e:
            attempts += 1
            print(f'Error creating nodegroup: {e}')
            time.sleep(random.uniform(1, 2))
    if not success:
        raise Exception('Failed to create nodegroup after maximum attempts')
```

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve deploying [[eks]] clusters across multiple availability zones (AZs) and regions. Multi-region deployments require using tools such as [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|latency-based routing]] or Global Accelerator.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls for [[eks]] nodegroups include setting up [[billing]] alarms and [[Budgets]] for each nodegroup. Calculation examples for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] include the following:

* Estimate the cost of running a nodegroup for one month:

```python
num_nodes = 10
cost_per_hour = 0.10 # $0.10 per hour
hours_per_month = 24 * 30
total_cost = num_nodes * cost_per_hour * hours_per_month
print(f'Total cost: ${total_cost:.2f}')
```

* Compare the cost of using on-demand instances versus [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]]:

```python
on_demand_cost_per_hour = 0.10 # $0.10 per hour
spot_cost_per_hour = 0.05 # $0.05 per hour
num_nodes = 10
hours_per_month = 24 * 30
on_demand_cost = num_nodes * on_demand_cost_per_hour * hours_per_month
spot_cost = num_nodes * spot_cost_per_hour * hours_per_month
print(f'On-demand cost: ${on_demand_cost:.2f}')
print(f'Spot cost: ${spot_cost:.2f}')
```

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
You are designing a Kubernetes infrastructure for a customer who requires high availability and low latency. The customer has two data centers in different regions. How would you design the [[eks]] infrastructure to meet these requirements?

Correct answer: Deploy two [[eks]] clusters in different regions, one in each data center. Enable cross-region replication for Kubernetes resources using tools such as Velero or Cluster API. Implement load balancing using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Network Load Balancer (NLB)]] and use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|latency-based routing]] to route traffic to the closest region.

Incorrect answer: Deploy a single [[eks]] cluster in one region and enable multi-AZ deployment within that region. This approach does not meet the requirement for low latency since traffic will need to travel between regions.

Scenario 2:
Your customer is concerned about the cost of running Kubernetes worker nodes in [[eks]]. What recommendations would you give them to optimize costs?

Correct answer: Recommend using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]] for worker nodes to reduce costs. Implement auto-scaling [[policies]] based on CPU utilization or memory usage to ensure optimal resource allocation. Set up [[billing]] alarms and [[Budgets]] for each nodegroup to monitor costs and receive notifications when spending thresholds are reached.

Incorrect answer: Recommend using on-demand instances for worker nodes. While this ensures consistent performance, it is more expensive than using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]]. Additionally, do not recommend disabling auto-scaling since it is essential for ensuring high availability and efficient resource utilization.