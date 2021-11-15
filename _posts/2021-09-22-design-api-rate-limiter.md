---
layout: post
title: Design API rate limiter
category: design-challenge
---

# Clarification

1. Is rate calculated on a per node basis or across nodes? (Across)
2. Do you want to fail the API if the throttling decision could not be made? (No)
3. Support bursting? (Yes)
   1. An API is configured to 30 req/sec for a user. But user wants to burst to 100 req/sec for a bit and then come back down

# Potentially interesting considerations (Not sure yet)

1. Performance consistency tradeoff

# Goals

1. Rate limit an API across N nodes
2. Availability of API MUST NOT be sacrified if throttling decision cannot be made
3. Support bursting

# Scale

1. 1M req/sec
2. requests arrive on 1000s of machines
3. 10s of APIs
4. Throttling APIs per tenant (user/account etc.) - 10k tenants

# High level design

1. Control plane - configuring rate limiting (IGNORE)
2. Data plane - Performing the throtting decision

### Data Plane

ticker      [0,  5, 10, 15, 20, ...]
reqs_global [10, 5, 10,  5, 10, ...]
locl_local  [ 1, 1,  1,  1,  1, ...]
refreshers  [ 0, 0,  0,  0, 60, ...]

Algorithm (Sliding window)
    1. Keep a sliding window of values on a per 10 second basis
    2. Upon request, check if you exceed limit of the total request count allowed for the window
    3. When the window expires, just subtract that amount to keep aggregate up-to-date
    4. If permitted increment counter of latest window

Algorithm (Token bucket)
    1. Init
       1. Fill the bucket initially with N tokens for N reqs/second
       2. Note down the time when the bucket was filled = T1
    2. Upon request, 
       1. if bucket is not empty and decrement will not go below zero, decrement from bucket and permit request
       2. if bucket is empty, T2 = current_time. new_refresh = N * (T2-T1).to_seconds. 
          1. If new_refresh > 0, add it to bucket, note down bucket_fill_time = T2. Go to "Upon request" for another iteration of loop
          2. if new_refresh < 0, reject request. Ask it to retry random_jitter * time_left_refill_bucket

Local caching algorithm
    1. Local decision based on snapshot of last N windows + purely local state of current window
    2. The staleness and inaccuracy can be managed via keep size of window 1/2 the size of the throttler window

# Feedback

1. I cared less about boundary edge case that makes Fixed interval worse over sliding interval
2. I didn't do trade-off between Token bucket and sliding window algorithm

# Alternative solutions

Please answer why I didn't consider that solution

1. https://engineering.classdojo.com/blog/2015/02/06/rolling-rate-limiter/
2. https://medium.com/figma-design/an-alternative-approach-to-rate-limiting-f8a06cf7c94c
3. https://www.educative.io/courses/grokking-the-system-design-interview/3jYKmrVAPGQ