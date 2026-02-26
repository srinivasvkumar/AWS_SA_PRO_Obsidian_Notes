**Advanced Architecture: Bottlerocket OS [[RDS_Instance_Types|Internals]], [[RDS_Instance_Types|Global Scale Considerations]], and 'Under the Hood' Mechanics**

Bottlerocket OS is a free and open-source Linux distribution primarily designed for container host optimizations. Its primary goals include improving [[appsync|security]], reliability, and performance in containerized workloads. The following section explains its [[RDS_Instance_Types|internals]] and [[RDS_Instance_Types|global scale considerations]].

*Bottlerocket OS Architecture Overview:*

```mermaid
graph LR
A[Container] --> B(Bottlerocket OS)
B --> C[Linux Kernel]
C --> D[Containers Runtime]
D --> E[User Namespace]
F[System Packages] --> B
G(Updates) --> F
H(Container Images) --> A