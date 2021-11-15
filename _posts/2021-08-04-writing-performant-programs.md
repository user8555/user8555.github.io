---
layout: post
title: Writing performant programs
categories: low-level-systems
---

A zoo of techniques are involved in writing performant programs. Here I make an attempt to categorize various techniques and organize them into a procedure that can hopefully help in performance improvement to future projects.

<!-- more -->

# Amazing data structures

## Judy trees

[Reference](http://judy.sourceforge.net/application/shop_interm.pdf)

Judy tree is a cache-optimized data structure which is designed with the sole purpose
of minimizing number of cache line fills. It is a 256-ary digital tree which does not need more than 4 cache line fills for a key width of 32 bits. Why so? [See this]( https://stackoverflow.com/questions/68734183/can-someone-explain-this-line-from-the-judy-data-structure-documentation/68735301#68735301)

Unlike a regular tree, the Judy tree splits its interior nodes by expanse and not by population. This means that it does not require any rebalancing. However, it does mean that it has all the entries it possibly can for a sub-expanse even if user never provided it (null entries) and so careful attention is given to memory optimize the data structure to make this practical.

Judy is a digital (or radix tree). In a digital tree, level0 covers 1...N bits, level1 covers N+1...2N bits, level2 covers 2N+1..3N bits and so on. If the hex value of a key is 0x487 and if N=4 bits for a (2^4=16)-ary tree, at level0 will follow pointer from offset 4, at level1 from offset 8 and at level2 from offset 7.

N above, is the order of the tree. 4 in above example. 2^order is arity of the tree.

Why are CPU caches faster than RAM? [See this](https://stackoverflow.com/questions/68735907/why-are-cpu-caches-faster-than-ram)

A straightforward digital tree is not efficient because the number of null pointers, and hence memory usage, is proportional to the order of the node and number of cache line fills is inversely proportional to order of the node.

Judy tree is fast because it compresses unused expanses, thereby minimizing memory usage and minimizing unnecessary cache line fills as it traverses down the tree. Compressing a node allows it to have higher order with low memory consumption and low tree depth.

Compression is done by 3 types of compressed node types - linear, bitmap and uncompressed.

```
Linear => {
        count of populated sub-expanses, 
        list of sub-expanses, 
        pointers to each sub-expanse
    }

Bitmap => {
        for _ in 0..8 {
            32-bit bit vector, bit is 1 if sub-expanse is populated
            pointer to vector of 32 pointers to sub-expanses,
        }
    }

Uncompressed = {
    flat array of 256 entries, each entry is pointer to corresponding sub-expanse
}
```

As population density increases, one transitions progressively from linear to bitmap to uncompressed data types.

Better than linear-probing open-addressing cuckoo hashing scheme if:
1. minimizing memory footprint is most important, even if that means trading off performance a little.
2. use case is that of a sorted set. i.e. ordered traversal, finding neighboring values, top-K etc are required
3. want low variance in performance (best vs. worst)
4. keys are not fully random. Judy is best for clustered keys

Better than B-Trees, chain-based hash maps all the time.

## Open addressing linear probing / Cuckoo hashing

[Reference](https://preshing.com/20130107/this-hash-table-is-faster-than-a-judy-array/)

Better than Judy if:
1. use case is that of a hashmap, not sorted set which would need Judy
2. keys are effectively random, not clustered or sequential where Judy beats this
3. lookup or insert is more frequent that deletes. deletes are usually done in bulk.

There are many variants of hashing. [See this](https://preshing.com/20110603/hash-table-performance-tests/)

Open addressing based schemes tend to generally do much better than Chain hashing based schemes due to CPU caching effects on modern CPUs. Cuckoo hashing in particular has worst case bound of max 2 hops for retrievals at the cost of some extra iterations to find the right spot to insert data.

If we have a read-mostly lookup heavy usage, then Cuckoo hashing will outperform other hash-tables and BTrees. There are some techniques to improve Cuckoo hashing bulk-loading performance as well. [See this](https://web.inf.ufpr.br/mazalves/wp-content/uploads/sites/13/2019/10/wscad2019.pdf)

# Relative costs

References:

| Metric                                                                                  | Value  |
| :-------------------------------------------------------------------------------------- | :----- |
| CPU Hz                                                                                  | 2.3GHz |
| Atomics clock cycles vs. Raw integer clock cycles                                       |        |
| Atomics vs. RwLock                                                                      |        |
| MD5 clock cycles                                                                        |        |
| Blake2B clock cycles                                                                    |        |
| RwLock vs. Size of critical section                                                     |        |
| L1D clock cycles vs. L2D clock cycles                                                   |        |
| L2D clock cycles vs. L3D clock cycles                                                   |        |
| L3D clock cycles vs. Main memory clock cycles                                           |        |
| Core bounce clock cycles vs. Size of critical section                                   |        |
| Main memory latency vs. NVMe latency                                                    |        |
| NVMe latency vs. HDD latency                                                            |        |
| HDD latency vs. Network within AZ latency                                               |        |
| Network within AZ latency vs. Network between regions (us-east-1 and us-west-2) latency |        |
| syscall clock cycles vs. 1 clock cycle                                                  |        |
| Pipeline flush due to memory Barriers vs. 1 clock cycle                                 |        |
| RDBMS insert/query performance                                                          |        |

# Concerns

## Being cache friendly

### Cache line bouncing across cores

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce condimentum odio at nisi pulvinar congue. Fusce urna lectus, pellentesque non magna eu, semper tempus risus

### Cache eviction

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce condimentum odio at nisi pulvinar congue. Fusce urna lectus, pellentesque non magna eu, semper tempus risus

## Minimize syscall overhead

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce condimentum odio at nisi pulvinar congue. Fusce urna lectus, pellentesque non magna eu, semper tempus risus

# Strategies

## Data oriented programming

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce condimentum odio at nisi pulvinar congue. Fusce urna lectus, pellentesque non magna eu, semper tempus risus

# How to measure this?

### Concurrent HashMap

```rust

/// Concurrent HashMap designed by bucketing

use std::collections::VecDeque;
use parking_lot::{RwLock};
use std::sync::{Arc};

struct Entry {
    key: u64,
    value: u64,
}

struct ConcurrentHashMap {
    inner: Arc<RwLock<Vec<Arc<RwLock<VecDeque<Entry>>>>>>,
}

fn hash(key: u64) -> usize {
    key as usize
}

impl ConcurrentHashMap {
    fn new() -> Self {
        Self {
            inner: Arc::new(RwLock::new(vec![Arc::new(RwLock::new(VecDeque::new())); 16])),
        }
    }
    
    fn put(&self, key: u64, value: u64) {
        let inner = self.inner.read();
        let len = inner.len() - 1;
        let hash_loc = hash(key) % len;
        
        let mut q = inner[hash_loc].write();

        for q_item in q.iter_mut() {
            if q_item.key == key {
                q_item.value = value;
                return;
            }    
        }
        
        q.push_back(Entry {
            key, value
        });
    }
    
    fn get(&self, key: u64) -> Option<u64> {
        let inner = self.inner.read();
        let len = inner.len() - 1;
        let hash_loc = hash(key) % len;
        
        let q = inner[hash_loc].write();
        for q_item in q.iter() {
            if q_item.key == key {
                return Some(q_item.value);
            }    
        }
        
        None
    }
}

```