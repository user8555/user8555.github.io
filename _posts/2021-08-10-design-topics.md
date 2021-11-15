---
layout: post
title: Design topics
category: design-challenge
---

# Curios choices

## Why Cassandra/RocksDB make heavy use of concurrent data structures?

# Systems

## ZippyDB

https://engineering.fb.com/2021/08/06/core-data/zippydb/

1. What is data shuffle?
2. What novel replication choices did they make?
3. What is protocol for RG partition?

## Concurrent data structures

## UUID and other ID generation schemes

1. [ID generation services](https://medium.com/@sandeep4.verma/system-design-distributed-global-unique-id-generation-d6a440cc8e5)
2. [Timeflake](https://github.com/anthonynsimon/timeflake)
3. [New UUID formats](https://news.ycombinator.com/item?id=28088213)
4. [ULIDs](https://github.com/ulid/spec) and [ULIDs HN discussion](https://news.ycombinator.com/item?id=24636204)
5. What is my design of such a service?

## Hash functions

1. Murmur3, Blake2b ... which one is best
2. What is spectral complexity. How is it measured?

## Consistent hashing

Design and code

## Clock synchronization

1. What is clock skew?
2. What is clock drift?
3. Can time go back?
4. How much drift is possible?
5. What are some modern techniques for measuring it?
6. What is Facebook's project about?