# NoSQL

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

## CAP Theorem
States no distributed system can strongly support more than 2 of the three
> **C**onsistency - system appears as a single node (every read returns the
 most recent write)
>
> **A**vailability - all non-failing nodes can be written/read from
>
> **P**artition Tolerance - system will function in spite of network partition

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