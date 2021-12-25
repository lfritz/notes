# Designing Data-Intensive Applications

*Martin Kleppmann: Designing Data-Intensive Applications (2017)*



## Part I: Foundations of Data Systems


### Chapter 1: Reliable, Scalable, and Maintainable Applications

This chapter introduces the approach of the book, starting with some basic
terminology:

 * Data-intensive applications are ones where CPU power is rarely the limiting
 factor. The amount of data, the complexity of data, and the speed at which
 it is changing are bigger issues.

 * Reliability means a system works correctly, even when faults occur.

 * Scalability means having strategies for keeping good performance even
 when load increases.

 * Maintainability has many facets, but in essence it’s about making
 life better for the engineering and operations teams who need to work with
 the system.

A fault is one component of a system deviating from its spec, whereas a failure
is when the system as a whole stops providing the required service. Systems
that anticipate faults and cope with them are called fault-tolerant or
resilient.

To talk about scalability, we need to decribe the load and performance of the
system. Load can be described with a few numbers called load parameters. The
best choice of load parameters depends on the application. Examples are the
number of requests per second, the ratio of reads to writes, or the number
of simultaneously active users.

For batch processing systems, performance usually means throughput -- the
number of records we can process per second. In an online system, we usually
care about the service’s response time -- the time between a client sending
a request and receiving a response.

The majority of the cost of software is not in its initial development,
but in its ongoing maintainance. To minimize pain during maintainance,
we should pay attention to three design principles:

 * Operability: make it easy for operations teams to keep the system running
 smoothly.

 * Simplicity: make it easy for new engineers to understand the system, by
 removing as much complexity as possible from the system. Reducing accidental
 complexity and finding good abstractions are important.

 * Evolvability: make it easy for engineers to make changes to the system
 in the future. This is closely linked to its simplicity and its abstractions.


### Chapter 2: Data Models and Query Languages

This chapter examines the three most common data models for data storage and
querying: the relational model, the document model, and a few graph-based
models.

The best-known data model today is that of SQL, based on the relational
model proposed by Edgar Codd in 1970. It has good support for joins and
many-to-one and many-to-many relationships, and uses and explicit schema to
validate data on insert.

Document-oriented databases represent data as a set of documents, typically
represented as JSON. For use cases where data mostly comes in self-contained
documents, this makes queries simpler and improves locality. However,
many-to-many relationships don’t fit the model very well.

Graph databases go in the opposite direction, targeting use cases where
anything is potentially related to anything. They model data as a web of
nodes and edges, with labels or properties attached to nodes and edges. They
use declarative query languages to match nodes in the graph.

Document and graph databases typically don’t enforce a schema for the
data they store, which can make it easier to eveolve the data model used by
an application. However, the application will still asuume the data has a
certain shape; it’s just a question of whether it’s explicit (enforced
on write) or implicit (enforced on read).


### Chapter 3: Storage and Retrieval

How do databases store data on disk, and retrieve it later on? Most databases
rely on either a log-structured storage engine or a page-oriented storage
engine such as a B-tree.

The idea behind log-structured storage is to store the data in append-only
files on disk, and maintain an in-memory index for fast retrieval. SSTables
are one implementation of that idea. They work as follows: When a write
comes in it’s added to an in-memory balanced tree called the memtable. When
the memtable gets bigger than some threshold, it’s written to disk as an
SSTable file, which contains the records in sorted order. The database keeps
a sparse index for each SSTable file in memory. To serve a read request,
it first tries to look up the key in the memtable, then the most recent
on-disk segment, then the next-older segment, and so on. From time to time,
a merging and compaction process runs in the background.

The most widely used indexing structure is still the B-tree. It’s essentially
a balanced search tree whose nodes (called pages) are the size of a filesystem
block (typically 4 KiB, but sometimes much larger). B-trees usually have a
very high branching factor: one page has several hundred references to child
pages. This design fites well with the way both hard disks and SSDs work.

As a rule of thumb, LSM-trees are typically faster for writes and B-trees
faster for reads. However, these benchmarks are sensitive to details of the
workload, so it’s worth testing empirically before making a decision.

Two common usage patterns of databases are transaction processing (OLTP, online
transaction processing) and analytics (OLAP, online analytic processing). They
have different access patterns: transaction processing involves large numbers
of low-latency requests that each read and write a small amount of data;
analyticstypically means a smaller number of read-only requests that each
aggregate over a large number of records. Therefore, there are different
databases, or different storage engines for the same database, that cater
to each of these uses.

One technique used for analytics is column-oriented storage. This means,
simply, storing all the values in one column together, instead of all the
values in one row. Queries for analytics often access only a few columns
of a very wide table, sol column-oriented storage drastically reduces the
amount of data to load. Where columns have many repeated values, compression
can further reduce data sizes.


## Chapter 4: Encoding and Evolution

Applications inevitable change over time. In most cases, that means the data
they store or transmit also changes. Making this process easy is a part of
the evolvability introduced in chapter 1.

In large applications, change usually doesn’t happen instantaneously:
with client-side applications, different users will update at different
times; with server-side applications we usually do rolling upgrades where
some nodes will be running the new version while others are still running
the old one. In order for the system to continue running smoothly, we need
compatibility in both directions:

 * Backward compatibility: newer code can read data that was written by
 older code.

 * Forward compatibility: older code can read data that was written by
 newer code.

When you want to write data to a file or send it over the network, you have to
encode it as some kind of self-contained sequence of bytes. Many programming
languages come with built-in support for encoding and decoding data structures
(e.g. pickle in Python). These libraries are convenient, but they have some
deep problems: they’re language-specific, they can easily open up security
holes, and versioning is usualy an afterthought. It’s generally a bad idea
to use them for anything other than very transient purposes.

JSON, XML, and CSV are widely-used textual formats. They are widely-supported
and have the advantage of somewhat human-readable syntax, but they also have
some subtle problems. As one example, JSON doesn’t distinguish between
integers and floating-point numbers and doesn’t specify a precision for
numbers. As a result, integers greater than 2<sup>53</sup> become inaccurate
when read in JavaScript, which uses 64-bit floats as its only number type.

Binary encodings for JSON and XML are more compact than the textual formats,
but they’re not as efficient as the binary formats described below. The
main reason is that they still need to encode field names.

Apache Thrift and Protocol Buffers are binary encoding libraries that require a
schema for data to be encoded and come with code generation tools that produce
classes that implement the schmea in various programming languages. Each field
has an integer tag that’s included in the encoding. Both protocols support
schema evolution in the same way: if a field is missing from a message,
it’s set to the type’s default value; if an unknown field is present,
it’s ignored. This allows adding fields while preserving backwards and
forwards compatibility.

Apache Avro is another binary encoding that uses a schema to specify the
structure of the data being encoded. However, it doesn’t include field or
type tags in the encoded data. Instead, the writer is expected to serialize
the schema along with the data. The reader then resolves differences between
its schema and the writer’s schema according to a set of schema evolution
rules that allow for forward and backward compatibility.

The most common ways of dataflow are dataflow through applications, dataflow
through services, and message-passing dataflow.

When you deploy a nwe version of your application, you usually replace all of
its code, but not its data. This is sometimes summed up as “data outlives
code.” An obvious consequence is that we often need forward compatibility
for databases. However, if we deploy services with rolling upgrades, we can
also have an older version of the code read data written by a newer version,
so we need backward compatibility as well.

For processes communicating over a network, the most common approaches
are REST and RPC. REST means sending messages (typically JSON or XML) over
HTTP. It’s more a design philosophy than a specific protocol.

The idea behind RPC (remote procedure calls) is to make requests to a remote
service look like method calls within a process. Historically, this has led
to a lot of problems because network requests are very different from function
calls in terms of reliability, performance, and compatibility concerns. Modern
implementations such as those built on top of Protocol Buffers, Thrift and
Avro are more explicit about the fact that a remote call is different from
a local one.

Asynchronous message-passing systems are somewhere between RPC and
databases. Services send messages to an intermediary called a message broker,
which stores the message temporarily and delivers it to consumer services. This
decouples sender and recipient to some degree, e.g. a sender can still send
messages if the recipient is temporarily down or overloaded.



## Part II: Distributed Data


### Chapter 5: Replication

Replication means keeping a copy of the same data on multiple machines
that are connected via a network. You might want to do this to keep data
geographically close to your users (and reduce access latency), to allow
the system to continue working even if some of its parts have failed (and
increase availability), or to scale out the number of machines that can
serve read queries (and increase read throughput).

If the data you’re replicating doesn’t change over time, then replication
is easy: just copy the data to every node and you’re done. All of the
difficulty in replication lies in handling changes to replicated data. This
chapter discusses three popular methods for replicating changes between nodes:
single-leader, multi-leader, and leaderless replication.

Each node that stores a copy of the database is called a replica. Every
write to the database must be processed by every replica; otherwise, the
replicas won’t contain the same data anymore. The easiest way to do that is
leader-based replication: one node is designated the leader, and it is the only
node that accepts write requests. The other replicas are known as followers.

Whenever the leader writes new data to its local storage, it also sends the
data change to all its followers as part of a replication log. This can be
synchronous replication, where the leader waits until the follower confirms the
write before reporting success to the client, or asynchronous. In practice,
a common setup is to have exactly one synchronous follower, to make sure
you always have an up-to-date copy of the data on two nodes.

For workloads that consist of mostly reads and a small percentage of writes
(a common pattern on the web), leader-based replication lets you scale up
capacity of read-only requests simply by adding more followers. However, this
approach only realistically works with asynchronous replication -- otherwise,
a single node failure will make the entire system unavailable for writing.

Unfortunately, an application that reads from an asynchronous follower may see
outdated information. This leads to apparent inconsistencies that disappear
as soon as the follower catches up with messages from the leader. This effect
is known as eventual consistency.

Common problems in this scenario relate to different types of consistency:

 * Read-after-write consistency: If an application sends an update to the
 leader, then reads related data from a follower, that data may not reflect
 the update. To the user, it looks as if the data they submitted was lost.

 * Monotonic reads: If an application makes two reads from different replicas,
 the second read might return older data than the first. To the user, this
 looks like the application moving backwards in time.

 * Consistent prefix reads: In some cases (especially in partitioned
 databases), two causally related writes may arrive in reverse order
 at a client. To the user, this can be very confusing, for example if a
 messageing app shows one user’s question after another user’s answer
 to that question.

When working with an eventually consistent system, it’s worth thinking
about how the application behaves if the replication lag increases to several
minutes or even hours. Sometimes, the answer is “no problem.” If it’s
not, application developers need to think about which type of consistency
they need and how to achieve it.

Multi-leader replication is a natural extension to the leader-based replication
model where more than one node accepts writes. Replication still happens in the
same way: each node that processes a write must forward that data change to all
the other nodes. Multi-leader replication rarely makes sense within a single
data center, but if a database has replicas in several different data centers,
having one leader in each data center can improve performance and availability.

The biggest problem with multi-leader replication is that write conflicts can
occur, which means that conflict resolution is required. There are multiple
ways to do this, including giving each write a unique ID and choosing the
one with the highest ID as the winner, merging conflicting values together,
and running application-specific conflict resolution logic.

There is some interesting research into automatically resolving conflicts
caused by concurrent data modification. Conflict-free replicated datatypes
(CRDTs) are a family of data structures that can be concurrently edited
by multiple users and which automatically resolve conflicts in sensible
ways. Mergeable persistent data structures track history explicitly and use a
three-way merge function. Operational transformation is the conflict resolution
algorithm behind collaborative editing applications such as Google Docs.

With leaderless replication, every node accepts both read and write
requests. In a simple implementation, clients send each write request to all
nodes, then wait for a quorum: once `w` nodes have acknowledged the writes,
the client considers it successful and moves on. Read requests are also sent
to multiple nodes; as soon as the client gets back `r` identical values,
it considers that value to be correct. As long as `r + w > n` (the number of
nodes), this should avoid inconsistencies despite nodes eventually becoming
unavailable.

Of course, if a node goes offline, then comes back, it will have some stale
data. How does it catch up on the writes it missed. Two mechanisms are common
with leaderless replication:

 * Read repair: when a client reads data from several nodes in parallel,
 it detects stale responses and issues writes to correct them.

 * Anti-entropy process: some datastores have a background process that
 constantly looks for differences in the data between replicas and copies
 any missing data from one replica to another.


### Chapter 6: Partitioning

Partitioning, also known as sharding, means breaking data up into partitions
such that each piece of data (each record, row, or document) belongs to exactly
one partition. The main reason for wanting to partition data is scalability. By
placing different partitions on different nodes in a shared-nothing cluster,
we can distribute data and query load.

Partitioning is usually combined with replication so that copies of each
partition are stored on multiple nodes for fault-tolerance. The choice of
partitioning scheme is mostly independent of the choice of replication scheme,
so this chapter will ignore replication.

How do you decide which records to store on which node? The goal with
partitioning is to spread the data and query load evenly across nodes. If
that’s not the case, the partitioning is called skewed.

If we assume that each record has a primary key, one option is to partition by
key range: assign a continuous range of keys to each partition. The boundaries
could be chosen manually by an administrator, or automatically. This scheme
has the advantage that range scans can be implemented efficiently, but the
key has to be chosen wisely. For example, if the key is a timestamp and new
records always have the current time, all writes for new records will go to
the same node.

An alternative is partitioning by hash of a key. This is simpler and less
likely to lead to hotspots, but we lose the ability to do efficient range
queries.

If records are only ever accessed via their primary key, we can determine the
partition from that key and use it to route read and write requests to the
partition responsible for that key. The situation becomes more complicated
if secondary indexes are involved. There are two main approaches:

 * A document-partitioned index means each node maintains an index for the
 records in its partition (a local index). This means writes touch only
 a single node, but reads using the index have to use a scatter/gather
 operation to fetch data from all nodes.

 * A term-partitioned index means we construct a global index that covers
 data in all partitions and distribute the index among the nodes. This makes
 writes slower and more complicated, but it makes reads more efficient.

Over time, we’ll need to add new nodes to the cluster to replace failed nodes
or to handle increased data size or query load. This requires rebalancing:
moving load from one node in the cluster to another.

When partitioning by the hash of a key, a simple way to do this is to create
many more partitions than there are nodes and assign several partitions to
each node. When a node is added to the cluster, it can steal a few partitions
from each other node until partitions are again fairly distributed. When a
node is removed, the process works in reverse.

Databases that use key range partitioning usually rely on dynamic partitioning:
when a partition grows to exceed a configured size, it’s split into two. One
of the two halves can then be transferred to another node in order to balance
the load.

Some databases support fully automated rebalancing, but this can be
unpredictable. Rebalancing is an expensive operation that can overload the
network or individual nodes, and it can interact in surprising ways with
automatic failure detection. Some databases generate a suggested partition
assignment automatically, but require an administrator to commit it before
it takes effect.

When a client wants to make a request to a partitioned database, how does it
know which node to connect to? This is an instance of a more general problem
called service discovery. At a high level, the knowledge of which parittion
is assigned to which node can live on the client, in a separate routing tier,
or on the nodes themselves.

In all cases, the key problem is: how does the component making the
routing decisions learn about change in the assignment of partitions to
nodes? Databases that use automatic rebalancing either rely on a separate
coordination service such as ZooKeeper or use a gossip protocol among the
nodes to disseminate changes in cluster state.


### Chapter 7: Transactions

In the harsh reality of data systems, many things can go wrong: hardware
fails, the application crashes, networks get cut off, and multiple clients
accessing the same database at the same time interact in surprising ways. In
order to be reliable, a system has to deal with these faults and ensure they
don’t cause failure of the whole system. Implementing these fault-tolerance
mechanisms requires a lot of careful thinking about what could go wrong and
a lot of testing to ensure the solution actually works.

Transactions are a way to simplify the programming model for applications,
providing certain safety guarantees so the application is free to ignore
certain potential error scenarios and concurrency issues.

A transaction is a way for an application to group several reads and writes
together into a logical unit. Conceptually, all the reads and writes are
executed as one operation: either the entire transaction succeeds, or it
fails and the application can safely retry.

The safety guarantees of transactions are often described by the acronym ACID:

 * Atomicity means when a transaction makes multiple writes, either none or all
 of them take effect. If there’s a fault part-way through the transaction,
 the database has to discard or undo any writes that have been made so far.

 * Consistency means certain invariants (e.g. a unique constraint) must
 always be true for the database. This is arguably the application’s
 responsibility, so perhaps it shouldn’t be part of the acronym.

 * Isolation means concurrently executing transactions are isolated from
 each other, so each can pretend it’s the only transaction running on the
 database. In practice this means different things depending on the database
 and how the settings used.

 * Durability is the promise that once a transaction has committed
 successfully, any data it has written will not be forgotten (e.g. in the
 event of a power outage).

These guarantees matter even when we’re only updating a single object. For
example, if you’re storing a JSON document and the network connection is cut
off after you’ve set half of the data, you wouldn’t want an unparseable
JSON fragment stored in the database. However, for most applications
multi-object transactions are needed to avoid complicated error handling.

In theory, isolation levels should let you pretend that no concurrency
is happening. In practice, many databases use weak isolation levels for
performance reasons. To evaluate these, we need to look at different types
of race conditions:

 * Dirty reads: a client reads another client’s writes before they have
 been committed.

 * Dirty writes: a client overwrites data that another client has written
 but not yet committed.

 * Read skew: a client that makes two reads during a transaction gets
 inconsistent results because another client’s transaction has been
 committed in the meantime.

 * Lost update: two clients concurrently perform a read-modify-write cycle
 and one overwrites the other’s write without incorporating its changes.

 * Write skew: a transaction reads something, makes a decision based on the
 value it saw, and writes the decision to the database. However, by the time
 the write is made, the premise of the decision is no longer true.

Common transaction isolation levels are:

 * Read committed prevents dirty reads and dirty writes. This is the default
 setting in many relational databases, including PostgreSQL 13.

 * Snapshot isolation also prevents read skew. The idea is that each
 transaction reads from a consistent shapshot of the databse.

 * Serializable is the strongest isolation level. It means even though
 transactions may be executed concurrently, the end result is the same as
 if they had run one at a time.

Three techniques are used to implement serializable execution. Actual serial
execution can work well if the transactions run entirely on the database
server, like stored procedures. Two-phase locking (2PL) uses locks so readers
block writers, writers block readers, and writers block writers. All this
locking can considerably reduce concurrency, but for around 30 years, 2PL was
the only widely-used algorithm for serializability in databases. A more recent
development is serializable snapshot isolation (SSI), which uses optimistic
locking to allow transactions to proceed without blocking until they commit.


### Chapter 8: The Trouble with Distribued Systems

When you’re writing a program on a single computer, there is no fundamental
reason why it should be flaky: when the hardware is working correctly,
the same operation always produces the same result. If there is a hardware
problem, the result is usually a total system failure. This is a deliberate
choice in the design of computers. If an internal fault occures, we prefer
a computer to crash completely rather than returning a wrong result, because
wrong results are difficult and confusing to deal with.

When you’re writing software that runs on several computers, connected by
a network, the situation is fundamentally different. There amy well be some
parts of the system that are broken while other parts are working fine. The
difficulty is that these partial failures are nondeterministic: if you try
to do anything involving multiple nodes and the network, it may sometimes
work and sometimes unpredictably fail. In addition, you may not even know
whether something succeeded or not, as the time it takes for a message to
travel accross a network is also nondeterministic!

This nondeterminism and possibility of partial failure is what makes
distributed systems hard to work with.

There’s a spectrum of philosophies on how to build large-scale computing
systems. At one end of the scale are supercomputers (or high-performance
computing, HPC); at the other end is cloud computing. With these philosophies
come very different approaches to handling faults. When a node in a
supercomputer fails, a common solution is to simply stop the entire workload,
repair the node, and restart the computation from a checkpoint. Thus, a
supercomputer is more like a single-node computer than a distribued system:
it deals with partial failure by letting it escalate into total failure --
if any part of a system fails, just let everything crash (like a kernel
panic on a single machine).

This book is focused on shared-nothing systems: a bunch of machines that can
only communicate by making requests over a network, usually an asynchronous
packet network. In this kind of network, one node can send a message (packet)
to another node, but the network gives no guarantee at to when it will arrive,
or whether it will arrive at all.

The sender can’t even tell whether the packe was delivered: the only option
is for the recipient to send a response message, which may in turn be lost
or delayed. This is usually handled with a timeout. After some time you give
up waiting and assume that the response is not going to arrive. However,
when a timeout occurs, you still don’t know whether the remote node got
your request or not.

This is not the only kind of network there is. For example, the traditional
(non-cellular, non-VoiP) phone network avoids all this because it’s
a synchronous network. When you make a call, it establishes a circuit:
a fixed, guaranteed amount of bandwidth allocated for the call, anong the
entire route between the two callers. We could use synchronous networks to
connect computers, but there’s a trade-off between latency and resource
utilization: by allocating bandwidth dynamically, asynchronous networks
utilize it as much as possible.

Distributed applications depend on clocks and time in various ways. We
always need them for timeouts, distributed databases may use timestamps to
order events, locks in distributed systems are often implemented as leases
that are valid during a specific interval, and failover from a node that’s
become unreachable requires measuring how long it has been unreachable.

Modern computers have two types of clocks:

 * A time-of-day clock returns the current date and time according to
 some calendar. They are usually synchronized with NTP, which means that a
 timestamp from one machine more-or-less means the same as a timestamp from
 another machine. It also means if the local clock is too far ahead of the
 NTP server, it may be reset and appear to be jumping backward.

 * A monotonic clock is suitable for measuring a duration such as a timeout
 or a service’s response time. It is guaranteed to always move forward.

Time-of-day clocks aren’t nearly as reliable or accurate as you might
hope. The quartz clock in a computer drifts (runs faster or slower than
it should), and NTP synchronization can only be as good as the network
delay. Further, if NTP doesn’t work at all (e.g. because of a misconfigured
firewall), the clock may drift for a long time without the problem being
noticed.

Monotonic clocks are generally unaffected by these problems because
the monotonic clock on one node is ompletely independent of that on
another. However, one thing to keep in mind is that the monotonic clock can
appear to jump forward at any time in a program’s execution if execution
is suspended. This may happen for several reasons:

 * In a garbage-collected language, the garbage collector occasionally has
 to stop all threads.

 * The operating system may suspend the thread or process to context-switch
 to another. If the machine is under heavy load, it may take some time until
 it runs again.

 * In a virtualized environment, a virtual machine can be suspended,
 pausing the execution of all processes and saving the contents of memory
 to disk. This is sometimes used for live migration of virtual machines from
 one host to another.

Algorithms designed to solve distributed systems problems need to tolerate the
various faults discussed in this chapter. To formalize the kinds of faults
that we expect to happen, we define a system model, which is an abstraction
that defines what things an algorithm may assume.

With regards to timing assumptions, three system models are in common use:

 * The synchronous model assumes bounded network delay, bounded process
 pauses, and bounded clock error. This is not a realistic model of most
 practical systems.

 * The partially synchronous model assumes that a system behaves like a
 synchronous system most of the time, but it sometimes exceeds the bounds
 for network delay, process pauses, and clock drift.

 * The asynchronous model says that an algorithm is not allowed to make any
 timing assumptions.

Besides timing issues, we have to consider node failures. The most common
system models are:

 * Crash-stop faults: An algorithm may assume that a node can fail in only
 one way, namely by crashing. After a node crashes it never comes back.

 * Crash-recovery faults: We assume that nodes may crash at any moment, and
 perhaps start responding again after some unknown time. Nodes are assumed
 to have stable storage that is preserved across crashes.

 * Byzantine (arbitrary) faults: Nodes may do absolutely anything, including
 trying to trick and deceive other nodes.

For modeling real systems, the partially synchronous model with crash-recovery
faults is generally the most useful model.


### Chapter 9: Consistency and Concensus

The easiest way to deal with all of the things that can go wrong in a
distributed system is to just let the entire service fail and show the
user an error message. If that’s unacceptable, we need to find ways of
tolerating faults.

Most replicated databases provide at least eventual consistency, which means
that if you stop writing to the database and wait for some unspecified length
of time, then eventually all read requests will return the same value. However,
this is a very weak guarantee -- it doesn’t say anything about when the
replicas will converge.

Linearizability is a much stronger guarantee. The basic idea is that the
database gives the illusion that there is only one replica, so every client has
the same view of the data, and you don’t have to worry about replication
lag. It still allows for concurrency: each request takes up a timespan
between when it’s sent by the client and when the response it received
by the client, and those timespans can overlap. We can imagine each request
being processed at some definite time within that timespan. Linearizability
means there is always a way to select those times that’s consistent with
having only one copy of the data.

Linearizability is appealing because it’s easy to understand, but it
has the downside of being slow, especially in environments with large
network delays. As a result, most distributed databases don’t provide
linearizability.

Another way to look at linearizability is to say it provides a total order
for when events happen in the system. This order will preserve causality:
if an earlier event (`set x=1`) influences a later even (`read x`),
the ordering will preserve causality. However, causality only implies
a partial order -- if two events don’t influence each other, they’re
incomparable. There is an efficient algorithm for maintaining a causal ordering
called Lamport timestamps, which assings a number (a “logical timestamp“)
to each operation in such a way that if an operation is causally before
another, it will have a lower timestamp.

However, this is not quite sufficient to solve many common problems in
distributed systems. For example, consider a system that needs to ensure
that a username uniquely identifies a user account. If two users concurrently
try to create an account with the same username, one of them should fail.

A total ordering would let us determine the winner after the fact: if two
accounts with the same username are created, pick the one with the lower
timestamp. But a node that has just received a request to create a user
account needs to decide right now if the request should succeed or fail. In
order to be sure that no other node is currently creating an account with
the same username, it would have to check with every other node to find out
which timestamps it has generated. If one node is unreachable, this system
would grind to a halt. That’s not the kind of fault-tolerant system we want.

The easiest way to deal with all of the things that can go wrong in a
distributed system is to just let the entire service fail and show the
user an error message. If that’s unacceptable, we need to find ways of
tolerating faults.

Most replicated databases provide at least eventual consistency, which means
that if you stop writing to the database and wait for some unspecified length
of time, then eventually all read requests will return the same value. However,
this is a very weak guarantee -- it doesn’t say anything about when the
replicas will converge.

Linearizability is a much stronger guarantee. The basic idea is that the
database gives the illusion that there is only one replica, so every client has
the same view of the data, and you don’t have to worry about replication
lag. It still allows for concurrency: each request takes up a timespan
between when it’s sent by the client and when the response it received
by the client, and those timespans can overlap. We can imagine each request
being processed at some definite time within that timespan. Linearizability
means there is always a way to select those times that’s consistent with
having only one copy of the data.

Linearizability is appealing because it’s easy to understand, but it
has the downside of being slow, especially in environments with large
network delays. As a result, most distributed databases don’t provide
linearizability.

Another way to look at linearizability is to say it provides a total order
for when events happen in the system. This order will preserve causality:
if an earlier event (“set x=1”) influences a later even (“read x”),
the ordering will preserve causality. However, causality only implies
a partial order -- if two events don’t influence each other, they’re
incomparable. There is an efficient algorithm for maintaining a causal ordering
called Lamport timestamps, which assings a number (a “logical timestamp“)
to each operation in such a way that if an operation is causally before
another, it will have a lower timestamp.

However, this is not quite sufficient to solve many common problems in
distributed systems. For example, consider a system that needs to ensure
that a username uniquely identifies a user account. If two users concurrently
try to create an account with the same username, one of them should fail.

A total ordering would let us determine the winner after the fact: if two
accounts with the same username are created, pick the one with the lower
timestamp. But a node that has just received a request to create a user
account needs to decide right now if the request should succeed or fail. In
order to be sure that no other node is currently creating an account with
the same username, it would have to check with every other node to find out
which timestamps it has generated. If one node is unreachable, this system
would grind to a halt. That’s not the kind of fault-tolerant system we want.

This all leads us to consensus, which is one of the most important and
fundamental problems in distributed computing. Informally, consensus means
getting several nodes to agree to something.

It’s usually formalized as follows: One or more nodes may propose values,
and the consensus algorithm decides on one of the value. A consensus algorithm
must satisfy the following properties:

 * Uniform agreement: no two nodes decide differently.

 * Integrity: no node decides twice.

 * Validity: if a node decides value v, then v was proposed by some node.

 * Termination: every node that doesn’t crash eventually decides some vlue.

The best-known fault-tolerant consensus algorithms are Viewstamped Replication
(VSR), Paxos, Raft, and Zab. These algorithms are famously hard to implement,
so it’s advisable to use an existing implementation such as Zookeeper or
etcd (which implement Zab and Raft, respectively).



## Part III: Derived Data


## Chapter 10: Batch Processing

The first two parts of the book talked a lot about requests and queries,
and the corresponding responses and results. This style of data processing
is assumed in many modern data systens: you ask for something, or you
send an instruction, and some time later the system (hopefully) gives you
an answer. Databases, caches, search indexes, web server, and many other
systems work this way.

But other approaches have their merit too. We can distinguish three types
of systems:

 * Services (online systems): A service waits for a request from a client, then
 tries to handle it as quickly as possible and send back a response. Response
 time and availability are very important.

 * Batch processing systems (offline systems): A batch processing system
 takes a large amount of data, runs a job to process it, and produces some
 output data. The primary performance measure is usually throughput.

 * Stream processing systems (near-realtime systems): Like a batch processing
 system, a stream processor consumes inputs and produces outputs. However,
 a stream job operates on events shortly after they happen, whereas a batch
 job operates on a fixed set of input data.

Standard Unix tools and pipelines illustrate some important concepts in batch
processing. As an example, the following command reads an nginx access log
file and produces a list of the five most popular pages:

    cat /var/log/nginx/access.log |
        awk '{print $7}' |
        sort |
        uniq -c |
        sort -r -n |
        head -n 5

This example illustrates some of the principles behind Unix pipelines:

 * Chain of commands instead of custom programs: The pipeline is built by
 combining some simple tools, each of which does one thing well.

 * A uniform interface: If you expect the output of one program to become
 the input to another program, it means those programs must use the same data
 format -- inother words, a compatible interface. In Unix, that interface is
 a file (or, more precisely, a file descriptor). A file is just an ordered
 sequence of bytes. By convention, many Unix programs treat this sequence of
 bytes as ASCII text, and that text is usually treated as a list of records
 separated by `\n` (the newline character).

 * Separation of logic and wiring: Another characteristic feature of
 Unix programs is their use of standard input (stdin) and standard output
 (stdout). By default, stdin comes from the keyboard and stdout goes to
 the screen, but you can use I/O redirection and pipes so stdin comes
 from a file, stdout goes to another program, and so on. Separating the
 input/output wiring from the program logic is a form of loose coupling;
 it makes it easier to compose small tasks into a bigger system.

 * Sorting versus in-memory aggregation: If you were to write a single-program
 implementation of the pipeline above, you’d probably use a hash table
 to keep track of URLs and thus aggregate the data in memory. The pipeline
 instead sorts the data in order to aggregate it. The sort utility in Linux
 automatically spills larger-than-memory datasets to disk and parallelizes
 sorting across multiple CPU cores, which makes this approach surprisingly
 scalable.

MapReduce is a system for distribued batch processing that’s a little like
Unix pipelines: it’s a fairly blunt, brute-force, but suprisingly effective
tool. A single MapReduce job is comparable to a single Unix process: it takes
one or more inputs and produces one or more outputs, usually without modifying
the input and without any side effects other than producing the output.

To create a MapReduce job, you need to implement two callback functions,
the mapper and the reducer:

 * The mapper is called once for every input record, and its job is to
 extract a key and a value. For each input, it can generate any number of
 key-value pairs. It does not keep any state from one input to the next.

 * The MapReduce framework takes the key-value pairs produced by the mappers,
 collects all the values belonging to the same key, and calls the reducer
 with an iterator over that collection of values. The reducer can produce
 output values.

Hadoop is a popular implementation of MapReduce that uses JVM classes for
mapper and reducer implementations, and a distributed filesystem called HDFS
(Hadoop Distributed Filesystem) to distribute input and output data. HDFS
replicates file blocks on multiple machines to allow Hadoop to tolerate
machine and disk failures.

A common need in processing large datasets is to perform a join -- combining
records such that a key in one record matches a key in another. How would
we do a join in MapReduce? We have two basic options:

 * In a reduce-side join, the mappers select the key used for th join as
 the map-reduce key. The reducers then receive the matching records together
 and can easily join them. A similar technique is used for grouping (GROUP
 BY in SQL).

 * A map-side join does the join using only mappers. That means we can
 skip the expensive sorting, copying to reducers, and merging of reducer
 inputs, but it only works if we can make certain assumptions about the
 input data. One type is a broadcast hash join, where a large dataset is
 joined with a small dataset. The small dataset is copied (“broadcast”)
 to all nodes, so the mapper has all the data it needs to do the join.

When MapReduce was published, it was -- in some sense -- not at all
new. Massively parallel processing (MPP) databases had implemented the same
techniques more than a decade earlier. However, Hadoop and other MapReduce
implementations differe from MPP databases in some important ways: they
don’t impose a specific data model on inputs and outputs; they run arbitrary
user-supplied code; and they are designed to cope well with frequent node
failures. This makes the combination of MapReduce and a distributed filesystem
more like a general-purpose operating system that can run arbitrary programs.

Although MapReduce received a lot of hype in the late 2000s, it is just one
among many possible programming models for distributed systems. It’s a good
model to study because it’ß simple in the sense of easy to understand,
but is’s not easy to use. Implementing a complex processing job using just
the MapReduce API is actually quite hard and laborious. Various higher-level
programming models (Pig, Hive, Cascading, Crunch) have been created as
abstractions on top of MapReduce.

However, there are also problems with the MapReduce execution model itself,
which are not fixed by adding another level of abstraction and manifest
themselves as poor performance for some kinds of processing. The main issue
is materialization of intermediate state: in a pipeline of MapReduce jobs,
each job’s outut is writen out (to a distributed, replicated filesystem),
then read in by the next job, but only after all tasks in the preceding job
have completed.

Several new execution engines for distributed batch computation have been
developed, the most well-known of which are Spark, Tez, and Flink. These
are called dataflow engines. Like MapReduce, they work by repeatedly
calling a user-defined function to process one record at a time on a single
thread. However, those functions need not take the strict roles of alternating
map and reduce; instead the dataflow engine provdes several different options
for connecting one operator’s output to another’s input.

The bulk synchronous parallel (BSP) model of computation, also called the
Pregel model, is a model for distributed processing of graph data. In this
model, a function is called for each vertex. The function can send messages
to other vertices, which become the input of each function in the next
iteration. The computation continues this way until no more messages are sent.

Unfortunately, parallel or distributed computation in this model is often
very inefficient because of huge communication overhead. If your graph can
fit into memory (or even on disk) on a single computer, it’s quite likely
that a single-machine algorithm will out-perform a distributed batch process.


### Chapter 11: Stream Processing

Batch processing produces derived data from input that is bounded -- i.e. of
a known and finite size -- so the batch process knows when it has finished
reading its input. The idea of stream processing is to process data that is
unbounded because it arrives gradually over time.

In a stream processing context, a record is usually known as an event. An
event is generated once by a producer (also known as a publisher or sender)
and processed by zero or more consumers (subscribers or recipients). Related
events are often groupd into a topic or stream.

In principle, a file or database is sufficient to connect producers and
consumers: a producer writes events to the datastore, and each consumer
periodically polls the datastore to check for events that have appeared since
it last ran. However, for continual processing with low delays, this polling
becomes expensive. Instead, it is better for consumers to be notified when
new events appear.

A common approach is to use a messaging system: a producer sends a message
containing the event, which is then pushed to consumers. Most messaging systems
allow multiple producer nodes to send messages to the same topic and multiple
consumer nodes to receive messages in a topic. This publish/subscribe model
can be implemented in different ways. To distinguish them, it’s useful to
ask two questions:

 1. What happens if the producers send message fastser than the consumers
 can process them? Broadly speaking, there are three options: the system
 can drop messages, buffer messages in a queue, or apply back-pressure to
 block the producers from sending more messages.

 2. What happens if nodes crash or temporarily go offline -- are any
 messges lost? Durability may require some combination of writing to disk
 or replication, which will affect performance.

Some messaging systems use direct communication between producers and
consumers, but it’s more common to send messages via a message broker,
which is essentially a kind of database that is optimized for handling
message streams.

When multiple consumers read messages in the same topic, two main patterns of
messaging are used. Load balancing means each message is delivered to one of
the consumers, so the consumers can share the work of processing the messages
in the topic. Fan-out means each message is delivered to all of the consumers.

To ensure that messages aren’t lost when a consumer crashes, message
brokers wait for an acknowledgement from the client before they remove a
message from the queue. If the connection to a client is closed or times out
without the broker receiving an acknowledgment, it assumes that the message
was not processed and it will try to deliver the message again. When combined
with load balancing, this redelivery behavior has the surprising effect that
messages arrive out-of-order.

A log-based message broker combines event streams with durable, log-structured
storage. It appends messages from producers to the end of the log, and sends
messages to consumers by reading the log sequentially. The log is usualy
partitioned to take advantage of multiple disks or multiple nodes.

Change data capture (CDC) is the process of observing all data changes written
to a database and extracting them as an event stream. In this scenario,
the database is the system of record and the log consumers are derived data
systems. CDC can be useful, for example, to build a search index, to update
a data warehouse, or to invalidate a cache.

Event sourcing is a technique developed in the domain-driven design (DDD)
community that incorporates some useful ideas for streaming systems. It
involves storing all changes to the application state as a log of change
events. Unlike with CDC, these events are designed to reflect things that
happened at the application level rather than low-level database changes. Event
sourcing distinguishes between events and commands: an event is an immutable
part of the log, while a command is a request from the user that may still
be rejected by the system.

An immutable log of events like that has several advantages: you can re-build
the current state of the system from the log (e.g. for a new software version),
you can derive several different read-oriented views from the same log, and
you can often capture more information in a high-level event than would be
recorded in a database transaction. On the other hand, an immutable history
can grow prohibitively large, and sometimes you really do need to delete data,
e.g. to comply with GDPR.

The rest of this chapter discusses processing streams to produce other,
derived streams. A piece of code that processes streams like this is known
as an operator or job. Some common applications of stream processing are:

 * Complex event processing (CEP) is an approach developed in the 1990s
 for analyzing event streams, in particular searching for certain event
 patterns. CEP lets you specify rules, often in a high-level declarative
 query language, to describe patterns of events. When a match is found,
 the engine emits a complex event (hence the name) with the details of the
 event pattern detected.

 * Search on streams implements searching for individual events based on
 complex criteria, such as full-text search.

 * Stream analytics compute aggregates and statistical metrics over a large
 number of events. They usually aggregate over a time interval known as
 a window.

 * Stream processing can also be a way for services to communicate with each
 other, as an asynchronous alternative to RPC.

With CEP and searching on streams, the relationship between queries and data
is reversed compared to a normal database: queries are stored persistently
while data is always transient.

Joins are a common task in stream processing. We can distinguish three types
of joins:

 * Stream-stream joins: Both input streams consist of activity events,
 and the join operator searches for related events that occur within some
 window of time.

 * Stream-table joins: One input stream consists of activity events, while
 the other is a database changelog. The changelog is used to keep a local copy
 of the database up to date, which is then used to join with incoming events.

 * Table-table joins: Both input streams are database changelogs. Every
 change on one side isjoined with the latest state of the other side. The
 result is a stream of changes to the view of the join between the two tables.
