## Advanced Architecture
At its core, Sticky Sessions in Elastic Load Balancing ([[elb]]) uses a hash-based algorithm to bind a user session to a specific backend server. This ensures that all requests from a particular session go to the same server, providing session persistence. However, there are more intricate details to consider when designing large-scale systems using Sticky Sessions.

### Hash Algorithm
The [[elb]] generates a hash based on the source IP address or an application-generated cookie value. The resulting hash is used to select a backend server for handling client requests. It's important to understand how this algorithm works because it impacts scalability and performance.

### Global Considerations
When working at a global scale, you might need to distribute users across multiple regions while maintaining session affinity. This can be achieved by combining Application Load Balancers ([[ALB]]) with Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|latency-based routing]]. In such cases, sticky sessions should only be enabled within each individual region due to the [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] of cross-region load balancer attributes synchronization.

## Comparison & Anti-Patterns
Comparing Sticky Sessions with alternative services like Amazon [[dynamodb|DynamoDB Accelerator (DAX)]] or Amazon [[elasticache]] helps us identify when not to use Sticky Sessions.

| Service          | Use Case                                                              |
|------------------|-----------------------------------------------------------------------|
| Sticky Sessions | Stateless applications requiring simple session affinity             |
| DAX              | High-performance, in-memory [[api-gateway|caching]] for Amazon [[dynamodb]] workloads   |
| [[elasticache]]      | General-purpose in-memory data store for [[api-gateway|caching]] and messaging       |

Anti-patterns include using Sticky Sessions for long-lived connections or stateful applications without considering potential single point of failure risks.

## [[appsync|Security]] & Governance
Implementing complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] requires careful planning. Here's a sample JSON policy granting [[ALB]] access to an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "elbv2:DescribeLoadBalancers",
                "elbv2:ModifyLoadBalancerAttributes",
                "elbv2:DeregisterTargets",
                "elbv2:RegisterTargets",
                "elbv2:DeleteListener",
                "elbv2:CreateListener"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
Cross-account access can be established through [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering or [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] endpoint interfaces, but keep in mind that stickiness settings aren't synchronized between accounts. Organizational Service Control [[policies]] (SCPs) can enforce restrictions on creating or configuring [[elb]] resources.

## Performance & Reliability
Throttling limits vary depending on the type of load balancer. For example, Network Load Balancers have higher request handling rates than Application Load Balancers. To prevent being throttled, implement exponential backoff strategies during transient [[api-gateway|errors]].

HA/DR patterns depend on the application requirements. Multi-region active-active setups with localized ALBs per region can provide high availability while minimizing latency. Alternatively, using a centralized [[ALB]] with [[elb|cross-zone load balancing]] may simplify management at the expense of increased latency.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls involve monitoring usage metrics like consumed GB-seconds or number of requests. By calculating expected costs based on these metrics, you can optimize your architecture accordingly. For instance, downsizing underutilized instances or adjusting target tracking [[policies]] could help minimize expenses.

## Professional Exam Scenarios

### Scenario 1
Your organization operates a multi-account environment with strict [[appsync|security]] [[policies]] enforced through Organization SCPs. Your application relies heavily on Sticky Sessions for session persistence. How would you ensure proper functioning while adhering to the [[appsync|security]] [[policies]]?

Correct Answer: Implement a combination of [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles with fine-grained permissions scoped to specific [[elb]] resources. Ensure that any changes made to the load balancer configuration are done via these roles.

Incorrect Answers: Using cross-account access without proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], directly attaching SCPs to individual accounts instead of enforcing them at the organization level, or neglecting to restrict access to [[elb]] resources altogether.

### Scenario 2
Your company experiences sudden spikes in traffic causing occasional throttling issues with your Application Load Balancer utilizing Sticky Sessions. What measures can you take to improve reliability without compromising session affinity?

Correct Answer: Employ exponential backoff strategies during transient [[api-gateway|errors]], monitor usage metrics closely, and adjust the target tracking policy as needed. Additionally, consider scaling up the underlying [[ec2]] instances to handle higher loads.

Incorrect Answers: Ignoring the issue hoping it will resolve itself, disabling sticky sessions entirely, or migrating to another load balancer type without evaluating whether it meets the application's needs.