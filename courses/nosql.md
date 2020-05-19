# NoSQL 
*Notes on 157c taught by S Kim*

question at (3, 17) (3, 25)

*Part 1*
## Why?
* More suitable to solve a different set of problems relational systems solve
    * data with little structure
    * a dataset that quickly grows (horizontal scaling)

## Characteristics
* Scalability
    * horizontal - add more nodes
    * vertical - build on top of existing nodes
* Replication and eventual consistency 
    * eventual consistency - all writes will eventually propagate to all
     nodes such that all nodes will be consistent with each other
* Replication methods
    * master slave - writes on master only, optional read from secondaries
    * peer to peer - reads/writes on any node
* Partitioning 
    * data is partitioned such that portions of data belong only to one
     partition
    * allows horizontal scaling
    * MongoDB - shard
* High performance data access 
    * hash partitioning - value found by hashing the partition key
    * range partitioning - value found by checking if partition key
     falls in a partition key range
        * pro: easy range scan
        * con: hot spots 
* Schema flexibility: schema on read (RDBMS is schema on write)
* Less powerful query language

## Data Models
How databases organize data

> Aggregate 
> - a unit composed of a collection of related objects
> - unit for data manipulation and management of consistency
> - update aggregates with atomic operations
> - communicate in terms of aggregates
> - unit of replication and sharding - eases process of operating a cluster
> (related data belong in an aggregate, an aggregate belongs to a node
> ==> easier to manipulate that unit of data)
> - easier for application programmers (no more impedance mismatch)

#### Main NoSQL Data Models:
* key-value 
    * strongly aggregate-oriented
    * aggregate is opaque to the DB
    * aggregate is accessed as a whole based on its key
* document 
    * database is aware of the internal structure of aggregate
    * means we can retrieve part of aggregate rather than whole
    * means database can create indexes based on aggregate contents
* graph
    * aggregates not ideal for data with complex relationships (FB)
    * emphasis on relationships
* wide-column
    * two level map: key -> column key -> column value
    * columns are units of access
    * row vs column oriented

## Distribution Models
* Sharding - partition data and put partitions on different nodes
    * allows for horizontal sharding
    * improves r/w performance
    * data accessed together is stored on the same node
    * keep load even by distributing aggregates strategically
* Replication - make copies of data over multiple nodes
    * master-slave 
        * good for read heavy application, even if master is down, slaves
         can still be read from and a slave can become a new master
        * consistency is an issue when different clients read from
          different nodes data that has yet to be propagated
        * bad for write-heavy (only master handles writes)
    * peer-to-peer
        * consistency issue when concurrent writes update the same data (2
         replicas of a data)
         * inconsistency solution: 
            * replicas coordinate to avoid conflict
            * cope with inconsistent writes
    * inconsistency in master-slave < inconsistency in peer-to-peer
       
## Consistency
#### Update Consistency
* write-write conflict: 2 people update same data at same time (increases
 with replication)
* Maintaining consistency: 
    * pessimistic - preventative: use locks (poor throughput, but safe/consistent)
    * optimistic - allow conflicts, but detect and resolve (good throughput
    , but unsafe/inconsistent)
#### Read Consistency
* logical consistency: ensuring different but logically related items make
 sense together
* logically inconsistent read-write conflict: p reads in middle of m's write
* replication consistency: ensuring same data item has same value when read
 from different replicas
* read your write consistency: once update is made, you are guaranteed to
 continue seeing the update
#### Relaxing Consistency
* inconsistency window: portion of time in which inconsistency is present
* eventual consistency: despite inconsistency window, if no further updates
, all nodes will eventually be updated to the same value

## CAP Theorem
States no distributed system can strongly support more than 2 of the three
> **C**onsistency - system appears as a single node (every read returns the
 most recent write)
>
> **A**vailability - all non-failing nodes can be written/read from
>
> **P**artition Tolerance - system will function in spite of network partition

Where the real choice is between (A) and (C) as a distributed system cannot
 give up (P). 

* CP
    * ex. Master slave replication (MongoDB)
* AP
    * ex. peer to peer replication (Cassandra)
    
## ACID Properties
In relation to transactions
> **A**tomicity - all or nothing, no partial failure
>
> **C**onsistency - begin and end in a valid state
>
> **I**solation - concurrent transactions are not entangled
>
> **D**urability Tolerance - if transaction succeeds, changes persist

## Consistency in CAP vs ACID

* Consistency (CAP) != Consistency (ACID)
    * Consistency in CAP is more concerned with the data being read 
    * Consistency in ACID is more concerned with the validity of data being
     written (in relation to the rules defined)
* Consistency (CAP) ~= Atomicity, Isolation (ACID)
    * Isolation (ACID) relates to data being read, describes a consistency
     model in which transactions appear to have been executed in serial order
    * Atomicty (ACID) standard (in the sense of all or none) for
     classifying a transaction as a success of failure. In a similar sense
     , a distributed system is only "consistent" if a change is propagated
      to all nodes successfully.
      
## Versioning -tbc
* concurrency control methods: lock vs versioning
* version stamp - a field that changes every time the underlying data in
 the record changes 
    * good for single server or master slave replication (single authority
     over data)

*Part 3*
## Replication in MongoDB
Way of keeping identical copies of data on multiple servers
> replica set - group of servers with one primary and multiple secondaries
>
> primary - source of truth at given moment for the replica set
> - only server that can be written to
>
> secondary - data carrying non primary that replicates data from other
> members via replication chaining as fast as possible
> - can be read from specifying read preference
>

#### OPLOG
* capped collection
* how replication is handled: each S contains copies of all DBs and replays OP
* if S is too far behind P (in event of down time or more writes than it can
 handle), S will become stale
* stale S cannot catch up to every op as it would be skipping some ops
* stale S solution: attempt to replicate from each member of the set in turn
 to find an OPLOG long enough to replicate --> if failure, fully sync from
  recent backup
  
#### Member Configuration Options
> Arbiters
> - non data bearing
> - only used when we have even number of nodes to break tie in election
> - always opt for data bearing node when possible
>
> Priority
> - indicates how strongly the member wants to be P
> - range: 0-100
> - 1 - default
> - 0 - will never be a P (Passive Secondaries > Hidden Secondaries
>  Delayed Secondaries) (Part 3, Slide 18)
> - highest priority is always elected as P, if highest priority is stale, it
> - catches up with the rest of the set and becomes P
>
> Hidden
> - maintains copy of primary's data but is invisible to client
> - no traffic (other than for replication)
> - used for deidated tasks: reporting, backing up
> - never can be a P
> - can vote in elections
>
> Slave Delay
> - S that delays applying operations a specified number of seconds behind P
> - used to restore DB corruption caused by human error
> - delay must be smaller than capacity of OPLOG --> WHY?!?!??! (3, 17)
>

## Replica Set Elections
* Election occurs when no primary:
    * initiating a new replica set
    * adding a new node to replica set
    * S loses contact with P
    * P steps down
        * forcefully with replSetStepDown
        * if a S is eligible for election and has higher priority
        * if it cannot contact majority (based on configuration, no care as
         to whether of not node is up or down) of member in replica set
* Process: 
    * (When there is no P, any eligible member can seek election)
    * member seeking an election will send out notice to all reachable
     (to it) members
    * S must be unanimously elected (if a single node vetoes -> election is
     canceled)
        * S will be vetoed if:
            * S is not up to date
            * S has lower priority than another member eligible for election
  
* Only one P per replica set
* If P is unavailable, election to find new P is held
* Fail over time - time P is down and thus replica set cannot accept writes
* heartbeat allows P to now if it can reach majority of set
* heartbeat is sent every 2 seconds, if no response in 10, the node is
 marked inaccessible
* majority is enforced to elect P to avoid more than one primary in case of
 network partition (split brain problem?) (3, 25)
 
## Rollback
* P1 does a successful write and P1 goes down before S's have a chance to
 replicate it --> next elected P (P2) may not have the write
* P2 resurrects and cannot find a sync source --> must roll back by undoing
 ops that were not replicated before fail over
* Undone ops are written to special rollback files that must be applied to
 current P --> write disappears until admin manually applies rollback file
  to current P
* Prevent by writing to majority --> newly elected P must then have a copy
 of that write to be elected
 
## Write Concern
* how many nodes these data need to have been safely committed to before it
 is considered to be complete
* allows us to express how concerned we are with the durability of a
 particular write in a replica set
* can be set for individual operations or collections
* Write:
    * 1. MongoDB applies write to P
    * 2. MongoDB records the operations to P's OPLOG
    * 3. S members replicate OPLOG and apply ops to their data sets
> w:majority
> - will not rollback
>

## ReadPreference (where are we reading?)
Allows application to select which member of replica set it wishes to read from

## ReadConcern (what are we reading?)
* Read:
    * all members in replica set can accept a read, but P is default to
     guarantee latest version of document --> bad for throughput
> r:local
> - returns INSTANCES'S most recent data
> r: majority
> - read data cannot be rolled back
> - returns data replicated to majority of nodes this particular node knows
> about
> r: linearizable
> - read data cannot be rolled back
> - ensures data is most recent by blocking read until last write is
> replicated (super slow)
