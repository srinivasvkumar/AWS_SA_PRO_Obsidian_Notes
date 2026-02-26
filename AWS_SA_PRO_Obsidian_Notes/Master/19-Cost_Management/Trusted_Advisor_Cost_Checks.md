### Advanced Architecture

At its core, Trusted Advisor is a cloud management tool that checks your environment against [[iam|best practices]] in various domains, including [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]]. The cost checks identify idle resources, underutilized instances, unattached [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes, and more. However, scaling these checks to a multi-account environment requires integration with [[organizations|AWS Organizations]] and Service Control [[policies]] (SCPs) to enforce centralized control over the usage of AWS services.

The following architecture diagram illustrates how Trusted Advisor fits into a multi-account setup:

```mermaid
graph LR
A[AWS Account #1] --> B[Trusted Advisor]
C[AWS Account #2] --> B
D[AWS Organizations] --> E(([[SCP]]))
B --> E
F[AWS Cost Explorer]
B --> F