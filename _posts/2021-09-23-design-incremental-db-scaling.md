---
layout: post
title: Incremental database scaling
---

# Moving a record from one partition to another

A partition is a collection of records. We want to move a record R from partition P1 to P2.

## Using 2PC

Let there be a transaction partition P3. There is a SegmentMap indicating which partition holds that segment. For the transaction to be complete following changes need to be made:

1. Change SegmentMap[bytes_range] from P1.seg1 to P2.seg2
2. Create P2.seg2 if it does not exist
3. Copy data from P1.seg1 to P2.seg2
4. Delete seg1 in P1

Approach 1

1. Create P2.seg2
2. Stop writes to P1.seg1
3. Copy from P1.seg1 to P2.seg2
4. Update SegmentMap[bytes_range] from P1.seg1 to P2.seg2
5. Restart writes that were meant for P1.seg1. They will now be re-mapped to P2.seg2