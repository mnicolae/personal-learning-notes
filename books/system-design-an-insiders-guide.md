# System Design Interview - An Insider's Guide

## Chapter 1: Scale from Zero to Millions of Users

### Database

* Deep dive on different types of NoSQL databases: key-value stores, graph stores, column stores, document stores.

### Database replication

* Deep dive on replication methods: multi-masters, circular replication.

### Cache

* Deep dive on cache policies: read-through, write-through, write-behind, refresh-ahead.
* Deep dive on eviction policies: LRU, LFU, FIFO.
* Deep dive on single point of failure as it pertains to caching.

### CDN

* Deep dive on object versioning as it pertains to CDNs.

### Stateless web tier

* Deep dive on sticky sessions.
* Deep dive on stateful vs. stateless applications.

### Data centers

* Deep dive on Route 53 routing policies.

### Message queue

### Database scaling

* Get more familiar with consistent hashing, hotspot key problem, database denormalization.

## Chapter 2: Back-of-the-envelope Estimation

### Power of two

Add table in markdown format.

### Latency numbers every programmers should know

### Availability numbers

### Tips

* Rounding and approximation
* Write down your assumptions
* Label your units
* Commonly asked back-of-the-envelope estimations: QPS, peak QPS, storage, cache, number of servers, etc.

## Chapter 3: A Framework for System Design Interviews

### A 4-step process for effective system design interview

Goal of system design interview is to give signals in
* technical design skills
* ability to collaborate
* ability to work under pressure
* ability to work under pressure to resolve ambiguity constructively
* ability to ask good questions

and to catch red flags in
* over-engineering

### Step 1 - Understand the problem and establish design scope

One of the most important skills as an engineer is to
* ask the right questions
* make the proper assumptions
* gather all the information needed to build a system

Write down your assumptions. Ask questions to understand the exact requirements.

### Step 2 - Propose high-level design and get buy-in

* Come up with an initial blueprint for the design.
* Draw box diagrams with key components on the whiteboard or paper.
* Do back-of-the-envelope calculations to evaluate if your blueprint fits the scale constraints.

If possible, go through a few concrete use cases.

### Step 3 - Design deep dive

At this step, you and your interviewer should have already achieved the following objectives:
* Agreed on the overall goals and feature scope
* Sketched out a high-level blueprint for the overall design
* Obtained feedback from your interviewer on the high-level design
* Had some initial ideas about areas to focus on in deep dive based on feedback

Work with the interviewer to identify and prioritize components in the architecture.

### Step 4 - Wrap up

The interviewer might ask you a few follow-up questions or give you the freedom to discuss other additional points
. Directions to follow:
* Identify system bottlenecks and discuss potential improvements
* Give the interviewer a recap of your design
* Error cases
* Operational logistics (monitoring, logging, CICD)
* Scale curve
* Other refinements

#### Time allocation

Step 1 - Understand the problem and establish design scope - 3 - 10 minutes
Step 2 - Propose high-level design and get buy-in - 10 - 15 minutes
Step 3 - Design deep dive - 10 - 25 minutes
Step 4 - Wrap up - 3 - 5 minutes

# Sample questions

| System | Examples |
|---|---|
| URL shortening service | TinyURL, bitly
| Video streaming service | YouTube, Netflix, Twitch
| Global chat service | Facebook Messenger, WhatsApp
| Social network + message board service | Quora, Reddit, HackerNews
| Social graph | Facebook, LinkedIn
| Global file storage & sharing service | Dropbox, Google Drive, Google Photos
| Social media service w/ 100M+ users | Facebook, Twitter, Instagram
| Ride sharing service | Uber, Lyft
| Web Crawler or Type-Ahead (search engine) | Google, Bing
| API Rate Limiter | Firebase, GitHub
| Proximity server | Yelp, Nearby Places/Friends

# API Rate Limiter

Benefits of using a rate limiter:
* Prevent resource starvation caused by DDoS attacks
* Reduce cost
* Prevent servers from being overloaded

## Step 1 - Understand the problem and establish design scope

Rate limiter implementation location:
* client
* server
* middleware (API gateway)

Rate limiting algorithms:
* Token bucket
* Leaking bucket
* Fixed window counter
* Sliding window log
* Sliding window counter

Token bucket is used by Amazon and Stripe.

Token bucket pros
* Easy to implement
* Memory efficient
* Token bucket allows a burst of traffic for short periods

Token bucket cons
* It might be challenging to tune parameters properly

Leaking bucket pros
* Memory efficient
* Suitable for use cases that a stable outflow rate is needed

Leaking bucket cons
* Recent requests may be rate limited
* It might be challenging to tune parameters properly

Fixed window counter pros
* Memory efficient
* Easy to understand
* Resetting available quota at the end of a unit time window fits certain use cases

Cons
* Spike in traffic at edges of a window could cause more requests that the allowed quota to go through

Sliding window log pros
* Very accurate

Cons
* Consumes lots of memory

Sliding window counter pros
* Smooths out spikes in traffic
* Memory efficient

Sliding window counter cons
* Only works for not-so-strict look back window

### High level architecture

![](https://s3.amazonaws.com/thinkific/file_uploads/334940/images/a97/360/83a/1595375483385.jpg)

## Step 2 - Design deep dive

### Rate limiting rules

Rules are generally written in configuration files and saved on disk.

### Exceeding the rate limit

In case a request is rate limited, APIs return a HTTP response status code 429 to the client. Depending on the use case, we may enqueue the rate-limited requests to be processed later.

The rate limiter returns the following HTTP headers to clients:

* `X-Ratelimit-Remaining`
* `X-Ratelimit-Limit`
* `X-Ratelimit-Retry-After`

### Detailed design

![](https://s3.amazonaws.com/thinkific/file_uploads/334940/images/759/cc4/589/1595375995072.jpg)

### Rate limiter in distributed environment

#### Race condition

Options:
* locks
* Lua scripts
* sorted sets data structure in Redis

#### Synchronization issue

Options:
* sticky sessions
* centralized data stores like Redis

### Performance optimization

* Edge servers
* Synchronize data with an eventual consistency model

### Monitoring

Want to make sure
* the rate limiting algorithm is effective
* rate limiting rules are effective

## Step 4 - Wrap up

Additional talking points
* Hard vs. soft rate limiting
* Rate limiting at different levels
* Avoid being rate limited

## Consistent hashing

Consistent hashing is a special kind of hashing such that when a hash table is re-sized and consistent hashing is used, only `k/n` keys need to be remapped on average, where k is the number of keys, and n is the number of slots.

Number of buckets on the ring: 2^160.

Basic steps:
* map servers and keys on to the ring using a uniformly distributed hash function
* to find out which server a key is mapped to, go clockwise from the key position until the first server on the ring is found

Problems:
* impossible to keep the same size of partitions on the ring for all servers
* possible to have a non-uniform key distribution on the ring

Solution? Virtual nodes (replicas). W/ 100 or 200 virtual nodes, the std dev is 5-10%. However, more space is needed to store data about virtual nodes. This is a trade-off to consider.

When a server is added or removed, a fraction of the keys need to redistributed.

![](https://s3.amazonaws.com/thinkific/file_uploads/334940/images/ad4/0c4/153/1595380135389.jpg)

Use cases:
* AWS DynamoDB partitioning
* Data partitioning across the cluster in Apache Cassandra
* Akamai CDN
* Discord
* Maglev NLB


## Key-value store

### Step 1 - Understand the problem and establish design scope

* The size of a key-value pair is small: less than 10KB.
* Ability to store big data.
* High availability.
* High scalability.
* Automatic scaling.
* Tunable consistency.
* Low latency.

### Single server key-value store

Optimizations:
* data compression
* store only frequently used in memory and the rest on disk

Distributed key-value store is required to store big data.

### Distributed key-value store

CAP theorem: impossible for a distributed system to simultaneously provide more than 2/3 guarantees:
* consistency
* availability
* partition tolerance

Consistency: all clients see the same data at the same time no matter which node they connect to.

Availability: any client which requests data gets a response even if some of the nodes are down.

Partition tolerance: cluster continues to work despite communication break between two nodes.

Choosing the right CAP guarantees that fit your use case is an important step in building a distributed key-value store.

### System components

#### Data partition

Solved using consistent hashing.

#### Data replication

To achieve high availability and reliability, data is replicated asynchronously over N servers, where N is a configurable parameter.

#### Consistency

N = The number of replicas

W = A write quorum of size W. For a write operation to be considered as successful, write operation must be acknowledged from W replicas.

R = A read quorum of size R. For a read operation to be considered as successful, read operation must wait for responses from at least R replicas.

| Use case | Config |
|---|---|
| Fast read | R = 1 and W = N |
| Fast write | W = 1 and R = N |
| Strong consistency | W + R > N |
| Not strong consistency | W + R <= N |

Consistency models
* strong consistency
* weak consistency
* eventual consistency

#### Versioning

Need a versioning system that
* detects conflicts
* reconciles conflicts

Vector clock is a `[server, version]` pair associated with a data item. It can be used to check if one version precedes, succeeds, or is in conflict with others.

Vector clock is represented by `D([S_1, v1], [S_2, v2], ..., [S_n, vn])`, where `D` is a data item, `v1` is a version counter, and `s1` is a server number, etc. If data item D is written to server `S_i`, the system must perform one of the following tasks.
* Increment `v_i` if `[S_i, v_i]` exists.
* Otherwise, create a new entry `[S_i, 1]`.

#### Handling failures

* Gossip protocol.
* Sloppy quorum.
* Hinted hand-off.
* Anti-entropy protocol.

#### System architecture diagram

#### Write path

* Sorted-string tables (SSTables)

#### Read path

* Bloom filter.

#### Summary

| Goal/Problems | Technique |
|---|---|
| Ability to store big data | Consistent hashing
| High availability reads | Data replication. Multi-data center setup.
| High availability writes | Versioning and conflict resolution with vector clocks.
| Dataset partition | Consistent hashing
| Incremental scalability | Consistent hashing
| Heterogeneity | Consistent hashing
| Tunable consistency | Quorum consensus
| Handling temporary failures | Sloppy quorum and hinted hand-off
| Handling permanent failures | Merkle tree
| Handling data center outage | Cross-data center replication

## Unique Id Generator in Distributed Systems

### Step 1 - Understand the problem and establish design scope

* IDs must be unique and sortable
* IDs are numerical values only
* IDs fit into 64-bit
* IDs are ordered by date
* Ability to generate 10,000 unique IDs per second

### Step 2 - Propose high-level design and get buy-in

#### Multi-master replication

Cons:
* hard to scale with multiple data centers
* ids do not go up with time across multiple servers
* does not scale well when a server is added or removed

#### UUID

Pros:
* No synchronization issues
* Easy to scale

Cons:
* IDs are 128 bits long, but our requirement is 64 bits
* IDs do not go up with time
* IDs could be non-numeric

#### Ticket server

Pros:
* numeric IDs
* easy to implement, works well for small to medium-scale applications

Cons:
* single point of failure

#### Twitter Snowflake

* Sign bit. Reserved for future uses.
* Timestamp. Milliseconds since custom epoch.
* Data center ID
* Machine ID
* Sequence number. For every ID generated on that machine / process, the sequence number is increased by 1. The number is reset to 0 every millisecond.

### Step 4 - Wrap up

Additional talking points:
* Clock synchronization
* Section length tuning
* High availability

## Design a URL shortener

### Step 1 - Understand the problem and establish design scope

Use cases:
* URL shortening: given a long URL => return a much shorter URL
* URL redirecting: given a shorter URL => redirect to the original URL
* High availability, scalability, and fault tolerance

#### Back-of-the-envelope calculation

* Writes / second: 1160
* Reads / second: 11600; assuming 10:1 ratio of reads to writes
* 365 billion records, assuming service runs for 10 years
* average URL length 100
* 36.5 TB storage requirement

### Step 1 - Understand the problem and establish design scope

* Popular social networks contain millions and even billions of connections between individuals.
* Design a system that allows a user to search for another person, and see the shortest path between them.

Questions to clarify requirements and constraints.

* How many users are there? 10M+.
* How many connections can a user have? 5000.
* How many connections does a user have on average? 500.

### Step 2 - Propose high-level design and get buy-in

![](https://s3.amazonaws.com/thinkific/file_uploads/334940/images/8b8/fee/a5f/1595390839913.jpg)

![](https://s3.amazonaws.com/thinkific/file_uploads/334940/images/0ec/4a4/106/1595390840050.jpg)

Hash function requirements:
* each `longURL` must be hashed to one `hashValue`
* each `hashValue` can be mapped back to the `longURL`

### Step 3 - Design deep dive

#### Data model

Store `<shortURL, longURL>` mapping in a relational database.

#### Hash function

Used to hash long URL to short URL.

#### Hash value length

![](https://s3.amazonaws.com/thinkific/file_uploads/334940/images/655/a7e/b2e/1595390931306.jpg)

Hash + collision resolution.

Base 62 conversion.

![](https://s3.amazonaws.com/thinkific/file_uploads/334940/images/6b3/02f/34d/1595390931870.jpg)

URL shortening deep dive

![](https://s3.amazonaws.com/thinkific/file_uploads/334940/images/cf9/43d/aa8/1595390932015.jpg)

URL redirecting deep dive

![](https://s3.amazonaws.com/thinkific/file_uploads/334940/images/ebe/a7f/c79/1595390932287.jpg)

### Step 4 - Wrap up

Additional talking points
* Rate limiter
* Web server scaling
* Database scaling
* Analytics
* Availability, consistency, reliability

## Social graph

### Step 1: Outline use cases and constraints

#### Use cases

* User searches for someone and sees the shortest path to the searched person
* Service has high availability

#### Constraints and assumptions

State assumptions

* Traffic is not evenly distributed
* Graph data won't fit on single machine
* Graph edges unweighted
* 100 mil users
* 50 friends per user average
* 1 bil friend searches per month

#### Back-of-the-envelope calculations

* 100 mil * 50 = 5 bil friend relationships
* 400 friend searches per second

### Step 2: Create a high level design

![](https://camo.githubusercontent.com/77cd62d1b709a77ceaf5543ab06bde1ba684f842e64755c198296ef87a0cd2c5/687474703a2f2f692e696d6775722e636f6d2f7778587971324a2e706e67)

### Step 3: Design core components

### Step 4: Scale the design

* To address the constraint of 400 average read requests / sec, person data can be served from the memory cache.

![](https://camo.githubusercontent.com/8966044b036b79f7ce1706d40ad214c74193ae8ded36743e37e03e79b2bf46d5/687474703a2f2f692e696d6775722e636f6d2f636443763567372e706e67)

#### Case 1: simplify the problem (not considering millions of people)

We can construct a graph by treating each individual as a node and letting an edge between nodes indicate the two
 individuals are connected. If we want to find the shortest path between two individuals, we can start with one
  individual and do a BFS. Alternatively, we can do a bi-directional BFS. This means doing two BFSs, one starting at
   the source, and one starting at the destination. When the searches collide, we've found the shortest path between
    them.

How fast is the search algorithm? Assume every individual has `k` connections, and source and destination have one
 individual in common.

 Time complexity for traditional BFS is `k + k * k`. Time complexity for bi-directional BFS is `2 * k`.

 Generalizing this for a path of length `q`:
 * BFS: `O(k ^ q)`.
 * Bi-directional BFS: `O(k ^ q / 2 + k ^ q / 2)`.

 If we imagine a path of length 4 where each person has 100 friends, BFS requires searching `100 ^ 4 = 100 million
 ` nodes, bi-directional BFS requires searching `2 * 100 ^ 2 = 20,000` nodes.

#### Case 2: Handle millions of users

We cannot possibly keep all data on one database server. Person data structure needs to be amended; the list of friends
 can
 be replaced with a list of friend IDs, and it can be traversed as follows:

 1. For each friend ID: `int shardIndex = getShardIDForUser(personId);`
 2. Go to `shardIndex`
 3. On that shard, do: `Person friend = getPersonWithID(personId)`

Optimizations and follow-up questions.

Optimizations: reduce shard jumps.

Switching from one shard to another is expensive. Instead of switching from one to shard to another, try to batch the
 switches (i.e., first group list of friends by shards, and process each group one-at-a-time).

 Optimization: smart division of shards and people.

 Rather than randomly assigning people to shards, assign them based on clusters in the graph (or other affinity
  metric, such as country, state, etc.).


## Video streaming service

### YouTube

Resources:
* [System Design Interview: Mini YouTube](https://eileen-code4fun.medium.com/system-design-interview-mini-youtube-5cae5eedceae)
* [System Design Interview - An Insider's Guide - Chapter 14: Design YouTube](https://systeminterview.com/design-youtube.php)

### Netflix

Resources:
* [Mastering Chaos - A Netflix Guide to Microservices](https://www.youtube.com/watch?v=CZ3wIuvmHeM)
* [NETFLIX system design](https://medium.com/@narengowda/netflix-system-design-dbec30fede8d)
* [A Design Analysis of Cloud-based Microservices Architecture at Netflix](https://medium.com/swlh/a-design-analysis-of-cloud-based-microservices-architecture-at-netflix-98836b2da45f)
* [Data Compression for Large-Scale Streaming Experimentation](https://netflixtechblog.com/data-compression-for-large-scale-streaming-experimentation-c20bfab8b9ce)