## Advanced Architecture

At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] provides developers a method to build applications that can respond in sub-10ms latency by deploying containers directly within 5G networks. This is achieved through partnerships between AWS and various 5G mobile operators. [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Zones are AWS infrastructure deployed at the edge of these 5G networks, allowing direct access to the 5G network via the [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Configure feature. The following diagram depicts an example [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] architecture:
```mermaid
graph LR
    subgraph A("AWS Account A")
        AWS_REGION((AWS Region))
        CONTAINER_REGISTRY[Container Registry]
        ECR[ECR Repository]
        
        AWS_REGION--"Hosts ECS/EKS clusters"-->APP
        CONTAINER_REGISTRY--"Store Docker images"-->APP
        ECR--"Store container images"-->APP
    end
    
    subgraph B("AWS Account B")
        WAVELENGTH_ZONE((Wavelength Zone))
        WCE["Wavelength Configure"]
        SUB_6GHZ_NETWORK(Sub 6 GHz Network)
        5G_UE[(5G User Equipment)]
    end
    
    AWS_REGION-- Send -->APP
    APP-- Publish -->CONTAINER_REGISTRY
    APP-- Publish -->ECR
    APP-- Deploy Containers -->WCE
    WCE-- Provision -->WAVELENGTH_ZONE
    WAVELENGTH_ZONE-- Connect to -->SUB_6GHZ_NETWORK
    5G_UE-- Access -->SUB_6GHZ_NETWORK
```

## Comparison & Anti-Patterns

| Service          | Use Case                                                              | Anti-pattern                                                           |
|------------------|----------------------------------------------------------------------|-------------------------------------------------------------------------|
| [[Collab_Notes_detailed/Core_Compute/Wavelength|Wavelength]]       | Applications requiring sub-10 ms latency in 5G coverage areas         | Non-5G use cases, non-mobile users, or users outside 5G coverage areas   |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]       | Content Delivery / [[api-gateway|Caching]], [[appsync|Security]], Load Balancing                | Real-time workloads, low-latency requirements                         |
| [[Git_hub_notes/certified-aws-solutions-architect-professional-main/05-compute/others|AWS Outposts]]     | On-premises hybrid connectivity                                      | Mobile use cases without 5G support                                     |

## [[appsync|Security]] & Governance

### Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]

To enable cross-account access from Account A to Account B, you must create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy in Account A, granting permissions to Account B's [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] resources:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "wavelength:AssociateWavelengthDevice",
                "wavelength:DisassociateWavelengthDevice"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:SourceVpc": "arn:aws:ec2:us-west-2:123456789012:vpc/vpc-0aabbccddeeff"
                }
            }
        }
    ]
}
```
Additionally, you may enforce centralized control using [[organizations|AWS Organizations]] Service Control [[policies]] (SCPs):
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "wavelength:*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "Null": {
                    "aws:RequestTag/Environment": "true"
                },
                "ForAnyValue:Null": {
                    "aws:RecipientAccount": [
                        "*"
                    ]
                }
            }
        }
    ]
}
```

## Performance & Reliability

[[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] has throttling limits on API calls related to [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] operations. To manage these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], implement exponential backoff strategies when handling throttled requests:

1. Initial request delay: 1 second
2. Subsequent request delays: 2\*, 4\*, ... \* 2n seconds, capped at 60 seconds

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls include per-second [[billing]] for running containers. Calculate costs as follows:

Total\_cost = Container\_cost + Data\_transfer\_cost

Container\_cost = Number\_of\_containers × Hours\_used × Price\_per\_hour

Data\_transfer\_cost = Amount\_of\_data_transferred × Price\_per\_GB

## Professional Exam Scenario 1

You need to provide a solution for a real-time [[iot]] application that requires high bandwidth and sub-10ms latency within a 5G coverage area. Which AWS services would you choose?

Correct answer: [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]], [[ecs]], ECR, [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Configure

Incorrect answer: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]], [[api-gateway|API Gateway]], [[lambda]]

Justification: The requirement for sub-10ms latency and high bandwidth makes [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] the best choice. Traditional AWS services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] and [[lambda]] do not offer the required ultra-low latency.

## Professional Exam Scenario 2

Suppose you have implemented a [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] solution for a client, but they report intermittent issues connecting their 5G devices. What should you investigate first?

1. Incorrectly configured [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering connections
2. Insufficient [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Configure capacity
3. High [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] API call rates causing throttling

First, verify if there is sufficient [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Configure capacity available. If so, check the API call rates to ensure throttling isn't occurring. Lastly, examine [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering configurations to identify any misconfigurations.