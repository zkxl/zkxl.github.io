---
layout: post
title: ICS223 Distributed Database Management
date: 2017-04-21 1:00:00 -0700
categories: Study-Notes
tag: distributed
---
* content
{:toc}




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
<img src="{{ '/styles/images/distributed-database-mgmt/irrecoverability.png' }}" width="50%" />

* reference http://www.edugrabs.com/2-phase-locking/

__Deadlock__  
<img src="{{ '/styles/images/distributed-database-mgmt/deadlock.png' }}" width="50%" />

* reference http://www.edugrabs.com/2-phase-locking/

__Starvation__  
<img src="{{ '/styles/images/distributed-database-mgmt/starvation.png' }}" width="50%" />

* reference http://www.edugrabs.com/2-phase-locking/

To prevent deadlock, we don't use prevention algorithm because it is too slow and 99% of cases don't yield any deadlock.
1. Instead, we use __conservative 2PL RW locking__, which acquires all the resources needed in a transaction before it starts to executing anything. But it leads to problems such as less resources utilization and less concurrency.
2. On the other side, __strict 2PL RW locking__ will execute whatever the current resources allow in a transaction, under the condition that all the write locks will be not be released until the transaction commits. Of course, strict 2PL ensures serializability but doesn't prevent deadlock.
3. More strictly, __rigorous 2PL RW locking (strongly strict 2PL)__ will hold all the R/W locks till the transaction commits.

__Relations between CSR, VSR and variations of 2PL lockings__
<img src="{{ '/styles/images/distributed-database-mgmt/coverability-csr-vsr-2pl.png' }}" width="50%" />


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
2. $$ T_{-1} $$ consisting of READ only, one READ for each object (final state).

Make each R/W operation a vertex.  

Draw an edge from READ to WRITE if they are in the same transaction.  

_OR_  

Draw an edge from WRITE to READ if they are in different transactions.

Delete vertices and edges that don't lead to the final state.  

__Claim:__ Two schedules are final-state equivalent if they produce the same graph after the above processes, which take O(n) to run, where n is the number of the operations. One schedule is final-state serializable if it produces the same graph as the serialized execution of transactions does.  


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
