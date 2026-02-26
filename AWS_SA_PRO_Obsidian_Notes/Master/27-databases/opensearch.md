---
aliases:
- "AWS OpenSearch (Formerly known as ElasticSearch)"
- "ELK Stack"
---

# AWS OpenSearch (Formerly known as ElasticSearch)

- Is a managed implementation of ElasticSearch, an open source search solution. It is part of the ELK stack (ElasticSearch, Logstash, Kibana)
- OpenSearch is not serverless, it runs in a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] using compute
- OpenSearch is usually an alternative to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]]
- Can be used for log analytics, monitoring, [[appsync|security]] analytics, full text search and indexing, click stream analytics

## ELK Stack

- ElasticSearch (or OpenSearch on AWS): provides search and indexing services
- Kibana: visualization and dashboard tool
- Logstash: similar to [[cloudwatch]] Logs, needs a Logstash agent installed on anything to ingest data

