## DBMS Uses
### ER Diagram
entity, relationship, degree + cardinality

### three elements in sharing structured data
meta data + logical & physical data independence + tx

### Storeage Engine & Relational Engine
#### Storeage Engine
def
: a software module used to create, read, and update data between the disk and memory while still maintaining data integrity (rollback journals and write-ahead logs).

component
: indexing, backup and restore, concurrency control, logging, and recovery.
#### Relational Engine
def
: Optimizes the query plan and chooses the best algorithms such as for searching and sorting. (Query Optimizer)
component
: SQL parser, catalogs, query optimization, query execution.

## SQL
### 5 integrity constraints
Not NULL, CHECK, UNIQUE, PK, FK
 - PK: functional dependency

### 5 data type

### 3 Join Type
inner join, semi join, outer join

## Query execution algorithms
### aggregate (group by)/dup removal (distinct) Basic Methods
*in stream*
- sorted input

*in sort*
- sort intermediate result (min/max, count, sum)

*hash*
- unsorted input and in-memory hash table

### relation algebra
label aggregation (group by what and compute what)

## Query Optimization
### query execution plan
requirements similar to relation algebra

### 4 Join Alg & pros and cons
- join predicate
>> equality?
- input requirement
>> in mem (input size)? sorted? idx?
- output 
>> pipeline?

*naive nlp*
- easy to implement~

*idx nlp*
- \\; deal with any size of input; sorted outer input may help but not required; idx on inner input 

*merge join*
- require at least one equality predicate; \\; sorted; \\
- not friendly for dup join key values

*hash join*
- require at least one equality predicate; build input fit in mem; \\; \\

*hash join VS hash aggregation*
1. hash join stores each build row, so the total memory requirement is proportional to the number and size of the build rows; 
hash aggregate stores one row for each group, so the total memory requirement is proportional to the number and size of the output groups or rows; 
2. hash aggregation collapse duplicate rows into one group

## Tree-based Storage Structures
- num of comparison: log<sub>2</sub>N
- height: log<sub>fan-out</sub>N
### b tree / hash Index
#### Hash-Based Index
- Put records into buckets according to hash(key)
- can also add Axulleries key in each bucket (sort in each bucket)
### key value store / RDBMS
RDBMS: 
- pros: more comprehensive; better for data organization and management 
- cons: complex; require more memory

key value store: 
- pros: more concurrency control; light weight; better for scalability and performance; faster for recovery; (data model is not hierachical); 
- cons: does not support sql; not great for tx and physical data independence

## Transaction Management
### ACID
- Atomicity: multiple actions indivisible, all or nothing
- Consistency: for every tx, it sees the same database. integrity constraints, keep in valid state. 
- Isolation: concurrency control, illusion of single user access
- Durability: Can survive craches.
### WAL
record any changes in the persistent database. log first and then modify the database. 
### 3 Failure
tx, sys, media
### Tx F Recovery
scan log + log the rollback + rollback 
### Sys F Recovery
- checkpoint: scan active tx, most recent log rec, dirty pg in buf pool
- log analysis + redo + undo
- incomplete tx + mem failure + good rec log + good db (slightly out of date)

## Concurrency Control (isolation)
### multi-version storage
#### copy-on-write B-Tree
version number of idx entries as key suffix (e.g. LSN)

example:
In a B+ tree with 7 nodes (blocks), the root node is A, and the leaf node C has modification operations. Copy C and its ancestor nodes: C', B', A', then modify the data on these new copies of the nodes, and finally write the modified data to the new location of the disk file. In this process, if a business is reading the B+ tree, the complete old data of C, B and A can still be read. When the data of C', B' and A' nodes is brushed to the disk, the root node of the B+ tree is changed to A', then the business can read the new data of the B+ tree. At this time, the old data A, B and C still exist. You can choose to keep them as backup or recycle the disk space.

### high level VS low level
- tx level: DB contents, locks, entrire tx (hrs), S/X/U
- thread level: in-mem data structure, latches, critical section (secs), R/W
### OCC & PCC
#### Optimistic Concurrency Control
Assume most transactions won't have conflicts
1. Read: the transaction executes, readling values from the database and writing to a private workspace.
2. Validation: if the transaction decides that it wants to commit, the DBMS checks whether the transaction could possibly have conflicted with any other concurrently executing transaction. If there is a possible conflict, abort.
3. Write: if validatipon determines that there are no conflicts, the changes to the data objects will be copied from the private workspace to the database.
#### Timestamps(Timestamp based concurrency control)
Each transaction can be assigned a timestamp at startup, and we can ensure , at execution time, that if action ai of transaction Ti conflicts with action aj of transaction Tj, ai occurs before aj if TS(Ti) < TS(Tj)

If an action violates this ordering, the transaction is aborted and restarted.

#### Snapshot isolation
- get rid of read lock
- read from the version at the beginning of tx
- does not care changes happen after the point
- good for read only tx, but have problem with read-write tx
1. The basic idea is take a snapshot(a copy of the entire database at a certain moment) once a while. 
2. Delete logs that don't have new information on tip of the snapshot.
3. By doing so, we could have a much more compact log which save time for recovery. 

### 3 locks
*KVL*
- key value locking
- distinct key value + gap to prior key value
- good for searching

*IM*
- index management
- (in all idx) logical row + gap to prior logical row
- good for update

*KRL*
- key range locking
- (one row, one idx) index entry + gap to prior idx entry
- good for searching

*comparison*
- KVL+IM lock range larger, management easier (acquisition fewer). 
- KRL lock range smaller, concurrency higher. 

## Columnar Storage
### application scenario
many rows(1‰) & few cols

## Scaling & parallel query execution
### partitioning & pipelining
Splitting the data across the machine; Splitting query operations
### cost metric
response time, total resource consumption
### information transfer
#### 2 phase commit
1st phase:

All participants leave the final commit decision to the coordinator and promise (guarantee) that they will implement the coordinator’s decision, i.e., they will commit or abort (roll back) as directed.

2nd phase:

the coordinator informs all participants of its decision and waits for them to report completion.

#### redundancy: mirroring/log shipping

