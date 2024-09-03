---
params:
  math: true
title: "The Raft Consensus Protocol"
description: "An approachable consensus protocol for replicated state machine."
summary: "An approachable consensus protocol for replicated state machine."
tags: ["Distributed Systems"]
date: 2024-07-14
author: Ishaan Dham
---

## Introduction

Consensus involves a collection of machines working together as a coherent group while being tolerant of failures.

Imagine a large bank, with branches distributed across cities. Each branch maintains its local database storing customer information. Each database must be consistent with customer data regardless of where a transaction takes place. Given no faults i.e no packet loss, network partitions, server failures we could let each node communicate with every other node before committing a transaction. However, in the real world this is rarely achievable. By using a consensus protocol this distributed system can maintain data consistency and fault tolerance (ability to function with majority of the nodes alive).

This blog introduces the Raft consensus protocol that is used to manage a replicate log in a distributed system. A replicated log allows each node to maintain a fault-tolerant state machine. In the banking example, the state machine is the database, or in a distributed key-value store it would be the hashmap used to store the key-values. By replicating the state machine across nodes, clients appear to interact with a single, reliable state machine, despite failures occurring in certain nodes.

Paxos (multi-Paxos) has been the consensus algorithm of choice since 1989 and is used in multiple systems like Spanner, DynamoDB and Apache Cassandra. Raft is equivalent to Paxos in fault-tolerance and performance. It's impact primarily stems from its understandability, approachability, and intuitive implementation.

## Terminology

The terms defined below are commonly used when talking about distributed systems, and are used in other sections of this writeup. 

**Non-Byzantine faults** - refers to failures such as network partitions, message loss, or server crashes that are expected in a distributed system. Whereas, Byzantine faults include malicious or arbitrary behavior of nodes, and require more complex fault tolerant mechanisms.

**Linearizability** - a strong consistency model that ensures that operations appear to take effect instantaneously at some point between their invocation and completion. This means that a distributed system following linearizable semantics ensures that a operation appears to be executed exactly once, atomically, and in the correct order - similar to how requests made to a single node would be executed.

**CAP theorem** - A distributed system can only guarantee either strong consistency or high availability under network partitions.

### Common Properties of Consensus Algorithms

1. **Safety** - A system never returns an incorrect result under all non-Byzantine conditions.

2. **Availability** - The system is fully functional as long as majority of the servers are operational.

3. **Not Time Dependent** - The algorithm does not rely on time to maintain consistency due to the high possibility of faulty clocks and large delays.
  (*Google ignores this property for* - [Spanner!](https://static.googleusercontent.com/media/research.google.com/en//archive/spanner-osdi2012.pdf))


Raft adheres to all of the above properties.

## Methodology

The Raft algorithm uses a strong leader approach to maintain consensus. Time is divided into terms, and for each term a leader is elected. The leader is responsible for managing the replicated log. It does so by accepting requests from clients, replicating the log entries across other severs, and maintaining consistency by applying log entries to the state machine in a safe manner.
If the leader fails or is disconnected, a new leader is elected.

At any given time each server is in one of three states: leader, follower, or candidate. Typically, there exists a leader and the remaining servers are followers.

The nodes use the AppendEntries and RequestVote RPCs to communicate with each other. A leader sends AppendEntries RPCs to other nodes to replicate log entries, and also periodic heartbeats (AppendEntries RPC with no log entries) to maintain authority. The RequestVote RPC is used by 

(See [figure 2](https://raft.github.io/raft.pdf) for details about the RPCs)


by decomposing the major pieces of the algorithm into relatively independent sub-problems.


### Leader Election

* Heartbeat Mechanism - 

* Election Timeout - randomized to ensure split votes (multiple servers in *candidate* state) are rare.

* Voting - about RequestVote RPC

### Log Replication

* 

Mention log compaction here

### Safety

#### Demo

This visualization demonstrates the Raft protocol in 5 node cluster. It allows interactions such as stopping, making requests, and timing out a given node. 

<iframe src="https://raft.github.io/raftscope/index.html" title="raft visualization" aria-hidden="true" style="border: 0; width: 800px; height: 580px; margin-bottom: 20px"></iframe>

## Implementation and Performance

Mention client interaction here - requests go to the leader node


### Correctness

TLA+

### Performance



## Conclusion


Raft is used in - 

Ectd - 

Apache Kafka - https://cwiki.apache.org/confluence/display/KAFKA/KIP-500%3A+Replace+ZooKeeper+with+a+Self-Managed+Metadata+Quorum

Hashicorp Consul uses Raft



Readers that are interested in trying to implement Raft themselves 

1. [MIT Distributed Systems RAFT Lab](https://pdos.csail.mit.edu/6.824/labs/lab-raft.html)

2. Maelstrom workbench that uses the Jepsen testing library to simulate network faults and latency - https://github.com/jepsen-io/maelstrom/blob/main/doc/06-raft/index.md



## References

1. [Raft paper](https://raft.github.io/raft.pdf)

2. [Raft visualization](https://github.com/ongardie/raftscope)

3. [Spanner](https://static.googleusercontent.com/media/research.google.com/en//archive/spanner-osdi2012.pdf) 