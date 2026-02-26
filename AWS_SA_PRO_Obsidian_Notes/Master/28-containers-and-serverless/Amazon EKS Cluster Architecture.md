```yaml
tags: [AWS/EKS, DataArchitecture]
status: to_review
```

### Under-the-Hood Mechanics

**Internal Data Flow**: 
- **Control Plane**: Manages the [[Kubernetes]] cluster and includes components like API server, etcd, controller manager, and scheduler.
- **Worker Nodes (Managed by [[00_Master_Links_and_Intro|EC2]] or [[Fargate]])**: Execute tasks assigned by the control plane. Communication between nodes and the control plane is through secure channels.

**Consistency Models**: 
- [[eks]] leverages the Kubernetes consistency model where changes are propagated via API calls to the [[etcd]] cluster, ensuring eventual consistency across nodes.
  
**Underlying Infrastructure Logic**: 
- **Control Plane**: Hosted by AWS in a highly available manner using multiple Availability Zones (AZs) within an [[AWS Region]].
- **Worker Nodes**: Managed by [[00_Master_Links_and_Intro|EC2]] instances or [[Fargate]]. [[00_Master_Links_and_Intro|EC2]] instances can be deployed with Auto Scaling groups for scalability, while Fargage eliminates the need to manage infrastructure.

### Exhaustive Feature List

1. **Cluster Management**:
    - Automated upgrades of Kubernetes versions
    - Secure communication between control plane and worker nodes
    - Highly available and fault-tolerant architecture
    
2. **Node Types**:
    - [[00_Master_Links_and_Intro|EC2]] Nodes: Managed by users, allowing full customization.
    - [[Fargate]] Nodes: Serverless compute layer managed entirely by AWS.

3. **Networking**:
    - [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Integration for network isolation and [[appsync|security]]
    - Service Load Balancers (ELB/NLB) integration
  
4. **[[appsync|Security]] & Compliance**:
    - [[00_Master_Links_and_Intro|IAM]] Roles for Service Accounts (IRSA)
    - Encryption at rest and in transit

5. **Monitoring & [[vpc-flow-logs|Logging]]**:
    - [[cloudwatch|CloudWatch Logs]] and Metrics
    - Prometheus and Grafana [[api-gateway|integrations]]

6. **Add-ons**:
    - Managed add-ons like AWS Load Balancer Controller, CoreDNS, Metrics Server
  
7. **Advanced Features**:
    - Network [[policies]] for fine-grained traffic control
    - Pod [[appsync|Security]] [[policies]] (PSP)
  
8. **Backup & [[00_Master_Links_and_Intro|Disaster Recovery]]**:
    - Backup and restore functionality using tools like [[Velero]]

### Step-by-Step Implementation

1. **Prerequisites**: Ensure [[00_Master_Links_and_Intro|IAM]] permissions, set up [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].
2. **Create Cluster**: Use AWS Management Console or CLI to create an [[eks]] cluster.
3. **Node Group Setup**: Configure node groups with desired instance types and ASGs.
4. **Networking Configuration**: Set up network [[policies]], [[appsync|security]] groups, and load balancers.
5. **[[appsync|Security]] Configurations**: Apply [[00_Master_Links_and_Intro|IAM]] roles for service accounts and configure encryption settings.
6. **Add-ons Installation**: Install managed add-ons using AWS Console or CLI.
7. **Deploy Applications**: Use Kubernetes YAML files to deploy applications.

### Technical Limits

- **Soft Quotas**:
  - Maximum number of worker nodes per cluster (200)
  - Maximum number of API server requests per second (1000)

- **Hard Quotas**:
  - Number of control planes per region (5)

### CLI & API Grounding

```bash
# Create an EKS Cluster
aws eks create-cluster --name mycluster --role-arn <your-role-arn> --resources-vpc-config subnetIds=<subnet-id1>,<subnet-id2>

# Describe the cluster
aws eks describe-cluster --name mycluster
```

### Pricing & TCO

**Cost Drivers**: 
- **Control Plane**: Free, but data transfer fees apply.
- **Worker Nodes**: Cost based on [[00_Master_Links_and_Intro|EC2]] instances or [[Fargate]] usage.

**Optimization Strategies**:
- Use [[00_Master_Links_and_Intro|spot instances]] for cost savings.
- Implement auto-scaling to adjust node groups as needed.
- Utilize managed add-ons to reduce overhead costs.

### Competitor Comparison

| Feature         | Amazon [[eks]]     | Google GKE    | Azure AKS   |
|-----------------|----------------|---------------|-------------|
| Managed Control Plane | Yes        | Yes           | Yes         |
| Auto-scaling    | Yes            | Yes           | Yes         |
| [[Fargate]] Support | Yes (Serverless)| No            | No          |
| [[Master/Srinivas_Notes/VPC|VPC]] Integration  | Yes            | Custom Subnets| Custom VNets|

### Integration

- **[[cloudwatch]]**: Logs and Metrics integration for monitoring.
- **[[00_Master_Links_and_Intro|IAM]]**: Role-based access control for Kubernetes clusters.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: Network isolation with subnets, [[appsync|security]] groups.

### Use Cases

1. **Microservices Architecture**: Deploying microservices in a highly available and scalable manner.
2. **[[cicd|CI/CD]] Pipelines**: Integrating [[eks]] with [[cicd|CI/CD]] tools like Jenkins or CodePipeline.
3. **Multi-tenancy Solutions**: Using Kubernetes namespaces for isolating different environments.

### Diagrams

**[[eks]] Cluster Architecture**
```
+---------------------+
|   EKS Control Plane | <--- Secure Communication
+----------+----------+       |
           |                   V
           |                +-------+  +-------+
           |                | Node1 |  | Node2 |
           |                +-------+  +-------+
           |                  /    \      / \
           |                 /      \    /   \
           V                /        \  /     \
       Kubernetes          /          \/         \
                           +----------+-----------+
                                Applications
```

### Exam Traps

1. **Misunderstanding Costs**: [[eks]] control plane is free but data transfer fees apply.
2. **Confusion between [[Fargate]] and [[00_Master_Links_and_Intro|EC2]] nodes**: [[Fargate]] simplifies management but does not support all Kubernetes features.

### Decision Matrix

| Feature              | Microservices    | [[cicd|CI/CD]]     | Multi-tenancy  |
|----------------------|------------------|-----------|----------------|
| Managed Control Plane| High             | High      | Medium         |
| Auto-scaling         | High             | High      | Low            |
| [[Fargate]] Support      | High             | High      | Medium         |

### Exam Scenarios

1. **Cluster Creation**: Scenario involving creating an [[eks]] cluster with specific node types and network configurations.
2. **[[appsync|Security]] Implementation**: Applying [[00_Master_Links_and_Intro|IAM]] roles, encryption settings, and network [[policies]].
3. **Monitoring Setup**: Configuring [[cloudwatch]] and other monitoring tools to ensure high availability.

### Interaction Patterns

- **Control Plane to Nodes**: API server communicates with worker nodes for task execution.
- **Nodes to Load Balancers**: Nodes send traffic to load balancers configured within the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

### Audit of Draft for Amazon EKS Cluster Architecture

#### Enhancement Requirements Analysis:

**1. Distractor Analysis**

Identify scenarios where Amazon [[eks]] is not the best fit, providing alternative services or solutions.

> [!danger] Distractor
- **Scenario 1: Non-Kubernetes Workloads**: For applications that do not require Kubernetes orchestration capabilities and can be managed using simpler service models like [[lambda|AWS Lambda]] for serverless functions.
  
> [!danger] Distractor
- **Scenario 2: Microservices on [[Fargate]]**: If the application is built with microservices and does not need full control over Kubernetes resources, Amazon [[ecs]] ([[Fargate]]) might be a better fit due to its fully-managed nature.

> [!danger] Distractor
- **Scenario 3: Traditional Applications**: For traditional applications that do not benefit from containerization or orchestration, [[00_Master_Links_and_Intro|EC2]] instances can provide a more straightforward and cost-effective solution.

> [!danger] Distractor
- **Scenario 4: Low-Traffic Websites**: Simple websites with low traffic might be better served by using AWS Amplify for continuous deployment of static web apps.

> [!danger] Distractor
- **Scenario 5: Batch Processing Jobs**: For batch processing tasks that do not require the orchestration capabilities provided by Kubernetes, [[aws-batch|AWS Batch]] could be a more appropriate solution due to its optimized performance and cost efficiency.

**2. The "Most Correct" Logic**

Trade-offs involved in using Amazon [[eks]] include:

> [!check] Implementation
- **Performance vs. Cost**: While [[eks]] provides high-performance container orchestration with advanced features like multi-AZ clusters for resilience, it comes at the cost of higher operational complexity and potentially increased costs due to infrastructure management overhead.
  
> [!check] Implementation
- **Flexibility vs. Complexity**: [[eks]] offers extensive flexibility in managing Kubernetes clusters within AWS but requires more administrative effort compared to fully managed services like [[ecs]] [[Fargate]].

**3. Enterprise Governance**

Integration with enterprise governance tools:

> [!warning] Quota
- **[[organizations|AWS Organizations]]**: Utilize [[organizations|AWS Organizations]] to manage multiple AWS accounts and apply consistent [[policies]] across different environments.
  
> [!warning] Quota
- **Service Control [[policies]] (SCPs)**: Implement SCPs to restrict access and enforce compliance at the organizational level, ensuring that only authorized actions can be performed on [[eks]] clusters.

> [!warning] Quota
- **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]**: Use [[ram]] for sharing resources like VPCs and [[appsync|security]] groups across multiple accounts, streamlining infrastructure management in multi-account setups.
  
> [!warning] Quota
- **[[00_Master_Links_and_Intro|CloudTrail]]**: Enable [[00_Master_Links_and_Intro|CloudTrail]] to log all API calls made against your AWS account, including those related to [[eks]]. This provides comprehensive audit trails and helps with compliance monitoring.

**4. Tier-3 Troubleshooting**

Complex failure modes:

> [!danger] Distractor
- **[[kms]] Key Policy Conflicts**: If [[kms]] key [[policies]] are not correctly configured for [[00_Master_Links_and_Intro|IAM]] roles used by the [[eks]] cluster, it can lead to permission issues that prevent proper functioning of the cluster or its components.
  
> [!danger] Distractor
- **Inconsistent State in Kubernetes Components**: Inconsistencies within the Kubernetes control plane (e.g., etcd database corruption) can result in partial failures and need expert intervention to restore consistency.

**5. Decision Matrix**

Use Case vs. Solution Table:

| Use Case                        | Recommended Service     |
|---------------------------------|-------------------------|
| Microservices orchestration      | Amazon [[eks]]              |
| Serverless Functions             | [[lambda|AWS Lambda]]              |
| Traditional Application Hosting  | [[00_Master_Links_and_Intro|EC2]]                     |
| Batch Processing                 | [[aws-batch|AWS Batch]]               |
| Static Website Deployment        | AWS Amplify             |

**6. Recent Updates**

Flag General Availability (GA) features and deprecated items for the year 2026:

> [!check] Implementation
- **GA Features**: Monitor any new GA releases such as enhanced [[appsync|security]] capabilities or improved integration with [[00_Master_Links_and_Intro|other AWS services]].
  
> [!warning] Quota
- **Deprecated Items**: Stay updated on deprecation notices to ensure that [[eks]] clusters are using current, supported versions of Kubernetes.