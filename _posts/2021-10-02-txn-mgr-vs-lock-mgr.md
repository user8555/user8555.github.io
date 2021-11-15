---
layout: post
title: Transaction manager vs. Lock manager
categories: transactions
---

## Transaction manager

Implements policies for concurrency control for transactions to guarantee atomicity, consistency of transactions appropriate to the isolation levels

## Lock manager

Manages status of locks for various objects, upgrade/downgrade of locks, acquire/release of locks and lock policies like 2-phase locking. It is not reponsible for managing correctness of execution of transactions at all.