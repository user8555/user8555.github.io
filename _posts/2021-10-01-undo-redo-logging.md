---
layout: post
title: Undo/redo logging basics
---

Redo logging is needed because:
1. Atomicity, Consistency and Durability guarantees for database updates are simpler with appends to a log
2. Higher performance for writes because there is no need to update full pages to update database indexes. Database index updates can be flushed back to disk asynchronously later via applying logging techniques.

Undo logging is needed because:
1. Redo log size is fixed forcing database pages to be updated before txn commit.
2. Older versions of records are needed for rollback & snapshot isolation
3. Query peformance is better because it does not need to linear scan redo log which is not indexed by table's fields.

## References

1. [Undo and redo log recovery procedure](https://blog.cykerway.com/posts/2018/11/18/database-undo-log-and-redo-log.html)
2. [MySQL undo logging](https://blog.jcole.us/2014/04/16/the-basics-of-the-innodb-undo-logging-and-history-system/)
3. [InnoDB](https://blog.jcole.us/innodb/)
4. [InnoDB details](https://javamana.com/2020/11/20201112235405930m.html#16__207)