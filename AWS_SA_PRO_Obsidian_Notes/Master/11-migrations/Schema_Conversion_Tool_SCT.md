**[[RDS_Instance_Types|1. Advanced Architecture]]**

Schema Conversion Tool (SCT) is a utility that assists in converting database schemas from one database engine to another. It supports homogeneous migrations such as Oracle to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Amazon Aurora]] or PostgreSQL, as well as heterogeneous migrations like Microsoft SQL Server to MySQL.

Internally, SCT operates in three phases: Discovery, Conversion, and Validation. During Discovery, it connects to your source databases and generates a report detailing any potential issues with migrating to the target engine. The Conversion phase involves converting the schema definitions based on the findings from the discovery report. Lastly, during Validation, SCT verifies that the converted schema works correctly against the target database engine.

[[RDS_Instance_Types|Global scale considerations]] include the ability to convert schemas from multiple regions concurrently, allowing for large-scale migrations across geographically dispersed databases. However, when dealing with massive datasets or high-security environments, network latency and data privacy concerns may require setting up regional SCT instances.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service/Tool          | Use Cases                                                             |
| --------------------- | --------------------------------------------------------------------- |
| Database Migration