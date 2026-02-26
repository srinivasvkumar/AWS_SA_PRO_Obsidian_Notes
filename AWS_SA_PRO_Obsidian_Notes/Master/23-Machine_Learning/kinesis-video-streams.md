---
aliases:
- "Kinesis Video Streams"
---

# Kinesis Video Streams

- Allows us to ingest live video data from producers
- Producers can be [[appsync|security]] cameras, smartphones, cars, drones, time-serialised audio, thermal, depth and RADAR data
- Consumers can access the data frame-by-frame or as needed
- [[kinesis]] can persist and encrypt data in-transit and at rest
- We can not access data directly via storage, only via APIs!
- Data is not stored in its original format. Data is indexed and stored in a structured way
- [[kinesis]] Video Streams integrates with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] such as [[AWS_SA_PRO_Obsidian_Notes/Master/16-other/rekognition|rekognition]] and Connect