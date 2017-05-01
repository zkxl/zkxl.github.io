---
layout: post
title: ICS223 Distributed Database Management
date: 2017-04-21 1:00:00 -0700
categories: Study-Notes
tag: distributed
---
* content
{:toc}





#### DBMS -> Transactions -> Concurrency -> Serializability

__Why DBMS?__
DBMS supports concurrent uses without jeopardizing data __consistency__ and tolerance of system failure so that a user sees a transaction as a consistent and __failure-resistent__ execution of applications.

__DB operations to transactions__
DB operations include READ/WRITE. Each operation is assumed atomic. A __transaction__ is a sequence of DB operations plus __transaction specific operations__, such as BEGIN/END and COMMIT/ABORT. From the perspective of concurrency control mechanism, every record is a DB is an object associated with domain values.  A __state__ of the DBMS is a mapping relation of each object with its domain values.

__Why serialized execution of transactions is desired?__
Concurrent executions of transactions lead to issues, for example:  
* AccountX has $1000 and AccountY has $2000.
* App1 transfer $100 from AccountX t AccountY.
* Concurrently, there is a chance that app2 sums both accounts to $3100.

__Why want concurrency?__
1. Minimize average response time(some large transactions take long time and other transactions will be waiting in an entirely serialized system).
2. Minimize CPU idle time so that system throughput can be maximized.

__Middle ground: Forgo serialized execution of transactions to increase concurrency while following serialized schedules to ensure serializability.__
1. A schedule is an interleaving of operations of multiple transactions.
2. Such a schedule is __serializable__ if it is __equivalent__ to some sequential execution of each transaction.

__How to define equivalence?__
1. Final state equivalence -> Final state serializability (FSR)
2. View equivalence -> View serializability: respective transactions in two different schedules READ the same data values
3. Conflict equivalence -> Conflict serializability:


#### Final State Serializability

__Graph characterization of final state equivalence to test FSR__

For some transaction T on objects x and y:  

Augment it with:
1. $$ T_{0} $$ consisting of WRITE only, one WRITE for each object (initial state).
2. $$ T_{e} $$ consisting of READ only, one READ for each object (final state).

Make each R/W operation a vertex.  

Draw an edge from READ to WRITE if they are in the same transaction, no matter which data item they are on.  

_OR_  

Draw an edge from WRITE to READ if they are in different transactions and target the same data item.

Delete vertices and edges that don't lead to the final state.  

__Claim:__ Two schedules are final-state equivalent if they produce the same graph after the above processes, which take O(n) to run, where n is the number of the operations. One schedule is final-state serializable if it produces the same graph as the serialized execution of transactions does.  

#### View Serializability(VSR)

__Claim:__ Two schedules are view equivalent if they produce the same graph before "deleting" anything during the same process as in "FSR". Testing view equivalence is NP-hard.  

__Compare:__ VSR ensures that each transaction's view of the DB is the same as they are in a serialized execution. FSR only ensures that transactions that have some impact on the final state of the DB have the same view of the DB as they are in a serialized execution.  

#### Conflict Serializability(CSR)

__Operations are conflicting if:__
1. they are on the same object, such as R(x) and W(x)
2. they belong to different transactions
3. at least one of them is WRITE.

__Construct Serializability Graph - SG(V, E)__
1. Make a vertex for every transaction.
2. Draw an edge from $$ T_{x} $$ to $$ T_{y} $$ if:  
* $$ T_{x} $$ contains an operation $$ O_{x} $$
* $$ T_{y} $$ contains an operation $$ O_{y} $$
* $$ O_{x} $$ occurs earlier than $$ O_{y} $$ in schedule S.
* $$ O_{x} $$ and $$ O_{y} $$ are conflicting.

__Claim:__ A schedule is CSR iff the SG(V, E) constructed as above is acyclic. Testing if a graph contains a cycle takes O(n^2) where n is the number of vertices.

#### Recoverability, Cascadeless, Strictness

For a schedule S:

__Recoverability__ is ensured if $$ T_{x} $$ reads from $$ T_{y} $$, then $$ T_{y} $$ commits before $$ T_{x} $$ commits.

__Cascadeless or ACA(avoid cascading aborts)__ is ensured if $$ T_{x} $$ reads from $$ T_{y} $$, then $$ T_{y} $$ commits before $$ T_{x} $$ reads.

__Strictness__ is ensured if $$ T_{x} $$ reads from $$ T_{y} $$ or if $$ T_{x} $$ overwrites $$ T_{y} $$, it does so after $$ T_{y} $$ commits.

* Commercial DBMS ensures strictness.

#### Replications in Distributed System

__Why replica?__ Improve availability and performance.

To ensure distributed system consistency, our goal is __one copy serializability__, which means the distributed DBMS behaves like a serial processor of transactions on a one-copy database. Essentially, it is __synchronized replication__, which comes with huge cost.

Now we talk about __asynchronous replication__ strategies:
1. Primary copy
2. Multi-master
3. Quorum consensus

__Primary copy: (1SR, not partition tolerable)__
1. Clients READ from secondaries and WRITE through primary node only.
2. Updates flow from primary copy to secondaries asynchronously.
3. WRITE is not permitted during partition.

__Multi-master: (non-1SR, partition tolerable)__
1. Clients READ/WRITE to different masters directly.
2. Updates between multiple masters are designed to be __commutative or conflicting updates are merged__.

__Quorum consensus: (1SR, partition tolerable)__ Each READ/WRITE operation has to obtain a certain amount of votes (more than half of the total votes) before execution.

__Commutative downstream updates:__
Thomas's Write Rule:
* Assign timestamp to each of clients' WRITE operation.
* Apply downstream updates to every replica with the latest WRITE.

#### Isolation Levels

Refer to [wiki isolation](https://en.wikipedia.org/wiki/Isolation_(database_systems)) for detailed isolation levels and read phenomena.  

1. __Serializable:__ Hold R/W lock till the end of the transaction. Hold range lock in READ operation to avoid _phantom read_.

2. __Repeatable Reads:__ Hold R/W lock till the end of the transaction.

3. __Read Committed:__ Hold W lock till the end of the transaction. R lock is released as soon as READ operation is done. _Non-repeatable read_ may occur.

4. __Read Uncommited:__ Only W lock is managed. _Dirty read_ may occur.

#### Snapshot Isolation

Before executing transaction T:
1. take a snapshot of committed data
2. execute READ/WRITE operation on the snapshot
3. updates are not visible to other concurrent transactions
4. commit WRITE only if there is no other transaction has preceding WRITE on the same data
5. otherwise, abort and rollback the current transaction

Problem with SI:
1. It doesn't ensure Serializability.
2. [write skew anomaly](https://en.wikipedia.org/wiki/Snapshot_isolation).  

__Inconsistency is often not a showstopper, but a mere inconvenience. All we need to do is making sure that application logic doesn't lead to inconsistency or inconsistency can be worked around on application layers.__


#### Two phase RW locking

__2PL RW locking__ combines two-phase locking and share-exclusive locking.  

__2PL__ has two phases:  
1. expanding phase, new locks on the desired data item can be acquired but no locks can be released till the transaction terminates.  
2. shrinking phase, acquired locks can be released after the transaction terminates and no new locks can be acquired.  


__RW locking__ has
1. read (shared) locks that multiple clients can hold at the same time.  
2. write (exclusive) locks that only one client can hold while no one holds read lock on the same data item.  

2PL RW locking ensures __serializability__ but it is also subject to __irrecoverability, deadlock and starvation__.

__Irrecoverability__
picture reference http://www.edugrabs.com/2-phase-locking/  
<img src="{{ '/styles/images/distributed-database-mgmt/irrecoverability.png' }}" width="50%" />


__Deadlock__
picture reference http://www.edugrabs.com/2-phase-locking/  
<img src="{{ '/styles/images/distributed-database-mgmt/deadlock.png' }}" width="50%" />


__Starvation__
picture reference http://www.edugrabs.com/2-phase-locking/  
<img src="{{ '/styles/images/distributed-database-mgmt/starvation.png' }}" width="50%" />


To __prevent deadlock__, we need to detect deadlock first by using __timeout__ or constructing __wait-for graph__. But it is too slow and 99% of cases don't yield any deadlock. So:  
1. Instead, we use __conservative 2PL RW locking__, which acquires all the resources needed in a transaction before it starts to executing anything. But it leads to problems such as less resources utilization and less concurrency.
2. On the other side, __strict 2PL RW locking__ will execute whatever the current resources allow in a transaction, under the condition that all the write locks will be not be released until the transaction commits. Of course, strict 2PL ensures serializability but doesn't prevent deadlock.
3. More strictly, __rigorous 2PL RW locking (strongly strict 2PL)__ will hold all the R/W locks till the transaction commits.

__Relations between CSR, VSR and variations of 2PL lockings__ picture reference http://www.edugrabs.com/2-phase-locking/
<img src="{{ '/styles/images/distributed-database-mgmt/coverability-csr-vsr-2pl.png' }}" width="50%" />


#### Scheduling Protocol

__Scheduler__ takes write-ahead log as input and outputs a serializable (CSR) schedule to be applied to DB.  

---

__2PL Scheduling Protocol__ schedules operations under 2PL rules so basically:  
* For non-conflicting operations, their order doesn't matter.
* For conflicting operations, not only should they obey 2PL rules, the order that they came in should also be preserved.  

__Proof of correctness:__  
_Assume 2PL scheduling produced such a schedule that, if we construct a serialize graph (SG) for it, it will have a cycle._
_Let's say, T1, T2, ..., Tn formed such a cycle, which also means: T1 and T2 must contain conflicting operations, T2 and T3 must contain conflicting operations...and so on till Tn and T1 must contain conflicting operations._
_According to 2PL rules, T2 would not be able to acquire all the lock until T1 released all the lock. Same thing goes. T1 will not acquire all the locks until Tn released all the lock._ -> __Contradiction__  

__Distributed 2PL Scheduling__ says: transaction T doesn't release locks at any site it executes until it has acquired all the locks it needs at every site it executes on. It is implied that scheduler at each site synchronizes each other.

---

__Timestamp Ordering Protocol:__
1. Every time a transaction T comes in, it request a TimeStamp(T), which is unique and monotonically increasing.
2. Every data item x is associated with a last_read_timestamp and a last_write_timestamp. They are updated every time a READ/WRITE operation is executed.  
3. The protocal says: READ(x) operation in transaction T will be executed iff TimeStamp(T) >= x.last_write_timestamp. WRITE(x) operation in transaction T will be executed iff TimeStamp(T) >= x.last_read_timestamp.
* The "write" part is also known as [Thomas's Rule](https://en.wikipedia.org/wiki/Thomas_write_rule)

__Proof of Correctness:__
_Assume 2PL scheduling produced such a schedule that, if we construct a serialize graph (SG) for it, it will have a cycle._
_Let's say, T1, T2, ..., Tn formed such a cycle. According to Timestamp Ordering, TS(1) < TS(2) < ... < TS(n) < TS(1)._ -> __Contradiction__

---

__Serializable Graph Test (SGT) Protocol:__
1. For every incoming operation O from some transaction T, draw an edge from all the transactions whose operations are in conflict with O, to T. (Create a vertex for T if it doesn't already exist)
2. If it creates a cycle, reject the operation O and abort the transaction T. Else, add the edge.
3. For those transactions that are done and don't have incoming edges, we can remove them.

---

__Variations:__ (derived from application needs to push for better performance)
1. Conservative 2PL:
* try to acquire all the locks for the entire transaction
* if succeed, execute it and release.
* else, release all the locks it acquired immediately and wait. (Different from 2PL, conservative 2PL never aborts a transaction)
2. Aggressive 2PL:
* a transaction T will be executed iff all the active transactions don't have any operations conflict with T

__Optimistic Concurrency Control__ # TODO



<!--
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
buffer
-->
