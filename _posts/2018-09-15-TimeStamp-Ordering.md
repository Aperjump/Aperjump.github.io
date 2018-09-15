---
layout: post
title: "TimeStamp Ordering Concurrency Control"
description: "T/O Concurrency Control"
categories: [Database]
tags: [Database]
 - /2018/09/15/
---
## TimeStamp Ordering Concurrency Control
### Concurrency Control Approaches
- Two-phase Locking(2PL)
Determine serializability order of conflicting operations at runtime while txns execute **Pessimistic**
- Timestamp Ordering(T/O)
Determine serializability order of txns before they execute **Optimistic**
#### T/O Concurrency Control
Use timestamps to determine the serializability order of txns
if `TS(T_i) < TS(T_j)`, then the DBMS must ensure that the execution schedule is equivalent to a serial schedule where `T_i` appears before `T_j`. 
#### Timestamp Allocation
Each txn `T_i` is assigned a unique fixed timestamp that is **monotonically increasing**
- Let `TS(T_i)` be the timestamp allocated to txn `T_i`
- Different schemes assign timestamps at different times during the txn. 

Multiple implementation strategies
- System Clock
- Logical Counter
- Hybrid

**Agenda**
- Basic Timestamp Ordering
- Optimistic Concurrency Control

#### Basic T/O
Txns read and write objects without locks
Every object `X` is tagged with timestamp of the last txn that successfully did read/write:
`W-TX(X)` - Write timestamp on `X`
`R-TX(X)` - Read timestamp on `X`
We need to check timestamps for every operation:
if txn tries to access an object "from the future", it aborts and restart. 

##### Reads
`W-TS(X)` means the last write transaction on object `X`. 
If `TS(T_i) < W-TS(X)`, this violates timestamp order of `T_i` with regard to the writer of `X`(read into the future)
- Abort `T_i` and restart it with same TS

Else:
- Allow `T_i` to read `X`
- Update `R-TS(X)` to `max(R-TS(X), TS(T_i))`
- Have to make a local copy of `X` to ensure repeatable reads for `T_i`

##### Writes
If `TS(T_i) < R-TS(X)` or `TS(T_i) < W-TS(X)` 
- Abort and restart `T_i`

Else:
- Allow `T_i` to write `X` and update `W-TS(X)`
- Also have to make a local copy of `X` to ensure repeatable reads for `T_i`

#### Thomas Write Rule
If `TS(T_i) < R-TS(X)`:
- Abort and restart `T_i`
If `TS(T_i) < W-TS(X):
-> **Thomas Write Rule** : Ignore the write and allow the txn to continue
This violates timestamp order of `T_i`
Else:
-> Allow `T_i` to write `X` and update `W-TS(X)`

Basic Timestamp Ordering will generate a schedule that is conflict serializable if you do **not** use the Thomas Write Rule
- No deadlocks because no txn ever waits
- possibility of starvation for **long txns** if short txns keep causing conflicts. 

Permits schedules that are not **recoverable**

### Recoverable Schedules
A schedule is **recoverable** if txns commit only after all txns whose changes they read, commit. 
Otherwise, the DBMS cannot guarantee that txns read data that will restored after recovering from a crash. 
![Alt text](/assets/images/1537048174787.png)
### Basic I/O Performance Issues
High overhead from copying data to txn's workspace and from updating timestamps
long running txns can get starved.
- The likelihood that a txn will read something from a newer txn increases

Suffers from timestamp bottleneck. 

If you assume that conflicts between txns are **rare** and that most txns are **short-lived**, then forcing txns to wait to acquire locks adds a lot of overhead. A better approach is to optimize for no-conflict case. 
### Optimistic Concurrency Control
The DBMS creates a private workspace for each txn. 
-> All modifications are applied to workspace
-> Any object read is copied into workspace

When a txn commits, the DBMS compares its workspace write set to see whether it conflicts with other txns
If there are no conflicts, the write set is installed into the "global" database. 
OCC Phases:
(1) read phase:
track the read/write sets of txns and store their writes in a private workspace
(2) validation phase:
when a txn commits, check whether it conflicts with other txns
(3) write phase:
If validation succeeds, apply private changes to database. Otherwise abort and restart the txn. 

**Txns do not have a timestamp until they enter validation phase**
##### OCC - Validation Phase
The DBMS needs to guarantee only serializable schedules are permitted
`T_i` checks other txns for RW and WW conflicts and makes sure that all conflicts go one way(from older txns to younger txns)

We need to execute **Validation** and **Write** phase inside a protected critical section. 

When to use OCC
When number of conflicts is low.:
-> All txns are read-only
-> Txns access disjoint subsets of data

If the database is large and the workload is not skewed, then there is a low probability of conflict, so again locking is wasteful. 

When a txn commits, all previous T/O schemes check to see whether there is a conflict with concurrent txns. 
- This requires latches

If you have a lot of concurrent txns, then this is slow even if the conflict rate is low. 
### Partition-Based T/O
Split the database up in disjoint subsets called **partitions**(aka shards)
Only check for conflicts between txns that are running in the same partition. 
![Alt text](/assets/images/1537050396631.png)
Txns are assigned timestamps based on when they arrive at the DBMS
Partitions are protected by a single lock:
- each txn is queued at the partitions it needs
- The txn acquires a partitions's lock if it has the lowest timestamp in that partition's queue
- The txn starts when it has all of the locks for all the partitions that it will read/write

Partition-based T/O protocol is very fast if: 
- The DBMS knows what partitions the txn needs before it starts
- Most(if not all) txns only need to access a single partition

Multi-partition txns causes partitions to be idle while txn executes. 

