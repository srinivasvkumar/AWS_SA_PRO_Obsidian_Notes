---
aliases:
- "EFS - Elastic File System"
---

# EFS - Elastic File System

- It is a network based file system which can be mounted on Linux based instance
- Can be mounted to multiple instances at once
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] it is an implementation of the NFSv4 file system format
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file systems can be mounted in a folder in Linux based operating systems
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] storage exists separately from the lifecycle of an [[ec2]] instance
- It can be shared between many [[ec2]] instances
- It is a private service, it can be mounted via mount targets inside a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]. By default an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] it is isolated to the [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] in which was provisioned
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] can be accessed outside of the [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] over hybrid networking: [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|vpn]] or DX. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] is a great tool for storage handling across multiple units of compute
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] is accessible for [[lambda]] functions. [[lambda]] has to be configured to use [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] networking in order to use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]]
- Mount targets: provide IP addresses in the range of the [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]. For HA we should provision mount targets in every availability zones present in a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] offers 2 performance modes:
    - General Purpose: ideal for latency sensitive use cases (it is the default)
    - Max I/O: can be used to scale to higher levels of aggregate throughput. Has higher latencies
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] offers 2 different throughput modes:
    - Bursting (default): works similar to [[ebs]] GP2 storage
    - Provisioned: we can specify throughput requirements independent of the size
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] offers 3 [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]]:
    - Standard: for data that is accessed and modified frequently
    - Infrequent Access (IA): cost-optimized storage class for data that is less frequently accessed (few times a quarter)
    - Archive: for data that is accessed a few times a year
- We can automatically move data between these 2 classes using lifecycle [[policies]]

### Architecture Diagram
![[Lambda+EFS.png]]
