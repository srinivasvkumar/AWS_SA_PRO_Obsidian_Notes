## [[RDS_Instance_Types|1. Advanced Architecture]]

[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] is a managed service that enables secure connectivity between client devices and AWS resources. It uses OpenVPN protocol and provides SSL-based connections. The following diagram shows its architecture:

```mermaid
graph LR
subgraph Client VPN Endpoint
Server(Server) -- 1. Server Certificate --> Authority(CA)
Server -- 2. Security Policy --> Authentication[Authentication]
Server -- 3. DNS Settings --> Route53[Route53]
end
subgraph Client
Client_Device((Client Device))
Client_Device -- 1. Username & Password --> Authentication
Client_Device -- 2. Client Certificate --> Server
end
subgraph AWS Resources
Subnet_1[Subnet 1]
Subnet_2[Subnet 2]
Endpoints([Client VPN Endpoint])
Subnet_1 -- 1. Attach --> Endpoints
Subnet_2 -- 2. Attach --> Endpoints
Security_Group([Security Group])
Endpoints -- 3. Authorize --> Security_Group
Subnet_1 -- 4. Attach --> Security_Group
Subnet_2 -- 5. Attach --> Security_Group
S3[S3 Bucket]
Endpoints -- 6. Access --> S3
EC2[EC2 Instance]
Endpoints -- 7. Access --> EC2
]
end
Client -->|8. Connects to|--- Client_Device
Client_Device -->|9. Establishes SSL-based Connection|--- Endpoints
Endpoints -->|10. Routes Traffic|--- Subnet_1
Endpoints -->|10. Routes Traffic|--- Subnet_2
Subnet_1 -->|11. Accessible by|--- S3
Subnet_2 -->|11. Accessible by|--- EC2
```

Internally, [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] creates an OpenVPN server using [[ec2]] instances in the [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] Endpoint. The endpoint can be associated with subnets in one or more AWS accounts and regions. This allows global scale and cross-region connectivity. The endpoint has a [[appsync|security]] policy that defines [[api-gateway|authentication]] methods such as certificate-based, password-based, or multi-factor [[api-gateway|authentication]].

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

The following table compares [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] with other [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] services in AWS:

| Service | Use Case |
|---|---|
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/03-networking/vpn|Client VPN]] | Securely connect remote clients to AWS resources. |
| Site-to-Site [[Srinivas_Notes/VPN|VPN]] | Securely connect two networks over the internet. |
| AWS Managed [[Srinivas_Notes/VPN|VPN]] | Provides a managed version of Site-to-Site [[Srinivas_Notes/VPN|VPN]]. |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]] | Provides dedicated network connection between customer premises and AWS. |

Anti-patterns include using [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] to connect on-premise networks without considering Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] or Direct Connect. Another mistake is not configuring proper [[api-gateway|authentication]] methods and encryption settings in the [[appsync|security]] policy.

## [[RDS_Instance_Types|3. Security & Governance]]

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] involve allowing specific IP addresses or CIDR blocks to access the [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] endpoint. For example:

```json
{
    "Effect": "Allow",
    "Principal": "*",
    "Action": "vpn-connection:*",
    "Resource": "*",
    "Condition": {
        "IpAddress": {
            "aws:SourceIp": ["192.168.0.0/16"]
        }
    }
}
```

Cross-account access requires creating a Customer Gateway in another account and then creating a Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connection from the [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] Endpoint to the Customer Gateway.

Organization Service Control [[policies]] (SCPs) can enforce restrictions on [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] usage across multiple accounts. For instance, an [[SCP]] can prevent the creation of [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] endpoints in non-production environments.

## [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits in [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] include maximum number of concurrent connections, maximum [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] sessions per user, and maximum [[api-gateway|authentication]] attempts. To handle throttling, implement exponential backoff strategies in client applications.

HA/DR patterns include deploying [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] endpoints in multiple regions and configuring [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and DNS failover. Additionally, configure route propagation to ensure traffic flows through the optimal endpoint based on real-time performance metrics.

## [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls in [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] involve monitoring and optimizing the following components:

- Data processing fees: charged based on data transferred through the endpoint.
- Number of concurrent connections: manage the limit based on actual usage.
- Public IP address allocation: enable/disable public IP addresses based on requirements.

Calculating costs involves estimating the number of concurrent connections, data transfer, and duration of usage. For example, if a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] endpoint has 100 concurrent connections, generates 1 TB of data transfer in a month, and runs for 30 days, the estimated monthly cost would be:

- $0.10 per hour * 24 hours/day * 30 days = $72 for compute resources.
- $0.09 per GB for data processing fees ($0.09 * 1024 GB) = $92.16.
- Total cost: $72 + $92.16 = $164.16.

## 6. Professional Exam Scenario

Scenario 1: A company wants to allow employees to connect to their AWS environment remotely. They have 500 users and require strong [[api-gateway|authentication]] methods. Which solution meets these requirements?

Correct Answer: Implement a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] endpoint with a [[appsync|security]] policy requiring multi-factor [[api-gateway|authentication]] and associate it with subnets containing their AWS resources.

Incorrect Answer: Using Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] since it does not support remote client connectivity.

Scenario 2: A startup needs to minimize costs while providing secure access to their AWS resources for developers working remotely. They currently have 50 developers but expect to grow rapidly. Which solution meets these requirements?

Correct Answer: Implement a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Client VPN]] endpoint with a [[appsync|security]] policy requiring password-based or certificate-based [[api-gateway|authentication]] and monitor usage closely to adjust throttling limits accordingly.

Incorrect Answer: Using Direct Connect or Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] since they do not provide cost-effective solutions for small-scale remote connectivity.