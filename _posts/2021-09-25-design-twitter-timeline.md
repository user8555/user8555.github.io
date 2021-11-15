---
layout: post
title: Design twitter timeline and search
category: design-challenge
---

# Functional requirements

1. Timeline
   1. Show all tweets from followers sorted in reverse chronological order
2. Search
   1. Search by keyword and show the results

# Non-functional requirements

1. Mostly accurate
2. Eventual consistency is ok
3. HA
4. latency(get_timeline) = 20ms
5. latency(post_tweet) = 10ms
6. delay between post and show in timeline = 10 seconds

# Scale

total_users = 1,000,000,000
dau = 20% of total_users
tweet_rate = 200,000,000 / day
tweet_size = 256 bytes to gigabytes
incoming_bandwidth = tweet_size * tweet_rate * 365 days / year

# 