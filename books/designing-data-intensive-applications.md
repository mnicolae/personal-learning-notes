# Chapter 1. Reliable, Scalable, and Maintainable Applications

* Data-intensive vs. CPU-intensive
* Common building blocks: databases, caches, search indexes, stream processing, batch processing

# Chapter 3: Storage and Retrieval

# Chapter 5: Replication

Replication - keep a copy of the same data on multiple machines that are connected via a network.

Reasons:
* keep data geo close to users (and thus reduce latency)
* allow system to continue working even if some of its parts have failed (and thus increase availability)
* scale out number of machines that can serve read queries (and thus increase throughput)

Difficulty in replication: handling changes. Three algorithms for handling change:
* single-leader
* multi-leader
* leaderless

## Leaders and followers

Replica - node that stores a copy of the database.

Leader-based replication
1. One replica is designated the leader. When clients want to write to the database, they must send their requests to the leader, which first writes the new data to its local storage.
2. Other replicas are followers. Whenever the leader writes new data to its local storage, it also sends the data change to all of its followers as part of replication log or change stream. Each follower takes the log from the leader and updates its local copy of the database accordingly, by applying all the writes in the same order as they were processed on the leader.
3. When a client wants to read from the database, it can query either the leader or any of its followers. However, writes are only accepted on the leader (the followers are read-only from client's POV).

## Synchronous vs asynchronous replication

Semi-synchronous replication is best trade-off between durability and latency. One of the followers is synchronous, and the others are asynchronous. Guarantees that you have an up-to-date copy on at least two nodes.

## Setting up new followers

1. Take a snapshot of the leader's database.
2. Restore the snapshot on the new follower node.
3. Follower connects to the leader and requests all the data changes that have happened since the snapshot was taken. This requires that the snapshot is associated with an exact position in the leader's replication log.
4. When the follower has processed the backlog of data changes since the snapshot, it can continue to process data changes from the leader as they happen.

## Handling node outages

### Follower failure: catch-up recovery

Follower connects to the leader and requests all the data changes that occurred during the time when the follower was disconnected. When it has applied these changes, it has caught up to the leader and continue receiving a stream of data changes as before.

### Leader failure: failover

Failover - process by which one of the followers is promoted to be the new leader, client is reconfigured to send writes to the new leader, and the other followers start consuming data changes from the new leader.

Failover can happen manually or automatically. Automatic failover steps:

1. Determining that the leader has failed. Most systems simply use a timeout.
2. Choosing a new leader.
3. Reconfiguring the system to use the new leader.

Issues:
* Conflicting writes on asynchronous replication
* Discarding writes if other storage systems outside of the database need to be coordinated with the database contents
* Split brain
* Right timeout value

## Implementation of replication logs

### Statement-based replication

Leader logs every write request (statement) that it executes and sends that statement log to its followers. Each follower parses and executes that SQL statement as if it had been received from a client.

Issues:
* Non-deterministic functions
* Statements that use an auto-incrementing column when there are multiple concurrently executing transactions
* Statements that have a side effect

### Write-ahead (WAL) logging

Besides writing the log to disk, the leader also sends it across the network to its followers. When the follower processes this log, it builds a copy of the exact same data structure as found on the leader.

Issues:
* Replication is closely coupled to the storage engine.
* If replication protocol does not allow the follower to use a newer software version than the leader, such upgrade requires downtime.

### Logical (row-based replication)

Different log formats for replication and for the storage engine. Logical log - sequence of records describing writes to database tables at the granularity of a row.

Allows leader and follower to run different versions of the database software, or even different storage engines.

### Trigger-based replication

Replication is at the application layer.

Alternatively, use triggers and stored procedures. Trigger - custom application code that is automatically executed when a data change (write transaction) occurs in a database system.

Issues:
* greater overheads than other replication methods
* more prone to bugs and limitations than the database's built-in replication

## Problems with Replication Lag

Eventual consistency.

### Reading Your Own Writes

Read-after-write consistency.

Solutions:
* When reading something that the user may have modified, read it from the leader; otherwise, read it from a follower.
* Track the time of the last update and, for one minute after the last update, make all reads from the leader. Monitor the replication lag on followers and prevent queries on any follower that is more than one minute behind the leader.
* Client remembers timestamp of its most recent write - then the system ensures that the replica serving any reads for that user reflects updates at least until that timestamp. If replica is not sufficiently up-to-date, read from another replica or wait until the replica has caught up.
* Multi-region more challenging.

Cross-device read-after-write consistency.

Issues:
* Need to centralize the timestamp of the user's last update.
* No guarantee that requests from different devices are routed to the same data center.

### Monotonic reads

Eventual consistency < monotonic reads < strong consistency.

Solution: make sure that each user always makes their reads from the same replica.

### Consistent Prefix Reads

Consistent prefix reads - if a sequence of writes happens in a certain order, then anyone reading those writes will see them appear in the same order.

Solution: writes that are causally related to each other are written to the same partition.

## Multi-Leader Replication

### Use Cases for Multi-Leader Replication

#### Multi-datacenter operation

A leader in each datacenter.

Performance
* single-leader: adds significant latency to writes
* multi-leader: the inter-datacenter network delay is hidden from users

Tolerance of datacenter outages
* single-leader: failover required
* multi-leader: failover not required

Tolerance of network problems
* single leader: very sensitive to problems in the inter-datacenter link
* multi-leader: tolerates network problems better

Issues:
* conflict resolution

#### Clients with offline operation

Appropriate if you have an application that needs to continue to work while it is disconnected from the Internet (think Splitwise).

Each device has a local database that acts as a leader.

#### Collaborative editing

Make unit of change very small (i.e., keystrokes) and avoid locking. Requires conflict resolution.

### Handling Write Conflicts

#### Synchronous versus asynchronous conflict detection

Synchronous conflict resolution defeats the purpose of multi-leader replication.

#### Conflict avoidance

In application where a user can edit their own data you can ensure that requests from a particular user are always routed to the same datacenter and use the leader in that datacenter for reading and writing.

Issues:
* datacenter failure
* user reassigned to different datacenter

#### Converging toward a consistent state

