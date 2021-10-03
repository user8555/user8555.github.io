---
layout: post
title: Transactionally consistent distributed secondary indexes
---

## Landscape

![https://whimsical.com/distributed-system-strategies-KqZAphpqSQyp7FhZ1cMJcN](/assets/Distributed%20system%20strategies.png)

## Global log

Designing a fault tolerant log is at the heart of many applications. It can be used to build queues like Kakfa, stream layer for Azure storage, transaction log for FaunaDB and so on.

If the log can fit in 1 node from space and performance perspective, then the best approach is to have N replicas of an on-disk ring-buffer and use them to write your transaction logs.

If the log cannot fit in 1 node, then the log needs to be parititioned across many nodes and each parititon can be replicated like above approach.

If you have parititioned the log, decide if you care about a global order of all records in the log. 

If you do not care about a global order, then it's easier. This is the approach Kafka, SQS FIFO takes. Within each parition the append-log can have its own sequence number and a unique record can be fetched via (partition_id, sequence_number). A client reading from the log will read from all the parititions and merge the various logs as per its own business logic akin to merge sort algorithm.

If you do care about a global order and have multiple parititons, unfortunately this is a hard problem and the solution comes at a cost. Fundamentally it boils down to figuring out a deterministic scheme to order the order the records from various partitions such that each reader of the logs decides on the same global order. 

Since the logs are parititioned and the logs are inserted by many writers, it is impossible for the log record itself to specify a global sequence number.

Let's say there are 2 readers and 2 partitions to read from. Let's say reader-1 got a record from both partitions and it orders them as partiton-2-record, partition-1-record. Let's say reader-2 only was able to successfully read partition-1-record and got a network error reading from partition-2 due to a network partition. If reader-2 processes partition-1-record immediately it will be in violation of the order determined by reader-1 and consequently we don't have a global log anymore. 

What this means is that reader-2 needs to be able to determine if there is any record from partition-2 that it should wait for. And it needs to do this with contacting reader-1. The only way to do this is for reader-2 to wait for a message from partition-2 indicating that it does not have any records that it should consider. Until that message arrives reader-2 will be left wondering. 

So, now we see that each reader needs to get messages from each partition to determine if they have any records for that "batch" to consider. We need to determine an arbitrary batch duration because the partitions need to communicate with each reader at a certain cadence to communicate this decision. This leads us to the observation that the only way to have such a global log across partitions is to have a "ticker" or "epoch number" that increments every so often driving a global clock forward and telling the readers what messages they have from each partition for that epoch. It is impossible for messages to arrive at the same instant to all readers. But we can use the epoch number as essentially a substitute of time. i.e. epoch=1 => time=1, epoch=2 => time=2 and so on. If the partition does not have anything for that epoch, it still needs to communicate an empty set to communicate this decision and the reader needs to wait for 1 message from each partition for that epoch. 

A global log with global order thus requires a global ticker and events needs to be generated every tick and system can only go as fast as the duration of 1 tick. These systems have a lower bound latency of 1 tick and are not suitable for low latency systems if the tick duration is too big for their low latency use cases. As an example, building a globally distributed block store with <1ms latencies is impossible using a global log approach each reader will need to consume N partition messages per millisecond. The overall message cost here is N*M messages/second where N = num partitions and M = num readers.

