# Design Notes

## Improvements over Kafka

### Decoupling Peristence
Key difference is that we remove long-term persistence from the brokers.
Instead, we upload segments to S3 immediately after closing them.

Kafka currently needs to copy over historical data when redistributing
partitions.  Our active state will be tiny (just the append-only segment) so it
becomes trivial for us.

S3 also gives very cost affordable long term storage.  It becomes feasible to
have unlimited retention intervals.  Many of the S3 features are also useful,
including encryption-at-rest (compliance), lifecycle policies for TTL based
deletion, and using S3 notifications for further processing uploaded segments.

The brokers also give time-based batching for "free" in the segments that they
upload to S3.

### Autoscaling Partitions
Streamroller will auto partition a topic based on incoming message velocity.
When the velocity drops, the partition count will drop down again.  This
balances the number of segment files vs. the amount of data.  Scaling for
producer parallelism is built-in.

For consumer parallelism, we leave it to the consuming framework to shuffle out
work items to workers, if necessary.  We essentially leave this problem for the
downstream consumer and simply the brokers and configuration accordingly.

### Simple Clients
Client Implementation should be simple.  The original Kafka had the clients
manage collaborative consumption using Zookeeper.  This meant that only the Java
client was fully featured and that all non-JVM languages had poor experiences.
Kafka has made progress on this by moving more of the logic to the brokers
starting with 0.10, but we will do it right from the start.

### Avoid Touching Disk
We only need to maintain the currently segments and any that are pending flush
to S3.  We may be able to avoid touching disk entirely and rely on memory only.
The working set even for high-velocity topics should be modest.

We should use k-factor replication for durability.  The loss of data in worst
case is bounded by the amount of data pending S3 flush.

### Performance Goal
A target goal is 100x the msg/sec performance of equally priced hardware running
Kafka.

### Stable Communication Protocol
Kafka has built up a decent collection of protocol versions now.  It also looks
like they couple protocol changes to the Kafka version number, rather than use
separate versioning for the two.

Streamroller will have a separate protocol versioning.  Extreme effort will be
made to keep the protocol stable.

### No JVM
None of the operational tuning that comes with it.  No GC pauses.

## Misc Design Ideas

### Semantic Versioning
Use it from the start.

### Use Zstandard compression
This is a "maybe".  Currently there is only the canonical implementation which
is written is C.  That doesn't really play nice with Go, which has terrible Cgo
performance.


## Uncategorized
* non-latency sensitive readers can just consume S3, perhaps through S3 notifications
* operational docs should be amazing
* optimization? begin S3 upload before local segment hits time threshold to reduce time on box
