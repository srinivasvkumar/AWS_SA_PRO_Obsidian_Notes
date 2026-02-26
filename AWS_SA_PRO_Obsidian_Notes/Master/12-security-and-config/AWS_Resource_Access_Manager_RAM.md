### Advanced Architecture

At its core, AWS [[ram]] allows for the management and sharing of resources across different AWS accounts and regions within your organization. It operates at the granularity of individual resources such as Amazon [[ec2]] instance types, Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] DB instances, [[redshift|Amazon Redshift]] clusters, and Amazon License Manager resources.

The following diagram illustrates how AWS [[ram]] functions under the hood:

```mermaid
graph LR
A[AWS Account A] -- shares --> B[Resource]
B -- exists in --> C[AWS Account B]
subgraph AWS [[RAM]]
B -- registered with --> D[AWS RAM]
end