---
layout: post
title: Why build own raft protocol implementation, not use Tikv::raft
---

1. Tikv::Raft assumes log-writes are sync, hence can only write to in memory logs. If ABS were to use it, ABS will need to write to in-memory. That means:
   1. Explicit flushes to disk external to raft protocol
   2. Logs can go back in time upon restart complicating the raft protocol
   3. Snapshot creation is sync function. That is not possible for multi-GB RGs.
2. Log index is dense logical sequence like 0, 1, 2 ...
3. Message sending to peers does not support pipelining because it is a sync function that needs to be finished inline which we clearly cannot
4. Existing implementation is optimized for throughput not latency
   1. leader processes more ready messages on an advance()