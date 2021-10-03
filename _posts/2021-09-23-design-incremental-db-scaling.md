---
layout: post
title: Incremental database scaling
---

## Moving a record from one partition to another

Any move operation requires a quiscent point

A partition is a collection of records. We want to move a record R from partition P1 to P2.

Let there be a transaction partition P3. There is a SegmentMap indicating which partition holds that segment. For the transaction to be complete following changes need to be made:

1. Change SegmentMap[bytes_range] from P1.seg1 to P2.seg2
2. Create P2.seg2 if it does not exist
3. Copy data from P1.seg1 to P2.seg2
4. Delete seg1 in P1

### Approach 1

Idea: shadow paging and garbage collection

1. Create P2.seg2
2. Stop writes to P1.seg1
3. Copy from P1.seg1 to P2.seg2
4. Update SegmentMap[bytes_range] from P1.seg1 to P2.seg2
5. Restart writes that were meant for P1.seg1. They will now be re-mapped to P2.seg2

Atomicity comes from atomically updating the SegmentMap[bytes_range]. That is when the effects of the move operation become visible and the "move txn" is considered committed.

Consistency comes from pausing writes to P1.seg1 while a copy into P2.seg2 is being made and from restarting whole operation from (1) in case of partial failures.

Fault tolerance comes from being able to restart the entire operation from (1) in case of partial failures and not corrupting state anywhere with a partial failure. The orphan segments can be removed by a garbage collector since it will not have any references.

In above steps, user needs to restart the operation manually in case of partial failures because the move txn is not logged in a transaction log. If we don't want user to restart manually, add step (0) which will log the operation into the txn log and a commit_lsn which logs the lsn until which the operations have been completed.

The assumption in this approach is that the stopping the writes to P1.seg1 provides equivalent of locking P1.seg1 for any more udpates while the current move transaction is in progress. This assumption is true only if seg1 can receive writes from just one writer endpoint. This is not true in case of general purpose distributed transactions over multi-master databases but it is true for single-attach block stores.

### Approach 2 - 2PC

If partition can get writes from two different writer endpoints, then the write/delete lock cannot be held on the client and instead must be pushed down to the partition itself. This will require each partition to have a lock manager. The locks need to be replicated for locks to be retained on leader transfer between replicas. We also need a txn log like above approach. The txn log needs to be replicated.

1. Write the txn record to txn log and mark it to state 'preparing'
2. 