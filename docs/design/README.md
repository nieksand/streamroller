* brokers only hold actively mutating (append) segments
* roll on to blob store (S3) asap
* k-factor replication, don't touch disk.
* client implementation must be simple
* streams should be auto partitioning based on current write volume 
* goal is to support massive RPS at very low cost
* brokers give free arrival-based batching (to S3)
* non-latency sensitive readers can just consume S3, perhaps through S3
  notifications
* getting durable state of brokers big win over Kafka.  Drops cluster cost,
  simplifies scaling
* operational docs should be amazing
* optimization? begin S3 upload before local segment hits time
  threshold to reduce time on box
