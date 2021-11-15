---
layout: post
title: Design Amazon sales rank
category: design-challenge
---

# My attempt

## Functional requirements

1. Keeping track / computing what are the most popular productts per category
   in the past 7 days
2. Popularity rank is by sale count
3. Collection, processing and exposing the data

## Non functional requirements

1. Accuracy need not be 100%
2. Scale
  1. 10M products
  2. 10k categories
  3. 100M users
3. Latency/Perf
  1. API to view the top N per category in 150ms
  2. Batching processing requirement is to be able to keep up.
4. Refresh delay 1-2hrs
5. rate of sale of products = 20% DAU = 20M users
6. rate of ingestion = 20M / day = 250 ingestions/second
7. size of individual record = [time (8B), category (8), product_id (8)] = 24B
8. rate of ingestion = 6KB/second
9. total data per day = 3600*24*6KB = 518MB ~ 1GB


## High level design

General idea:
  1. Ingestion from N sources / sales system
  2. Process/Compute the results. Pre-compute the specific query
  3. Persist the results in a low latency store
  4. Queries can read the results from there.


## TopN calculation scheme

input: for each category, (top N for 2nd to 7th day, topN for 1st day)

# Interviewer feedback

1. Data model quite late.
2. Each product could be in multiple categories.
3. Writing pseudocode.

# System design primer solution

https://github.com/donnemartin/system-design-primer/tree/master/solutions/system_design/sales_rank#design-amazons-sales-rank-by-category-feature

https://levelup.gitconnected.com/system-design-interview-distributed-top-k-frequent-elements-in-stream-2e92d63d777e

# Open questions

1. I didn't use MapReduce. Why does the solutions use it?
2. What are the other different solutions to this problem?

# High level design

## Components

### Front End Buffer

input:
    record: [time, cat_id, pr_id]
output:
    [time, cat_id, pr_id, count]

### Ingestor

input:
    output of front-end buffer
data-model:
    S3 structure input data:
        /2021-09-24

        /2021-09-23
            /compacted-category_id_2.log  # 160 MB
            /compacted-category_id_2.log
    S3 structure output data:
        /latest
            manifest = 2021-09-23
        /2021-09-23:
            final_results.log # 160 MB
logic:
    approach-1:
        algorithm:
           1. compacting a bucket:
              1. process 2021-09-23 once 2021-09-24 bucket is created
              2. map-reduce over all contents in /2021-09-23 bucket
              3. produce for each category descending sorted order [pr_id, sale_count]
              4. remove the append_log files once the /compacted files are durably stored
                 1. GC process can clean it
           2. computing 7 day top N results
              1. Pull down compacted per category for each day (160MB x 7)
              2. Merge them (<10s)
              3. Sort them (<10s)
        pros:

        cons:
           - splits by date and by category don't generalize to other use cases
           - need to wait for the bucket to reach quiescent state before performing compaction. this works for current rates, but does not work if the rate is too high.

