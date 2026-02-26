**Advanced Architecture**

ROSA is a managed offering that combines Red Hat OpenShift, a container application platform, with Amazon Web Services (AWS). ROSA uses AWS Elastic Kubernetes Service ([[eks]]) under the hood and provides additional services such as automated patching, image building, and integration with AWS services like App Mesh, ECR, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]].

The architecture of ROSA consists of several components, including:

* **Data Plane**: Consists of OpenShift nodes that run containers and workloads.
* **Control Plane**: Includes OpenShift components such as API server, controllers, and etcd.
* **AWS Services**: Components include [[eks]], [[ec2]], [[ALB]], and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] used by ROSA.

ROSA supports multi-region deployments using AWS Global Services such as [[route53]], [[cloudformation]] [[cloudformation|StackSets]], and [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering. The control plane can be deployed in one region while data planes can be deployed across multiple regions. This allows for low-latency connections between the control plane and data planes.

[[RDS_Instance_Types|Global scale considerations]] include latency, network connectivity, and [[appsync|security]]. To address these concerns, ROSA uses [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]] to securely connect to ECR and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] without traversing the public internet. Additionally, ROSA uses [[transit-gateway|AWS Transit Gateway]] to interconnect VPCs across regions.

ROSA operates within the customer's AWS account, allowing full control over networking, [[appsync|security]], and [[billing]]. The customer has complete access to all underlying AWS resources, making it possible to integrate ROSA with existing tools and processes.

**Comparison & Anti-Patterns**

| Factor | ROSA | [[eks]] | [[ecs]] |
| --- | --- | --- | --- |
| Management | Managed OpenShift | Managed Kubernetes | Managed Docker Swarm |
| Workload Portability | High (OpenShift compatible) | Medium (Kubernetes compatible) | Low (Docker Swarm specific) |
| Integration with [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Other AWS Services]] | High (Integrated with [[eks]]) | Medium (Requires manual setup) | Low (Limited support) |

Common anti-patterns when using ROSA include:

* Running workloads directly on the control plane instead of using data planes.
* Using non-OpenShift compatible images or applications.
* Not properly configuring network [[policies]] or [[appsync|security]] groups.

**[[appsync|Security]] & Governance**

ROSA provides fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] through the use of OpenShift's `ClusterRole` and `RoleBinding` objects. For example, the following JSON policy grants permission to create and modify routes in a specific project:

```json
{
    "apiVersion": "rbac.authorization.k8s.io/v1",
    "kind": "Role",
    "metadata": {
        "name": "route-creator",
        "namespace": "myproject"
    },
    "rules": [
        {
            "apiGroups": ["route.openshift.io"],
            "resources": ["routes"],
            "verbs": ["create", "update"]
        }
    ]
}
```

Cross-account access can be achieved using AWS [[sts]] Assumed Role and OpenShift's OAuth tokens.

Organization [[appsync|Security]] Control [[policies]] (SCPs) can be used to restrict actions at the organization level, ensuring compliance with company [[policies]].

**Performance & Reliability**

ROSA uses throttling limits to prevent resource exhaustion and ensure fairness. These limits can be adjusted based on the number of nodes and desired performance levels. Exponential backoff strategies should be implemented when encountering [[api-gateway|errors]] due to rate limiting or exceeding quotas.

High availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] ([[dr]]) patterns involve running multiple replicas of ROSA across different regions or availability zones. Data replication and backup strategies should also be implemented to ensure data durability.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be achieved by setting up [[Budgets]] and alerts for individual projects or namespaces. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost optimization]] techniques include:

* Rightsizing instances based on workload requirements.
* Implementing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|spot instances]] for stateless workloads.
* Setting up automatic scaling rules for pod autoscaling.

An example calculation for ROSA costs would be:

* $0.10 per hour \* Number of worker nodes \* Number of hours = Total worker node cost
* $0.20 per hour \* Number of control plane nodes \* Number of hours = Total control plane cost
* Additional costs for data transfer, storage, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] used.

**Professional Exam Scenario**

Question 1: A company wants to deploy a containerized application on ROSA using OpenShift. They want to enforce strict [[appsync|security]] [[policies]] and have a separate team responsible for managing infrastructure. Which approach would meet their requirements?

Answer:

* Create an AWS organizational account structure with a separate account for ROSA.
* Set up cross-account access using AWS [[sts]] Assumed Role.
* Define [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] and roles using OpenShift's `ClusterRole` and `RoleBinding` objects.
* Implement Service Account Tokens for [[api-gateway|authentication]] and authorization.

Question 2: A company wants to deploy a containerized application on ROSA using OpenShift. They need to ensure high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] capabilities. Which approach would meet their requirements?

Answer:

* Deploy ROSA in multiple regions using AWS Global Services.
* Implement data replication and backup strategies using OpenShift's built-in features.
* Configure automatic scaling rules for pod autoscaling based on CPU usage and memory constraints.
* Monitor ROSA using AWS [[cloudwatch]] and Prometheus to detect and resolve issues quickly.