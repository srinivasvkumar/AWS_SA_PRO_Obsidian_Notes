## [[RDS_Instance_Types|1. Advanced Architecture]]

Local Zones are part of [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|AWS Outposts]] family, but unlike Outposts they deploy infrastructure within an existing metro population, not in a specific data center. They are designed to provide single-digit millisecond latency to applications that require fast response times. Local Zone resources include Amazon [[ec2]] instances, Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes, and Amazon [[Fargate]] tasks.

Local Zone architecture consists of two types of components: local and remote. Local components are physically located in the Local Zone while remote components reside in the nearest AWS Region. The connection between these components is established via AWS’s private network infrastructure called AWS Global Network.

The following Mermaid diagram illustrates the internal structure of a Local Zone:

```mermaid
graph LR