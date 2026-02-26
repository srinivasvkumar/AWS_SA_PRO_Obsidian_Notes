**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[elb]] (Elastic Load Balancing) SSL offloading is an advanced feature that allows decoupling of network traffic management from application processing by terminating SSL/TLS connections at the [[elb]] level. The internal architecture involves three main components: Elastic Network Interfaces (ENIs), [[elb]] Nodes, and [[elb]] Fleet.

- ENIs: Virtual interfaces within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] responsible for sending and receiving traffic. They provide private IP addresses for communication between instances in different subnets.
- [[elb]] Nodes: Responsible for distributing incoming requests across multiple [[ec2]] instances. They run inside Amazon's data centers, handling routing decisions based on rules configured within the load balancer.
- [[elb]] Fleet: A collection of [[elb]] nodes distributed globally or regionally.

Internally, SSL offloading works through two primary processes: certificate validation and handshake negotiation. Upon receiving a client request, the [[elb]] validates the presented certificate before initiating a TLS handshake with the backend server using its own certificate. This process enables centralized management of SSL certificates while reducing CPU cycles spent on cryptographic operations at the application layer.

Global scale can be achieved using Application Load Balancers ([[ALB]]) with [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]]. By doing so, you create a single entry point for end-users, regardless of their location, routing them to the optimal regional data center based on geographical proximity, performance metrics, and user configuration.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service          | Use Case                                                              |
| --------------- | -------------------------------------------------------------------- |
| [[elb]] + SSL Offload | Centralized SSL certificate management, high-volume workloads       |
| [[NLB]]             | Layer 4 load balancing, low latency requirements                    |
| [[ALB]]             | HTTP/HTTPS traffic, host-based routing, microservices architectures |

Anti-patterns include using [[elb]] solely for SSL offloading when [[transfer-family|other features]] like path-based routing or [[elb|session stickiness]] aren't needed, as it may lead to unnecessary costs. Also, directly exposing backend servers without proper [[appsync|security]] measures assuming [[elb]] will handle all [[appsync|security]] concerns is another common mistake.

**[[RDS_Instance_Types|3. Security & Governance]]**

For cross-account access, ensure proper [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles are set up allowing necessary actions. For instance, here's a JSON policy snippet granting `elasticloadbalancing:Describe*` permissions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:Describe*"
            ],
            "Resource": "*"
        }
    ]
}
```

To enforce organization-wide restrictions, use Service Control [[policies]] (SCPs):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "elasticloadbalancing:*",
      "Resource": ["*"],
      "Condition": {
           "StringNotEquals": {
              "aws:PrincipalArn": [
                 "arn:aws:iam::123456789012:root"
              ]
           }
      }
    }
  ]
}
```
This denies all [[elb]] actions except for the root account.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits vary depending on the type of load balancer. For Classic Load Balancers, there's a maximum of 5 concurrent connections per [[elb]] node, whereas [[elb|Application and Network Load Balancers]] support higher limits due to their dynamic scaling capabilities.

Exponential backoff strategies should be implemented when dealing with throttled requests to avoid flooding the system with retries. Here's a [[CLI__SDKs|Python example]] using the `time` module:

```python
import time
backoff_seconds = 2
for i in range(5):
    try:
        # Call your function here
        break
    except Exception as e:
        print("Request failed:", e)
        time.sleep(backoff_seconds)
        backoff_seconds *= 2
```

HA/DR patterns involve deploying multiple availability zones and regions respectively. Using Multi-AZ mode for [[elb]] ensures automatic failover in case of Availability Zone disruption.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls can be applied through tagging resources and setting up [[billing]] alerts based on these tags. Remember that each [[elb]] type has different pricing models. For example, [[ALB]] charges based on the number of active requests, while CLB charges based on the number of LPUs (Lowest Priority Units).

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
A company wants to implement SSL offloading but also needs to manage thousands of certificates. Which architecture would best suit this requirement?
Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Certificate Manager (ACM)]] along with Application Load Balancers to automate certificate issuance, renewal, and management.

Scenario 2:
An organization uses both [[organizations|AWS Organizations]] and [[elb]]. How can they restrict [[elb]] usage only to specific [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users?
Implement Service Control [[policies]] (SCPs) to deny [[elb]] actions except for the specified [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] users or roles.