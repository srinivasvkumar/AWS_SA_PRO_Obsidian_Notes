**[[fsx|FSx for OpenZFS]]: Advanced Architecture**

[[fsx|FSx for OpenZFS]] is a fully managed, high-performance file system that provides shared access to data using the OpenZFS open-source file system. It offers several advanced features such as data compression, snapshots, replication, and data tiering. The architecture of [[fsx|FSx for OpenZFS]] consists of three main components:

* **Storage servers:** These are the storage nodes that host the [[fsx|FSx for OpenZFS]] file systems. They provide the underlying infrastructure for storing and managing data. Each storage server runs a Linux operating system and has one or more SSD devices attached to it.
* **Metadata database:** This is a distributed database that stores metadata information about the [[fsx|FSx for OpenZFS]] file systems. It uses a distributed consensus protocol to ensure strong consistency and fault tolerance.
* **Access point:** This is a Network File System (NFS) or Server Message Block (SMB) endpoint that allows clients to connect to an [[fsx|FSx for OpenZFS]] file system. Access points can be configured to allow access from multiple VPCs or IP addresses.

At the global level, [[fsx|FSx for OpenZFS]] supports multi-Region deployments for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] purposes. Data is replicated across Regions using low-latency, high-throughput connections provided by AWS' Global Accelerator.

[[fsx|FSx for OpenZFS]] also supports integration with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] such as [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]], Amazon [[cloudwatch]], and AWS Key Management Service ([[kms]]). These [[api-gateway|integrations]] enable features like automated backups, monitoring, and encryption of data at rest.

**[[fsx|FSx for OpenZFS]]: Comparison & Anti-Patterns**

| Feature | [[fsx|FSx for OpenZFS]] | Amazon [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] | Amazon [[fsx]] for Windows File Server |
| --- | --- | --- | --- |
| File system type | OpenZFS | NFSv4.1/NFSv4.0 | SMB 2.1/3.0/3.1.1 |
| Performance mode | Throughput optimized | Latency optimized | Throughput optimized |
| Maximum throughput | 24,000 MiB/s | 10,000 MiB/s | 12,000 MiB/s |
| Multi-Region support | Yes | No | No |
| Encryption at rest | Yes (using [[kms]]) | Yes (using [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] encryption) | Yes (using BitLocker) |
| Snapshots and clones | Yes | No | No |

Anti-patterns include using [[fsx|FSx for OpenZFS]] when low latency is required, as its performance is optimized for high throughput. Additionally, [[fsx|FSx for OpenZFS]] does not support NFSv3, which may be a requirement for some legacy applications.

**[[fsx|FSx for OpenZFS]]: [[appsync|Security]] & Governance**

To enforce [[appsync|security]] and governance [[iam|best practices]], [[fsx|FSx for OpenZFS]] supports fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]. For example, you can restrict access to specific directories within a file system based on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles. Here's an example JSON policy that grants read-only access to a directory named `confidential`:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "[[fsx]]: