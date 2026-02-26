**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[rds|RDS Proxy]] operates at the DB connection level, providing transparent load balancing, automatic failover, and efficient connection pooling. It supports [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] (MySQL, PostgreSQL, MariaDB, Oracle, and SQL Server) and [[aurora]] engines.

Internally, [[rds|RDS Proxy]] uses a *star schema* architecture, with each proxy connecting to one or more [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] dataclusters (*target groups*). The target groups can be spread across multiple Availability Zones (AZs) for high availability.

When a client connects to an [[rds|RDS Proxy]] endpoint, the connection is routed to the appropriate target group based on the specified default or custom routing rules. This enables global scale active-passive replication using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Multi-AZ deployments and Amazon's Global Accelerator.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service          | Use Case                                                              |
|------------------|----------------------------------------------------------------------|
| [[rds|RDS Proxy]]        | Managing connections to [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instances in high-concurrency workloads   |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]] | HTTP/HTTPS traffic only                                             |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Network Load Balancer (NLB)]] | TCP traffic only                                                     |
| [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] | Improving latency by directing traffic through optimal AWS edge locations |

Anti-pattern: Using [[rds|RDS Proxy]] as a general-purpose load balancer instead of [[ALB]] or [[NLB]].

**[[RDS_Instance_Types|3. Security & Governance]]**

To enable cross-account access, create a role in the source account with permissions to assume the destination account's [[rds|RDS Proxy]]'s instance profile. In the destination account, grant the necessary permissions to the assumed role.

Example JSON policy for cross-account access:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::source_account_ID:role/rds-proxy-access"
            },
            "Action": "sts:AssumeRole",
            "Condition": {}
        }
    ]
}
```

For centralized management, use [[organizations|AWS Organizations]] Service Control [[policies]] (SCPs) to enforce tagging, resource creation restrictions, and [[policies|permission boundaries]].

**[[RDS_Instance_Types|4. Performance & Reliability]]**

[[rds|RDS Proxy]] has built-in throttling limits to prevent overloading [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instances. If the limit is reached, [[rds|RDS Proxy]] will return an error. To handle these situations, implement exponential backoff strategies in your application code.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns include:
- Deploying [[rds|RDS Proxy]] with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Multi-AZ for automated failover
- Using Amazon's Global Accelerator for low-latency access to [[rds|RDS Proxy]] endpoints

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost control is achieved through [[billing]] metrics such as number of active connections, data processed, and number of prepared statements. Calculate costs using the AWS Pricing Calculator.

Example calculation for MySQL [[rds|RDS Proxy]]:

- Number of active connections per hour: 1,000
- Data processed per month: 1 TB
- Prepared statements per hour: 50,000

Total monthly cost = ($0.000008 x 1,000 x 24 x 30) + ($0.09 x 1) + ($0.0000004 x 50,000 x 24 x 30)

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1: A company wants to improve database performance while maintaining high availability. They currently use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] Multi-AZ deployments but have high-concurrency workloads causing connection delays.

Correct answer: Implement [[rds|RDS Proxy]] to manage connections and provide efficient connection pooling.

Incorrect answer: Use [[ALB]] or [[NLB]] since they do not support non-HTTP/HTTPS traffic.

Scenario 2: An organization needs a solution to securely share resources between accounts while enforcing governance [[policies]].

Correct answer: Create a role in the source account allowing it to assume the destination account's [[rds|RDS Proxy]] instance profile. Enforce additional [[appsync|security]] measures using [[organizations|AWS Organizations]] SCPs.

Incorrect answer: Grant full [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] access to the source account, which does not ensure proper governance.

### AI Linked Diagram
![[Screenshot 2026-01-01 at 10.23.35 PM.png]]
