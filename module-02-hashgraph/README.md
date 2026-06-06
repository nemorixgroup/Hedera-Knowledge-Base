# Module 02 - The Hashgraph Algorithm
 
> The technical heart of Hedera. This module covers the Hashgraph data structure, the gossip-about-gossip protocol, virtual voting, asynchronous Byzantine Fault Tolerance (aBFT), and why Hashgraph represents a fundamental advancement over traditional blockchain consensus.
 
## Table of Contents
 
- [Blockchain vs Hashgraph: The Core Difference](#blockchain-vs-hashgraph-the-core-difference)
- [The DAG: Hashgraph's Data Structure](#the-dag-hashgraphs-data-structure)
- [Gossip About Gossip](#gossip-about-gossip)
- [Virtual Voting](#virtual-voting)
- [Asynchronous Byzantine Fault Tolerance (aBFT)](#asynchronous-byzantine-fault-tolerance-abft)
- [Fairness: A Mathematical Guarantee](#fairness-a-mathematical-guarantee)
- [Cryptographic Security](#cryptographic-security)
- [Performance and Throughput](#performance-and-throughput)
- [Summary Comparison Table](#summary-comparison-table)
- [Key Takeaways](#key-takeaways)
- [References](#references)
 
## Blockchain vs Hashgraph: The Core Difference
 
To understand Hashgraph, you first need to understand the fundamental limitation of traditional blockchains.
 
### How a blockchain works (and where it fails)
 
In a blockchain, transactions are grouped into blocks. Blocks are chained linearly, one after another. If two blocks are produced at the same time (which happens constantly), the network must choose one and discard the other. This is called a fork resolution.
 
```
Blockchain - linear chain (branches get discarded):
 
[Block 1] --> [Block 2] --> [Block 3] --> [Block 4]
                  |
                  +--> [Block 3b] (DISCARDED - stale block)
```
 
This creates three structural problems:
 
| Problem | Description |
|---------|-------------|
| **Wasted work** | Discarded blocks represent real computation and bandwidth that is permanently lost |
| **Artificial slowdown** | Blockchains need Proof of Work or similar mechanisms to slow block creation, preventing forks from growing faster than they can be resolved |
| **Probabilistic finality** | A transaction is never truly final - it just becomes increasingly unlikely to be reversed as more blocks are added on top of it |
 
### How Hashgraph solves this
 
Hashgraph uses a fundamentally different data structure: a **Directed Acyclic Graph (DAG)**. Instead of discarding competing branches, every event is preserved and woven into a single unified ledger.
 
```
Hashgraph - DAG (nothing is discarded, everything is woven together):
 
  A1 --\         A2 --\         A3
        \               \
  B1 ----+--> B2 --------+--> B3
        /               /
  C1 --/         C2 --/         C3
 
Every event connects forward to future events via two parent hashes.
No branch is ever discarded. 100% efficiency.
```
 
## The DAG: Hashgraph's Data Structure
 
The Hashgraph is a DAG where each node represents an **event**. Every event is a data structure that contains:
 
- A **timestamp** (when the event was created)
- An array of zero or more **transactions**
- A **self-parent hash** - the hash of the last event this node created
- An **other-parent hash** - the hash of the last event received from another node
- A **cryptographic signature** from the creator
Because every event links to two parents via cryptographic hashes, the entire history of communication between nodes is encoded into the graph itself. This is what makes gossip-about-gossip possible.
 
## Gossip About Gossip
 
This is the communication layer of the Hashgraph algorithm. It operates in two layers.
 
### Layer 1: The Gossip Protocol
 
The gossip protocol is how information spreads through the network. It works like real-world gossip:
 
1. Alice creates a transaction (TX1) - only she knows it
2. Alice randomly picks Bob and shares everything she knows - now Bob knows TX1
3. Bob randomly picks Carol and shares everything he knows - now Carol knows TX1
4. Simultaneously, Alice picks David, Bob picks Eve, and so on
5. Information spreads **exponentially** - in O(log N) rounds, every node knows TX1
```
Round 1:  Alice knows TX1
Round 2:  Alice + Bob know TX1        (Alice told Bob)
Round 3:  Alice + Bob + Carol + David know TX1
Round 4:  Most of the network knows TX1
Round 5:  Entire network knows TX1
```
 
This is extremely bandwidth-efficient: no central server, no broadcast flooding. Each sync is between exactly two nodes.
 
### Layer 2: Gossip About Gossip
 
Here is the core insight of the algorithm. Nodes do not just share transactions - they share the **history of who told what to whom**.
 
Each time two nodes sync (a "gossip sync"), both nodes record that sync as a new **event** in the Hashgraph. This event includes:
 
- What transactions were shared
- The self-parent hash (the creator's last event)
- The other-parent hash (the other node's last event)
Because every event references its parents via cryptographic hashes, the graph grows as a permanent, tamper-proof record of all communication history. Every node, by looking at the graph, can reconstruct exactly what every other node knew at any given point in time.
 
This property - every node knowing what every other node knows - is what makes virtual voting possible without sending any additional messages.
 
## Virtual Voting
 
Traditional distributed consensus algorithms require nodes to explicitly send vote messages to each other. This consumes significant bandwidth and adds latency.
 
Hashgraph eliminates this entirely through **virtual voting**.
 
### How it works
 
Because every node has a complete copy of the Hashgraph (thanks to gossip-about-gossip), every node can independently calculate - with mathematical certainty - how every other node would vote on any given transaction, without asking them.
 
There is no need to send vote messages across the network. Each node runs the same deterministic algorithm on the same graph and arrives at the same consensus result independently.
 
```
Traditional BFT:
Node A --> sends VOTE to B, C, D, E...
Node B --> sends VOTE to A, C, D, E...
(O(N^2) messages for N nodes)
 
Hashgraph Virtual Voting:
Each node calculates locally how others would vote
0 additional messages needed for voting
```
 
The algorithm uses concepts called **"strongly seeing"** and **"famous witnesses"** to determine when enough of the network has received an event that consensus can be mathematically computed. These are defined precisely in the Hashgraph whitepaper and proven correct via the Coq formal proof system.
 
### Consensus on timestamps
 
Through virtual voting, every transaction receives a **consensus timestamp** - the median of the timestamps recorded by all the nodes that first received that transaction. This timestamp is agreed upon by the entire network without any node having special authority over it.
 
Once a transaction has a consensus timestamp, it is ordered relative to all other transactions and achieves **absolute finality** - it cannot be changed or reversed.
 
## Asynchronous Byzantine Fault Tolerance (aBFT)
 
aBFT is the highest security level achievable in distributed systems. Hashgraph is the only public distributed ledger that has formally proven this property.
 
### The Byzantine Generals Problem
 
The Byzantine Generals Problem is a classic thought experiment in distributed systems. Imagine several generals surrounding an enemy city, each commanding a separate army. They must coordinate - either all attack or all retreat. They communicate only via messengers. The challenge: some generals (or messengers) may be traitors sending false information. How can the honest generals reach a reliable agreement?
 
This maps directly to distributed networks:
- Generals = network nodes
- Messengers = network messages
- Traitors = malicious nodes or attackers
### Degrees of Byzantine Fault Tolerance
 
Not all BFT systems are equal:
 
| Type | Description | Weakness |
|------|-------------|----------|
| **Non-BFT** | Cannot handle malicious nodes at all (e.g., standard Paxos) | Breaks under any attack |
| **Partial BFT** | Tolerates some attacks but assumes message timing | Fails if attacker controls network timing |
| **Synchronous BFT** | Assumes messages arrive within a known time window | Vulnerable to botnets, DDoS, and network manipulation |
| **Asynchronous BFT (aBFT)** | Works even if an attacker controls message delivery | Requires only that >2/3 of stake is honest |
 
### Why aBFT matters
 
Partial or synchronous BFT systems assume that messages are delivered within a fixed time window (e.g., 10 seconds). This assumption can be broken in the real world by botnets, DDoS attacks, or sophisticated network-level adversaries.
 
aBFT makes no such assumption. The only requirements are:
 
1. More than 2/3 of the total HBAR stake is controlled by honest nodes
2. If a message is sent repeatedly between two nodes, eventually at least one will arrive
Under these minimal assumptions - and even if an attacker controls the network and can delay or drop messages - Hashgraph still reaches consensus. Every honest node eventually knows that consensus has been reached, with 100% certainty.
 
### The Coq Formal Proof
 
The aBFT property of Hashgraph is not just a claim - it is a **mathematical theorem proven by computer**.
 
The proof was verified using the **Coq proof assistant** - a formal verification system used in high-assurance software, operating systems, and mathematical research. A Coq proof is checked by the computer itself, eliminating the possibility of human error in reading a proof.
 
This proof was completed by a professor at Carnegie Mellon University, one of the world's leading computer science institutions.
 
**Key security threshold:** As long as malicious actors control less than 1/3 of the total HBAR stake, they cannot stop consensus or manipulate transaction ordering or timestamps. This is a mathematical guarantee, not an estimate.
 
## Fairness: A Mathematical Guarantee
 
Hashgraph provides three distinct forms of fairness that traditional blockchains cannot offer.
 
### 1. Fair Access
 
No individual node can prevent a transaction from entering the network, or even significantly delay it. If a malicious node refuses to forward a transaction, the random nature of the gossip protocol ensures the transaction reaches other nodes through different paths.
 
In blockchains, a miner can choose to simply ignore certain transactions, effectively censoring them for as long as they have mining power.
 
### 2. Fair Timestamps
 
Every transaction receives a consensus timestamp computed as the **median** of the timestamps assigned by all nodes that first received it. No single node can manipulate when a transaction "officially" happened.
 
In blockchains, the miner who creates a block has full control over that block's timestamp (within some limits), which can be used to manipulate time-sensitive applications.
 
### 3. Fair Ordering
 
Transactions are ordered by their consensus timestamps. No node has the ability to reorder transactions for profit.
 
In Ethereum, this vulnerability is known as **MEV (Maximal Extractable Value)** - miners and validators can reorder, insert, or censor transactions within a block to extract profit from users. This is impossible in Hashgraph.
 
## Cryptographic Security
 
All communication and data in Hedera is protected using cryptographic standards equivalent to those required for U.S. government Top Secret information:
 
| Component | Standard |
|-----------|----------|
| Network communication | TLS 1.2 |
| Transaction signing | Digital signatures |
| Graph integrity | Cryptographic hashes |
| Symmetric encryption | AES-256 |
| Asymmetric encryption | RSA 3072 |
| Hash function | SHA-384 |
| Elliptic curve | ECDSA and ECDH with P-384 |
| Key exchange | Ephemeral keys (perfect forward secrecy) |
 
These standards are specified by the **CNSA Suite (Commercial National Security Algorithm Suite)**, the U.S. government standard for protecting classified information.
 
## Performance and Throughput
 
Hashgraph's performance characteristics follow directly from its design:
 
**Bandwidth-limited throughput:** Hashgraph is limited only by the available network bandwidth of the nodes. If each node can handle N transactions per second of bandwidth, the network as a whole handles close to N TPS. There is no artificial throttling.
 
**100% efficiency:** No events are discarded. Every gossip sync contributes to consensus. No bandwidth is wasted on blocks that get thrown away.
 
**Zero voting overhead:** Virtual voting adds no additional messages beyond the gossip syncs already required to share transactions. The consensus mechanism runs "for free" on top of communication that would happen anyway.
 
**Near-instant finality:** Once a transaction achieves consensus timestamp (typically within 3-5 seconds on the Hedera network), it is final with 100% certainty. There are no confirmation periods, no block depth requirements, no probabilistic uncertainty.
 
**Current Hedera mainnet performance:**
- Throughput: 10,000+ TPS (native services)
- Finality: ~3-5 seconds
- Average fee: ~$0.0001 USD
- Energy: Carbon-negative footprint
 
## Summary Comparison Table
 
| Property | Traditional Blockchain | Hashgraph |
|----------|----------------------|-----------|
| Data structure | Linear chain of blocks | Directed Acyclic Graph (DAG) |
| Discarded blocks/events | Yes (stale blocks) | No - everything is preserved |
| Efficiency | Partial | 100% |
| Consensus mechanism | PoW / PoS with leaders or validators | Gossip + Virtual Voting (leaderless) |
| Security level | Partial BFT or probabilistic | aBFT (highest possible) |
| Formal proof | None | Coq-verified mathematical proof (CMU) |
| Finality | Probabilistic (grows with confirmations) | Absolute (100% in seconds) |
| Fairness | Not guaranteed | Mathematically guaranteed (3 types) |
| Transaction ordering manipulation | Possible (MEV) | Impossible |
| Typical speed | 7-30 TPS | 10,000+ TPS |
| Artificial slowdown required | Yes (PoW or equivalent) | No |
| Cryptographic standard | Varies | CNSA Suite (U.S. Top Secret grade) |
 
## Key Takeaways
 
- Hashgraph is **not a blockchain** - it is a DAG-based data structure with a fundamentally different consensus mechanism
- **Gossip about gossip** propagates information exponentially through the network while building a tamper-proof history of all communication
- **Virtual voting** allows every node to independently calculate consensus with zero additional messages, making the algorithm maximally bandwidth-efficient
- **aBFT** is the highest security grade achievable in distributed systems - Hashgraph is one of the only public networks to have formally proven this property using the Coq proof system
- **Fairness** (access, timestamps, and ordering) is mathematically guaranteed - not just a design goal
- The security guarantee holds as long as honest nodes control more than **2/3 of the total HBAR stake**
- All cryptography meets the **CNSA Suite** standard used for U.S. government Top Secret information
 
## References
 
| Resource | URL |
|----------|-----|
| What is Hashgraph Consensus? (Official) | https://hedera.com/learning/what-is-hashgraph-consensus |
| What is Gossip About Gossip? (Official) | https://hedera.com/learning/what-is-gossip-about-gossip |
| What is aBFT? (Official) | https://hedera.com/learning/what-is-asynchronous-byzantine-fault-tolerance-abft |
| What is Hedera? (Official) | https://hedera.com/learning/what-is-hedera-hashgraph |
| Hashgraph Technical Whitepaper (SWIRLDS-TR-2016-01) | https://hedera.com/wp-content/uploads/2025/11/SWIRLDS-TR-2016-01.pdf |
| Hashgraph Detailed Examples (SWIRLDS-TR-2016-02) | https://hedera.com/wp-content/uploads/2025/11/SWIRLDS-TR-2016-02.pdf |
| Hedera Main Whitepaper | https://hedera.com/wp-content/uploads/2025/12/hh_whitepaper.pdf |
| Hedera Knowledge Center (all papers) | https://hedera.com/knowledge-center |
 
---
 
*Previous: [Module 01 - Foundations & History](../module-01-foundations/README.md)*  
*Next: [Module 03 - HBAR Token](../module-03-hbar/README.md)*
 
---
 
*Last updated: June 2026*
*Part of the [Hedera Knowledge Base](https://github.com/nemorixgroup/hedera-knowledge-base)*
