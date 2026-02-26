---
aliases:
- "Amazon Athena"
---

# Amazon Athena

- It is a serverless interactive querying service
- We can take data stored in [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] and perform ad-hoc queries on the data paying only for the data consumed
- [[AWS_SA_PRO_Obsidian_Notes/Master/Analytics/Athena|Athena]] uses a process named **Schema-on-read** - table-like translation
- Original data in [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] is never changed, it remains in its original form. It is translated to the predefined schema when it is read for processing
- Supported formats by [[AWS_SA_PRO_Obsidian_Notes/Master/Analytics/Athena|Athena]]: XML, JSON, CSV/TSV, AVRO, PARQUET, ORC, Apache, [[cloudtrail]], [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] Flowlogs, etc. Supports standard formats of structured data, semi-structured and unstructured data
- "Tables" are defined in advance in a [[glue|data catalog]] and data is projected through when read. It allows SQL-like queries on data without transforming source data
- [[AWS_SA_PRO_Obsidian_Notes/Master/Analytics/Athena|Athena]] has no infrastructure. We don't need set up anything in advance
- [[AWS_SA_PRO_Obsidian_Notes/Master/Analytics/Athena|Athena]] is ideal for situations where loading/transforming data isn't desired
- It is preferred for querying AWS logs: [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] Flow Logs, [[cloudtrail]], [[elb]] logs, cost reports, etc.
- Can query data form [[glue]] [[glue|Data Catalog]] and Web Server Logs
- [[AWS_SA_PRO_Obsidian_Notes/Master/Analytics/Athena|Athena]] Federated Query: [[AWS_SA_PRO_Obsidian_Notes/Master/Analytics/Athena|Athena]] now supports querying other data sources than [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]]. Requires a data source connector (AWS [[lambda]])