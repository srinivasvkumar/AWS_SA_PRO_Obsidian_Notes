## [[RDS_Instance_Types|1. Advanced Architecture]]

AWS Private Certificate Authority (Private CA) is a managed service that enables you to create and manage private CAs within your AWS environment. It operates in two modes: [[acm]] Private CA and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Certificate Manager (ACM)]] Public CA. The former is used for issuing certificates within your AWS account or across accounts while the latter issues publicly trusted SSL/TLS certificates.

At its core, AWS Private CA uses HashiCorp's Vault PKI secrets engine under the hood, providing a highly scalable and available solution. Each Private CA has an associated certificate hierarchy, which includes a root certificate and one or more subordinate certificates. These hierarchies can be configured globally through [[organizations|AWS Organizations]] for centralized management.

### [[RDS_Instance_Types|Global Scale Considerations]]