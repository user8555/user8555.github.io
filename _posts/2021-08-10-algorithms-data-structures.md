---
layout: post
title: Algorithms and data structures
---

# B-Tree

Better than Judy if:
1. licensing issue
2. Keys/Values are neither integers nor strings

Better than Binary heap if:
1. Need O(k) top-k
2. Do NOT need O(1) get-min

Better than general Hash table if:
1. keys are not uniformly distributed

Better than open addressing hash table if:
1. low-variance insertion performance matters

# Binary heap

Better than B-Tree only if:
1. Need O(1) get-min

In all other cases B-Tree is better.
