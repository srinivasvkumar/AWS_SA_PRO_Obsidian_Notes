**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] encryption uses Amazon's Key Management Service ([[kms]]) to encrypt [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes and snapshots. The following diagram shows an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volume being encrypted using [[kms]]:

```mermaid
graph LR
    A[EBS Volume] -- Encrypts with --> B(AES-256)