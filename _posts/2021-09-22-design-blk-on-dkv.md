---
layout: post
title: Layered storage systems
---

# Design block-store on top of a distributed KV-store

## Goals

1. Supports standard Block APIs
2. Availability (99.9%)
3. Durability (99.999%)
3. Optimized for mixed r/w workloads
4. IOPS @4kb -> 16000
5. latency @4kb qd=1 P99.9 -> 10ms
6. Advanced features ? (No)
   1. Snapshot
   2. Clones
   3. Multi-attach

## Clarification

1. Distributed? (Y)
2. Is KV-distributed (Y)
3. Standard read(offset, len) and write(offset, len, bytes) and flush() API? (Y)

### Clarification distributed Kv-store

1. What is its API?
   1. create_table(name) -> t
   2. insert(t, k, v) -> ()
   3. get(t, k) -> v
   4. scan(t, start_key_prefix) -> iterator<(k, v)>
2. Strongly consistent? (Y)
3. Availabilty, Durability > Block store needs
4. Perf read/write -> 20ms P50, 100ms P99.9

### Observations (Thinking)

0. How does the Block store build on top of the distributed Kv-store
1. Needs data model to further the flow

## Leveraging Kv store

## Data model

vol_0_table:
    sector_number -> content

vols:
    vol_id -> vol_table_name

vol_attachments:
    vol_id -> (csd_id, lease_renewal_time)

## Flow

def write(vol_id, offset, len, content):
    vol_table_name = kv_get(VOLS, vol_id)?
    futures = []

    for (sector, offset, len) in get_write_spltis(offset, len, SECTOR_SIZE):
        existing_content = kv_get(vol_table_name, sector)
        futures.push(apply_update(existing_content, offset, len))

    join_all(futures).await

def read(vol_id, offset, len):
    1. vol_id -> vol_table_name
    2. send reads to all the individual rows that cover the requested range
    3. get responses from all of them
    4. stitch together response
    5. return response

def attach(vol_id, csd_id) :
    v1 = kv_get(vol_attachments, vol_id)?

    if v1 == None:
        is_ok = kv_insert(vol_attachments, csd_id, my_time())
        if is_ok:
            return True
    
    v1 = kv_get(vol_attachments, vol_id)?
    wait 2*N seconds
    return kv_insert(vol_attachments, vol_id, csd_id, v1, my_time())?

def detach(vol_id, csd_id):
    stop serving

## Performance

1. Write requires a read of entire block from KV-Store
   1. Single attach. CSD can maintain a cache which is easy to keep consistent with a write-through cache
2. Writes to distributed KV store are higher latency than those of Block store
   1. Writes aren't required to be durably persisted until flush() is used
   2. Writes can be buffered in the write through buffer-cache and can be written to the storage layer in the background
   3. flush() will force sync everything to disk

