## Advanced Architecture

At its core, a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]] is a managed network address translation (NAT) service provided by AWS that allows instances in private subnets to communicate with the internet in order to download patches or upload logs, while preventing incoming internet connections that might expose those instances to threats. NAT Gateways are highly available within an Availability Zone, but they do not have built-in redundancy across different AZs. To achieve high availability across multiple AZs, you should deploy a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]] in each AZ and use a single Elastic IP address for internet connectivity.

Here's a Mermaid representation of a multi-AZ [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]] deployment:
```mermaid
graph LR
A[Private Subnet 1] --1. NatGateway-- B[NAT Gateway 1]
C[Private Subnet 2] --2. NatGateway-- B
D[Internet]
B --NATed Internet Traffic--> D