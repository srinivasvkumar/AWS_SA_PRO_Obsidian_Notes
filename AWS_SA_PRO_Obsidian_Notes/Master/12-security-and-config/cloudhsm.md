---
aliases:
- "CloudHSM"
- "CloudHSM Architecture"
- "CloudHSM Use Cases/Limitations"
---

# [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]]

- Similar to KSM, it creates, manages and secures cryptographic material or keys
- [[kms]] is a shared service. AWS has a certain level of access to the product, they manage the hardware and the software of the system
- [[kms]] uses behind the scene HSM devices
- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]] is true single tenant HSM(Hardware [[appsync|Security]] Module) hosted by AWS <span style="background-color: Red"><-- REMEMBER THIS FOR EXAM</span>
- AWS provisions the hardware for [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]] but they do not have access to it. In case of losing access to a HSM device there is no easy way to re-gain the access to it
- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]] is fully compliant with FIPS 140-2 Level 3 ([[kms]] is L2 compliant overall) <span style="background-color: Red"><-- REMEMBER THIS FOR EXAM</span>
- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]] is accessed with industry standards APIs: PKCS#11, Java Cryptography Extensions (JCE), Microsoft CryptoNG (CNG) libraries. It is not that integrated with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] by design (in comparison, [[kms]] integrates with basically every service) <span style="background-color: Red"><-- REMEMBER THIS FOR EXAM</span>
- [[kms]] can use [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]] as a **custom key store**, [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]] integration with [[kms]]

## CloudHSM Architecture

- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]] devices are deployed into a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] managed by AWS, on which we don't have visibility
- They are injected into customer managed VPCs using ENIs (Elastic Network Interfaces)
- For HA we need to deploy multiple HSM devices and configure them as a cluster
- A client needs to be installed on the [[ec2]] instances in order to be able to access HSM modules
- While AWS do provision the HSM devices, we as customers are responsible for the management of the customer keys
- AWS can provide software updates on the HSM devices, but these should not affect the encryption storage part

## CloudHSM Use Cases/Limitations

- There is no native integration with AWS services (except [[kms]]) , this means [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]] can not be used for [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] SSE
- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]] can be used for client-side encryption before uploading data to [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]]
- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]] can be used to offload SSL/TLS processing for web servers. It's economical and efficient to use cloud HSM.
- Oracle Databases from [[rds]] can perform Transparent Data Encryption (TDE) using [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]]
- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/CloudHSM|CloudHSM]] can be used to protect private keys for an Issuing Certificate Authority (CA)