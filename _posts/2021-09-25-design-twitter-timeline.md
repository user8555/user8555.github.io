---
layout: post
title: Design Twitter timeline and search
category: design-challenge
---

# Functional requirements

1. Timeline: Show all tweets from followers sorted in reverse chronological order
2. Search by keyword and show the results

# Non-functional requirements

1. Mostly accurate
2. Eventual consistency is ok
3. HA
4. latency(get_timeline) = 20ms
5. latency(post_tweet) = 10ms
6. delay between post and show in timeline = 10 seconds
7. 10:1 read:write ratio
8. handle celebrities

# Scale

total_users = 1,000,000,000
dau = 20% of total_users
tweet_rate = 200,000,000 / day
tweet_size = 256 bytes to gigabytes
incoming_bandwidth = tweet_size * tweet_rate * 365 days / year

# Thinking out loud

1. caching is useful
2. precomputation and caching those results is useful
3. precomputed results can be updated reactively in near real time
4. writes to DB can publish a change in CDC stream for poster_id
5. post service servers can be partitioned per user_id. Each shard can similarly get notifications 
   for follower updates and for each follower subscribe a notification stream
6. Write rate not high. Q size should remain small
7. celebrities will have high subscribers, but that should be ok
8. write latency not affected by num subscribers

# High level design

```
USER --(1 read)--> FEED_SVC --(2 fwd)--> FEED_SVC_SHARD_LEADER --(3 send result)--> FEED_SVC --(4 return)--> USER

USER --(1 post)--> FEED_SVC --(2 fwd)--> FEED_SVC_SHARD_LEADER --(3 write_thru)--> DB_LEADER

DB_LEADER --(4 replicate)--> DB_FOLLOWER
          --(5 ack)--> FEED_SVC_SHARD_LEADER
          --(6 publish)--> CHANGE_STREAM
                           
DB_LEADER     <--(1 reload)----------------------\                           
CHANGE_STREAM <--(1 subscrbe,pull)--/--(1 push)--> FEED_SVC_SHARD_LEADER { OBSERVER --(2 apply)--> CACHED_STATE }

```
# DB design

```sql
create table user(
  user_id bigint,
  user_name varchar(128),
  
  primary key (user_id)
)

create table post(
  user_id bigint,  
  post_id bigint,
  post_time time,
  post text(10000),
  
  primary key (post_id),
)

create table followers(
  user_id bigint,
  follower_id bigint,
  
  primary key (user_id, follower_id),
)

```