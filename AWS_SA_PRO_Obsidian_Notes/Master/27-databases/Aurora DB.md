
1)  [[aurora]] Multi-Master works within a Region only, it cannot be used to synchronize writes across Regions.
2) The _scaling policy_ defines the minimum and maximum number of [[aurora]] Replicas that [[aurora]] Auto Scaling can manage. 
3) [[aurora]] Auto Scaling adjusts the number of [[aurora]] Replicas up or down in response to actual workloads, determined by using Amazon [[cloudwatch]] metrics and target values.
4) By default, round robin routing algorithm is used to route requests at the target group level.
5) Consider using least outstanding requests when the requests for your application vary in complexity or your targets vary in processing capability.
6) 