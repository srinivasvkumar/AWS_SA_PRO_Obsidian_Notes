**Advanced Architecture**

App Mesh is a service mesh that allows you to control and monitor traffic between microservices in your applications. It uses envoy proxy as a data plane and provides a consistent way to manage services deployed across multiple environments. App Mesh supports AWS services like [[ecs]], [[eks]], and [[lambda]], as well as any other service running in [[ec2]] instances.

Internally, App Mesh uses virtual nodes, virtual routers, and routes to direct traffic between services. Virtual nodes are attached to service instances and Envoy proxies are injected as sidecars. Virtual routers are used to route requests to appropriate virtual nodes based on rules and [[cloudformation|conditions]]. Routes define how traffic should flow from one virtual node to another.

Global scale can be achieved using [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] which improves application availability and performance by directing user traffic through the optimal regional endpoint. To achieve high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], you can deploy your services in different regions or Availability Zones and use App Mesh to route traffic between them.

**Comparison & Anti-Patterns**

| Service | Use Case |
| --- | --- |
| App Mesh | For managing and monitoring traffic between microservices within a single account or globally distributed applications. |
| [[ALB]] Ingress Controller | For exposing Kubernetes services externally using Application Load Balancer. |
| [[ecs]] Service Connect | For connecting services within an [[ecs]] cluster without configuring network details. |

Anti-pattern: Using App Mesh for load balancing at the edge of your network. Instead, consider using [[ALB]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]].

**[[appsync|Security]] & Governance**

To secure App Mesh, you can apply complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to restrict access to specific resources. Here's an example policy that grants access to a specific virtual node:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "appmeshv1.*"
            ],
            "Resource": [
                "arn:aws:appmesh:*:123456789012:mesh/my-mesh/*",
                "arn:aws:appmesh:*:123456789012:route/my-mesh/*",
                "arn:aws:appmesh:*:123456789012:virtualNode/my-mesh/*",
                "arn:aws:appmesh:*:123456789012:virtualRouter/my-mesh/*"
            ],
            "Condition": {
                "StringEquals": {
                    "appmesh:resourceType": "virtualNode",
                    "appmesh:resourceName": "my-virtual-node"
                }
            }
        }
    ]
}
```

Cross-account access can be enabled using AWS [[sts]] AssumeRole or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Role for Identity Provider. Organization Service Control [[policies]] (SCPs) can be used to enforce centralized governance across accounts.

**Performance & Reliability**

Throttling limits in App Mesh depend on the number of virtual nodes, virtual routers, and routes. If the limits are reached, you can request a limit increase. Exponential backoff strategies can be applied when creating or updating resources to handle throttled requests.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns include deploying services in different regions or Availability Zones, and using App Mesh to route traffic between them.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls in App Mesh include setting up [[billing]] alarms and [[Budgets]] to track costs. Cost calculation examples:

* $0.10 per virtual node per hour
* $0.05 per virtual router per hour
* $0.0000000002 per HTTP/HTTPS request

**Professional Exam Scenario**

Scenario 1: A company has a containerized application running on Amazon [[eks]] and wants to monitor inter-service communication. They have a requirement to implement a consistent way to manage services deployed across multiple environments.

Correct Answer: Implement App Mesh to manage and monitor traffic between microservices in the [[eks]] cluster.

Incorrect Answer: Implement Application Load Balancer to manage traffic between microservices. This solution does not provide a consistent way to manage services deployed across multiple environments.

Scenario 2: A company has a multi-account environment and wants to implement App Mesh for their microservices. They want to ensure that only specific users can create and update App Mesh resources.

Correct Answer: Apply complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to restrict access to specific App Mesh resources.

Incorrect Answer: Share the App Mesh resources between accounts. This approach does not provide fine-grained access control.