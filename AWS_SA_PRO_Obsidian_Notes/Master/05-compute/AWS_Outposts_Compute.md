### Advanced Architecture
At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|AWS Outposts]] is a fully managed service that extends AWS infrastructure, services, APIs, and tools to virtually any datacenter, colocation space, or on-premises facility for a hybrid experience. It offers consistent functionality across both on-premises and public cloud environments. The compute aspect of Outposts includes two options: VMware Cloud Foundation (VCF) on Outposts and AWS native variants using [[ec2]] Outposts.

The following diagram illustrates an example architecture with multiple accounts, subnets, and services involved in managing an Outpost:

```mermaid
graph LR
A[Outposts