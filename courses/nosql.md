# NoSQL 
*Notes on 157c taught by S Kim*
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