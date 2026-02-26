---
aliases:
- "Amazon MQ"
---

# Amazon MQ

- [[sns]] and [[sqs]] are AWS services using AWS APIs
- [[sns]] provides topics which are one to many communication channels, [[sqs]] provides queues which are one to one communication channels
- Both [[sns]] and [[sqs]] are public services, HA and AWS integrated
- Larger organization might already use on-premise messaging systems, which are not entirely compatible with [[sns]] and [[sqs]]
- Amazon MQ is an open-source message broker, is a managed implementation of Apache ActiveMQ
- It supports the JMS API and protocols such as AMQP, MQTT, OpenWire and STOMP
- It provides both queues and topics
- It uses message broker services which can be single instance (test, dev) and HA pair (Active/StandBy for production)
- Unlike [[sqs]] and [[sns]], Amazon MQ is not a public service, it runs in a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]
- It does not have native integration with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] in the same way as [[sns]]/[[sqs]]
- Amazon MQ considerations:
    - By default we should chose [[sns]]/[[sqs]] for newer implementation
    - We should use Amazon MQ if we migrate from an existing system with little to no application change
    - We should use Amazon MQ if we need APIs such as JMS or protocols such as AMQP, MQTT, OpenWire, STOMP
    - Amazon MQ requires the appropriate private networking