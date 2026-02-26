**Advanced Architecture**

Traffic Mirroring allows you to replicate network traffic to and from Amazon Elastic Compute Cloud ([[ec2]]) instances, Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] infrastructure components, and outbound bound traffic. It operates at the interface level by creating a copy of the traffic before it is processed by the final destination. Traffic Mirroring supports both IPv4 and IPv6 traffic and can mirror traffic from one Elastic Network Adapter (ENA)-enabled ENI to another ENI or to an Amazon [[cloudwatch]] agent for monitoring purposes.

Internally, Traffic Mirroring functions as follows:

* A source ENI mirrors traffic to a target ENI through a Traffic Mirror Session.
* The source and target belong to different subnets within the same [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].
* Filters allow specific traffic types based on protocols, ports, and packet headers.
* Multiple filters per session are supported, each filter being applied in order until one matches.
* Up to 10 targets can be configured per source.

When considering global scale, there are some [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]]:

* Traffic Mirroring is only available within a single region.
* To mirror traffic across regions or between [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering connections, you need to configure inter-region VPCs.

**Comparison & Anti-Patterns**

| Service               | Use Case                                                              |
| --------------------- | -------------------------------------------------------------------- |
| Flow Logs             | Auditing, compliance, troubleshooting                                |
| [[Srinivas_Notes/VPC|VPC]] Traffic Mirroring | [[appsync|Security]] assessment, load testing, DDoS simulation                   |
| Promiscuous Mode      | Legacy applications requiring broadcast, multicast, or unknown protocols |

Anti-pattern: Using Traffic Mirroring when Flow Logs would suffice. Flow Logs provide sufficient data for most auditing, compliance, and troubleshooting tasks without consuming additional resources.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeTrafficMirrorFilters",
                "ec2:DescribeTrafficMirrorSessions",
                "ec2:CreateTrafficMirrorFilter",
                "ec2:CreateTrafficMirrorSession"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
Cross-Account Access:
- Enable necessary permissions in the source account.
- Attach an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role to the [[ec2]] instance allowing actions like `ec2:CreateNetworkInterface`, `ec2:DeleteNetworkInterface`, etc.

Organization SCPs:
Limit usage of Traffic Mirroring via Service Control [[policies]] (SCPs):
```yaml