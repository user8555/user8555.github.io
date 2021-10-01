---
layout: post
title: snapshot isolation
---

## Summary

1. Requires HLC or TrueTime
2. Does not need to read every key version to create a snapshot summary

## Snapshot isolation algorithm

Constraints:
1. Transactions don't commit in txn_no order
2. Txn does not block the world to read all the keys for its transaction. It happens gradually concurrently with other operations
3. Tolerate message delays


```
          active @           active after
       txn create time      txn create time
      +--------------+    +----------------+       
      |    |         |    |                |    
      v    v         v    v                v    
[xxxx].[xx].[xxxxxxx].[+]...................
                       ^
                       |
                 current txn

Notation:
[.] = status in {not_started, in_progress} = not done
[x] = status in {committed, aborted} = done
```

Visibility criteria for my_txn_num:
1. txn_num < my_txn_num AND
2. captured_status[txn_num] in {in_progress, not_started}

## References:
1. https://timilearning.com/posts/ddia/part-two/chapter-7/#indexes-and-snapshot-isolation
2. https://github.com/Sdaas/SnapshotIsolationDemo/blob/93987045dd709c0a81f4cef9fa33e7c221b5d22b/src/main/java/com/daasworld/Account.java#L26

## Faq

**Does snapshot isolation require globally unique sequence numbers?**

Let's say
my_txn=5
txn_state_committed = {
  k0: [1,2],
  k1: [1],
}

When my_txn=5 goes to read k1, it is [1] and it includes [1]

Then, txn=4 completed on k0. So, now:
txn_state_committed = {
  k0: [1,2,4],
  k1: [1],
}

When my_txn=5 goes to read k0, should it include [4]?

We should not include it, because original snapshot didn't have it. But we don't read every key txn summary before transaction begins, so we cannot determine that.

In fact, it is impossible to read N keys instanteously without a global lock
   
So, we can never compute this unless the txn status is centralized in a single place and if we dont want to maintain centralized status per key, because really what's the point -- that would be almost a copy of the entire database; we need to maintain this status for a number that has meaning across keys.

Hence, yes, txn_numbers need to make sense cross-keys. Translating that to distributed databases, yes, the txn_numbers need to be globally unique

**Do you actually have to visit every affected row and take a snapshot before beginning the transaction?**

No

**Does snapshot isolation require monotonically increasing txn numbers?**

Yes, as you can clearly see from above visibility rules, they rely on < and > operators.