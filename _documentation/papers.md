---
layout: page
---

## Important and Useful Research Papers
---

This document is basically a bibliography of important research papers that I know of, or have read, or that I found during research and thought that they might be useful, along with some notes that I made about them. Full breakdowns/reviews for different papers may be found elsewhere as I find them useful, but this page is just about the resources themselves.

Also note that some notes on here are possible incorrect/out-of-date. I'll try to keep them updated as I work through stuff, but in practice that will likely be secondary to writing up and working through actual Stardust problems, rather than reviewing existing literature "correctly". But if you see something wrong, please point it out to me and I'll try to get it corrected!

## Transactions, Distribution, Consensus
---

### Consensus Protocols

#### Pineapple: 
[paper](https://www.usenix.org/system/files/nsdi25-bantikyan.pdf)
#### EPaxos: 
[paper](https://www.usenix.org/system/files/nsdi21-tollman.pdf)
#### Raft
[paper](https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14)
#### FLEET
[paper](https://www.vldb.org/pvldb/vol18/p1522-fan.pdf)
#### Alvin
[paper](https://www.ssrg.ece.vt.edu/papers/opodis14-alvin.pdf)

### Transaction Design

#### Google Spanner
[paper](https://storage.googleapis.com/gweb-research2023-media/pubtools/1974.pdf)

##### Notes:
* Relies on "TrueTime":
    * Instead of measuring a single clock timestamp, you measure an interval of time `[s,e)` in which the "true" time always exists. 
    * If you measure "true time" on any two nodes `A` and `B` in the cluster, the difference between the two clocks must always be bounded to `max(B.e - A.s, A.e - B.s)`. 
    * Thus, you can assign a "real" timestamp to an operation (and therefore order everything in the system with respect to that) if you wait until your local TrueTime clock's `s` value is greater than a given `e`.
* Two layers of transactionality:
    * Low-level (within a single cluster of nodes): Consensus protocol + TrueTime coordination
    * High-level (across clusters of nodes): Two-phase-commit
* TrueTime transaction
    * Begin: Pull a truetime read, and assign the timestamp to the beginning of the interval. Any writes with higher timestamp will not be read
    * Commit: 
        * Pull a truetime read to get a commit timestamp
        * Wait until the end of the commit truetime interval has passed to ensure no other transactions have a timestamp which can overlap
        * You now have a commit timestamp( it's more complicated than that, but that's the basic idea)
* _Heavily_ dependent on accurate clocks:
    * Commit time is a function on how much skew there is between any two clocks in the system, so if the skew is high, then performance will be terrible. However, when skew is low, performance can be very good because it doesn't rely on any type of cross-node coordination to determine ordering (although it relies on consensus to ensure that data is replicated).
        * This was a bigger problem when spanner was first written than it is now, because most people didn't have access to hyper-accurate timekeeping in their clusters. But these days, sub-millisecond precision is normal. Hell, Amazon offers 100 microsecond accuraccy through PTP _for free_. And there are ways of ensuring even tighter bounds if you want to go really crazy.
* Two-Phase-Commit is tricky and kinda error-prone. Lots of ways for it to fail, which makes it tough. 
* Inherently multi-tenant (due to 2PC and cluster transactions). Transactions which don't affect a cluster don't participate, so no additional cost to multi-tenancy

#### Amazon Aurora
[paper](https://assets.amazon.science/dc/2b/4ef2b89649f9a393d37d3e042f4e/amazon-aurora-design-considerations-for-high-throughput-cloud-native-relational-databases.pdf)

##### Notes:
* Disaggregated architecture: Query execution occurs on different cluster than data storage.
* Data is written through a MySQL execution engine, which logs to a shared (distributed log). Different machines then pull the data off the log and build tables and indexes. Then, at read time the data is read from the backing tables and index nodes directly.
* Primary-secondary design, with one write-capable MySQL node, and a some read-only replicas
    * In the original paper, up to 15 read-only replicas could be managed. Possible that number has grown by now, but I kinda doubt it
* Not inherently sharded
    * The thing is built on EC2 and other AWS distributed storage products, so it just shrugs and lets the storage grow as needed
    * The read-only replicas handle the read scaling, but a single writer node limits throughput pretty severely.
* Not inherently multi-tenant
    * If you want a multi-tenant aurora, you spin up more clusters of MySQL.
    * Technically, could say that tenancy is "externally provided"


#### Alibaba PolarDB

[paper](https://www.vldb.org/pvldb/vol18/p5059-chen.pdf)

#### Microsoft Socrates

[paper](https://www.microsoft.com/en-us/research/wp-content/uploads/2019/05/socrates.pdf)

#### Calvin/ FaunaDB

[paper](https://www.cs.albany.edu/~jhh/courses/readings/thomson.sigmod12.calvin.pdf)

##### Notes:

* Relies on Raft to establish a global transaction order. 
    * This may not be ideal, because raft is leader-follower, which makes resource management harder.
    * I don't think alternative (leaderless) consensus protocols like EPaxos can establish a global order, which _probably_ violates serializability.
* Protocol limits multi-tenancy
    * All transactions for everyone has to pass through the same distributed log to be ordered correctly
    * Limits throughput to the speed of the raft implementation (which is often _much_ faster than a query engine can push anyway)
* No associated query planner


## Resource Management
---
# Efficiency

### Proactive Energy Management in Database Systems

[paper](https://hotcarbon.org/assets/2024/pdf/hotcarbon24-final111.pdf)

