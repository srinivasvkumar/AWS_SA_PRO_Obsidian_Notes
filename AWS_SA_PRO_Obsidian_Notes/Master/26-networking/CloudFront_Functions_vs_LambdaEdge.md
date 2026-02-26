Advanced Architecture
---------------------

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]]

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]] are lightweight, edge-based functions that run in every [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] Edge location. They are intended for simple manipulations of HTTP(S) requests or responses, such as header modifications, URL rewrites, or request collapsing. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]] are written in JavaScript and have a maximum size limit of 128 KB.

![Mermaid diagram illustrating CloudFront Function architecture](https://mermaid.ink/img/eyJjb2RlIjoic2VwYXJhdG9y BertGCgtLS0-IENvbnRlbnQgd2UgZGF0YSAxIFwiUHJvcGlibGUgQWxsIGFib3V0LTEpICsgQyAtLT4gVGhlIEFJIGRlbXBsYXRlIGFuZCBodW1hbiBmdW5jdGlvbigpIHsKICAgIHsKICAgIHdpZHRoPSIxMDAiDQogICkgewogICsgZmlsbDpmdWwsICAgeGxpbms6YXdzCnByaW50IHByb2plY3QgPSAoZSl7czogeDIpOwoKICAgIGxhdyAqIHMsCiAgICApOwoKfQ)

### [[Lambda@Edge]]

[[Lambda@Edge]] allows you to run Node.js 12.x, Node.js 14.x, and Python 3.8 functions in [[lambda|AWS Lambda]] at the edge using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]. It enables more advanced transformations than [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]], like [[api-gateway|authentication]], content generation, or business logic execution. [[Lambda@Edge]] functions are stored across all [[lambda|AWS Lambda]]@Edge locations, but only executed when requested by an end user.

![Mermaid diagram illustrating Lambda@Edge architecture](https://mermaid.ink/img/eyJjb2RlIjoic2VwYXJhdG9y BertGCgtLS0-IENvbnRlbnQgd2UgZGF0YSAyIEVwaXMuUi5EID0gQyAtLT4gVGhlIEFJIGRlbXBsYXRlIGFuZCBodW1hbiBmdW5jdGlvbigpIHsKICAgIHsKICAgIHdpZHRoPSIxMDAiDQogICkgewogICsgZmlsbDpmdWwsICAgeGxpbms6YXdzCnByaW50IHByb2plY3QgPSAoZSl7czogeDIpOwoKICAgIGxhdyAqIHMsCiAgICApOwoKfQ)

Comparison & Anti-Patterns
---------------------------

| Service          | Suitable For                                           | Not Suitable For                                            |
| ---------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] Func. | Simple request/response manipulation                   | Complex processing, database access, or long-running tasks   |
| [[Lambda@Edge]]     | [[api-gateway|Authentication]], content generation, business logic      | Extremely high memory requirements, large codebase       |

[[appsync|Security]] & Governance
----------------------

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]]

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] may be applied to control access to specific functions:

```json
{
    "Effect": "Allow",
    "Action": [
        "cloudfront:GetFunction",
        "cloudfront:UpdateFunction"
    ],
    "Resource": "arn:aws:cloudfront::*:function/*",
    "Condition": {
        "StringEquals": {
            "aws:SourceVpce": "vpce-1234abcd"
        }
    }
}
```

Cross-account deployments can be achieved through resource sharing or manual uploads. Organizational Service Control [[policies]] (SCPs) could restrict creating [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]] within the organization.

### [[Lambda@Edge]]

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles grant necessary permissions for [[Lambda@Edge]] functions. The same cross-account and organizational [[SCP]] restrictions apply as for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]].

Performance & Reliability
--------------------------

Throttling is applied per function version or alias, with a default limit of 1000 executions per minute. To mitigate throttling, implement exponential backoff strategies with jitter, resilience4j, or similar libraries.

To ensure High Availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Disaster Recovery]], distribute your functions across multiple AWS regions and leverage [[route53]] [[route53|health checks]].

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
------------------

Granular cost controls can be implemented by applying lifecycle management [[policies]] to unused versions and aliases. Consider calculating costs based on invocation frequency and duration, e.g., for a single [[Lambda@Edge]] function:

```yaml
Total Cost = (Invocations * Execution Duration (s)) * Price
```

Professional Exam Scenarios
---------------------------

### Scenario 1

Your company operates a multinational e-commerce platform with static assets hosted on Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and distributed via Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]. To optimize performance, they want to cache frequently accessed objects for longer periods while adhering to [[appsync|security]] [[iam|best practices]]. Which solution would you recommend?

* Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]] that modify Cache-Control headers to set higher [[api-gateway|caching]] durations for selected object types.
* Utilize [[Lambda@Edge]] for modifying headers, since it supports more sophisticated logic.

Correct Answer: Option A
Justification: Given the requirement for simple manipulation of HTTP(S) requests, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]] are sufficient and recommended due to their simplicity and lower cost compared to [[Lambda@Edge]].

### Scenario 2

A media streaming company wants to perform real-time image transcoding using Amazon Elastic Transcoder. Transcoded images should be served from Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]]. Design a highly available, secure, and scalable solution.

* Create a RESTful [[api-gateway|API Gateway]] endpoint that triggers an Elastic Transcoder job upon POST requests.
* Implement a [[Lambda@Edge]] function that calls the [[api-gateway|API Gateway]] endpoint whenever a new image is uploaded.

Correct Answer: Option B
Justification: This scenario requires more advanced processing than [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront Functions]] support. Therefore, [[Lambda@Edge]] is recommended to call the [[api-gateway|API Gateway]] endpoint whenever a new image is uploaded.