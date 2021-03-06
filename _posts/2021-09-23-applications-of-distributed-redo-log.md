---
layout: post
title: Applications of distributed redo logs 
categories: distributed-logging
---

## When to use distributed redo log

You want to record the complex transaction in a distributed transaction log upfront if:

1. The txn is expensive or queued up and you want to respond to caller quickly and asynchronously apply it later
2. You need to record tentative new versions of segments if there is no automatic garbage collection scheme and need the txn coordinator to rollback the txn explicitly
3. Shadow paging copy-on-write cannot be used to make all updates visible externally atomically and consistently. In this case, the Txn log can be used by a fault tolerant txn coordinator to perform 2 phase commit on the transaction

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
2. Perform prepare phase
   1. Request X lock on P1, P2 and SegmentMap. 
   2. A move transaction cannot move an old version of data to P2 from P1. That would violate the repeatable read guarantee. Hence, the most recent committed value must be read and any further updates must be prohibited. Hence the X lock on P1.
   3. Write log records in P2 indicating creation of seg2 with the content read from P1.seg1 as initial value
   4. Write log record in P3 indicating the SegmentMap update
   5. Write log record in P1 indicating deletion of seg1
   6. Each partition writes those log records to Redo log durably.
   7. Each partition ACKs back if write to disk was successful.
3. Enter Commit phase
   1. Each partition writes a Txn commit record in each transaction.
   2. Each partition ACKs back if write is durably written to Redo log.
   3. Write txn record to txn log indicating it is committed

## Atomically consistent multi-partition operations

Both the approaches below can utilize a distributed transaction redo log to record the RG split transaction if any of the reasons mentioned [here](#when-to-use-distributed-redo-log) are true.

### Approach 1

Another approach is similar to Approach 1. Each partition supports multi-version segments. Write to a segment for partition 'i' moves it from V[i]_n to V[i]_n+1. Each partition does the write. If all writes are successful we can write a single txn record to the SegmentMap indicating that the new version of all the affected versions is now V[i]_n+1 for all 'i'. Atomically commit this txn record. If the version update is successful then the multi-partition write is successful and there will not be any torn writes.

### Approach 2

We can do similar to approach 2 above. This will force all multi-partition writes to be written to the transaction log first and adds the 2N message overhead for writing across N partitions.

## Split partition - Incremental DB scaling

### Approach 1

1. Create another RG
2. Define it as synchronous mirror of original with a filter
3. Writes to segment belonging to new RG go through original RG and new one and needs quorum from original RG and new RG.
4. Writes to segment belonging only to original RG dont go to new RG
5. Reads are forwarded as well
6. Once catchup is complete, writes/reads to segments belonging to new RG are pass through for original RG and don't make any changes there
7. Transfer ownership of half range atomically via metadata update to new RG. Keep rest in original RG
8. Forward metadata update info to both RGs, so that they stop accepting writes for invalid ranges
9. Instruct original RG to remove entries that it is not a owner of

#### Cons:
1. Requires a synchronous filtering mirror feature

### Approach 2

1. Create 2 RG clones, each with filter for different halves
2. These 2 clones will be located in the same 3 places the original RG is located. These 2 clones are shallow clones; essentially metadata specifications for filters when the clone is created and hence very fast operations. 
3. These are shallow clones pointing to the same underlying log and compacted region of original RG. Hence any data updates to the original RG will be visible to the shallow clones as well
4. Make an atomic metadata update removing the old key range mapping to old RG and mapping the new 2 halves to the 2 new RG clones instead. All writes made before this metadata update will be sent to the original RG. They will still be visible to the 2 clones as per (3). 
5. Writes after the metadata update will be sent to the 2 new RGs. Since this was an transactional metadata update, we will never see writes go to new as well as old RGs simultaneously guaranteeing correctness and consistency.

#### Pros:
1. Operation is very quick because splitting an RG is purely metadata updates, thanks to shallow filtered clones
2. Can leverage regular RG rebalancing feature to move the RGs to different places after the update.

#### Cons:
1. Requires a RG clone with filtering feature