# Module 01 - Foundations & History
 
> Understanding where Hedera comes from, who built it, why it was created, and how it evolved from a research idea into one of the most enterprise-ready distributed ledger networks in the world.
 
## Table of Contents
 
- [The Problem Hedera Was Built to Solve](#the-problem-hedera-was-built-to-solve)
- [The Founders](#the-founders)
- [Chronological Timeline](#chronological-timeline)
- [The Governing Council](#the-governing-council)
- [Mission and Vision](#mission-and-vision)
- [Key Takeaways](#key-takeaways)
- [References](#references)
---
 
## The Problem Hedera Was Built to Solve
 
Traditional blockchains like Bitcoin and Ethereum introduced a revolutionary idea - decentralized, trustless systems where no single party controls the data. However, they came with serious limitations that made them impractical for real-world enterprise adoption:
 
| Problem | Traditional Blockchains | Hedera's Answer |
|---------|------------------------|-----------------|
| **Speed** | Bitcoin ~7 TPS, Ethereum ~15-30 TPS | 10,000+ TPS |
| **Cost** | Variable, unpredictable gas fees | Fixed ~$0.0001 per transaction |
| **Finality** | Probabilistic - transactions can be reversed via forks | Absolute finality in ~3-5 seconds |
| **Fairness** | Miners/validators can reorder transactions for profit | Mathematically proven fair ordering |
| **Energy** | Proof of Work consumes massive electricity | Proof of Stake - minimal energy footprint |
| **Governance** | No formal structure - anyone can fork the network | Formal Governing Council with legal accountability |
 
These limitations were not minor inconveniences. They were fundamental architectural barriers preventing blockchains from being used in healthcare, finance, supply chains, and government systems at scale.
 
Hedera was designed from the ground up to solve all of them simultaneously.
 
## The Founders
 
### Dr. Leemon Baird - The Scientist
 
Dr. Leemon Baird is the inventor of the Hashgraph algorithm and co-founder of Hedera. His background spans computer science, artificial intelligence, and cybersecurity, including work at Wright Laboratory for the U.S. Air Force.
 
Key milestones:
- **2015** - Invented the Hashgraph consensus algorithm
- **2016** - Published the Hashgraph whitepaper (May 31, 2016)
- **2017** - Co-founded Hedera Hashgraph alongside Mance Harmon
- **2024** - Led the open-sourcing of the codebase via Project Hiero
His core insight was that it was possible to build a distributed ledger that was simultaneously fast, secure, fair, and mathematically provable - something no blockchain had achieved before.
 
### Mance Harmon - The Visionary
 
Mance Harmon is the business co-founder of Hedera. His background includes cybersecurity and missile threat simulation for the U.S. Air Force, followed by executive leadership at multiple technology companies.
 
Key contributions:
- Designed the Governing Council governance model - inspired by VISA's original 1968 structure
- Led the formation of the Hedera Governing Council
- Oversaw Hedera's growth from a research project to an enterprise-grade global network
- **July 2025** - Elected Chairman of Hedera Council for a two-year term
Both founders are U.S. Air Force veterans. Their military background heavily influenced Hedera's emphasis on security, reliability, and long-term thinking.
 
## Chronological Timeline
 
### 2015 - The Origin
 
Dr. Leemon Baird develops the Hashgraph algorithm. The core idea: instead of chaining blocks in a linear sequence (as blockchains do), use a Directed Acyclic Graph (DAG) where every node in the network gossips information to random peers, and those gossip events themselves become the data structure used to reach consensus.
 
Shortly after, Baird and Mance Harmon form **Swirlds, Inc.** to battle-test the algorithm for enterprise use.
 
### 2016 - The Whitepaper
 
The Hashgraph whitepaper is published on **May 31, 2016**, introducing the world to:
- Gossip about gossip protocol
- Virtual voting consensus
- Asynchronous Byzantine Fault Tolerance (aBFT)
### 2017 - Hedera Is Born
 
Hedera Hashgraph, LLC is formed as a Delaware limited liability company by Swirlds, Inc. The vision: a public distributed ledger powered by the Hashgraph algorithm, governed by a council of leading global organizations.
 
Hedera raises **$124 million USD** in 2018 via Simple Agreements for Future Tokens (SAFTs).
 
### 2018 - Mainnet Goes Live
 
- **August 24, 2018** - Hedera mainnet goes live in private beta
- The total supply of **50 billion HBARs** is minted
- The inaugural Hedera Governing Council is formed
- The first five council members are announced: Deutsche Telekom, DLA Piper, Magazine Luiza, Nomura Holdings, and Swisscom Blockchain
- Early access programs begin for developers
### 2019 - Public Access
 
- **September 16, 2019** - Hedera mainnet opens to the public - any developer can now create an account and build applications
- The Mirror Node software enters alpha, providing a way to query historical ledger data
- HBAR token distribution begins for SAFT participants
- Additional council members join: Google, Wipro, Avery Dennison, UCL, LG Electronics, Zain Group, Boeing, IBM, and Tata Communications
### 2020 - Services Launch
 
- **February 13, 2020** - **Hedera Consensus Service (HCS)** launches on mainnet - enables any application to use Hedera as a decentralized message bus
- Hedera surpasses Ethereum in daily transaction count
- **August 6, 2020** - Network services are open-sourced under Apache 2.0 license
- **December 7, 2020** - **Hedera Token Service (HTS)** launches on testnet
### 2021 - Ecosystem Growth
 
- HTS launches on mainnet - enabling creation of fungible and non-fungible tokens natively
- The HBAR Foundation is formed to fund ecosystem development
- Hedera processes its **1 billionth transaction**
- Additional enterprise council members join across multiple industries
### 2022 - EVM Compatibility and Open Source
 
- **February 2022** - Smart contracts upgraded to full EVM compatibility (Solidity support)
- **July 21, 2022** - Native HBAR staking and delegation launched on mainnet
- **2022** - The Governing Council votes to purchase the Hashgraph patent rights from Swirlds, Inc. and open-sources the algorithm under Apache 2.0
This was a pivotal moment: the Hashgraph algorithm - previously patented and exclusively licensed - became fully open source. A frequent criticism of Hedera was resolved permanently.
 
### 2023 - EVM Tooling
 
- **August 16, 2023** - Mainnet support for third-party EVM tooling launches:
  - JSON-RPC access via third-party providers
  - Smart contract deployment via Truffle, Hardhat, Foundry, EthersJS, Web3JS
  - Data indexing via The Graph
- Hedera becomes accessible to any Ethereum developer without learning new tools
### 2024 - Project Hiero
 
- **September 14, 2024** - Hedera contributes its complete codebase to the **Linux Foundation Decentralized Trust (LFDT)** as **Project Hiero**
- Hedera becomes the **first Layer 1 public network** to contribute its code to a neutral foundation
- This eliminates the last remaining concern about vendor control over the codebase
### 2025 - Hiero Graduates
 
- **February 2025** - The Hedera mainnet begins running entirely on Hiero's open codebase
- Hiero achieves **graduated status** within LFDT - the highest level of technical maturity, governance standards, and community participation
- **July 2025** - Mance Harmon elected Chairman of Hedera Council
- Council grows to 31 members across 11 industries
- Cross-border payment costs reduced from $40 to $0.10; settlement time from one week to under one minute
### 2026 - Enterprise Adoption Accelerates
 
- NVIDIA joins via the HEAT program for AI data provenance
- McLaren Racing joins as a full Council member
- Wyoming's Frontier Stable Token (FRNT) - the first U.S. state-issued stable token - launches on Hedera
- USDT0 integrates for cross-chain stablecoin liquidity
- Canary Capital's spot HBAR ETF launches on Nasdaq
- The SEC and CFTC classify HBAR as a digital commodity
- The network has processed over $10 billion in real-world asset settlements
 
## The Governing Council
 
The Hedera Governing Council is one of the most distinctive aspects of the entire network - and one of the reasons Hedera is considered enterprise-grade.
 
### What Is It?
 
The Council is a formal body of up to **39 global organizations** responsible for:
- Operating the consensus nodes on the network
- Managing the HBAR treasury
- Voting on network upgrades, fees, and strategic direction
- Ensuring the network remains stable, secure, and resistant to forks
### How It Works
 
The model is inspired by **VISA's original 1968 governance structure** - a consortium of competing organizations that cooperate on shared infrastructure.
 
Key rules:
- Each member gets **equal voting rights** regardless of size or industry
- Members serve **staggered 3-year terms**
- No more than **two consecutive terms** per member
- Former members can rejoin after a 3-year waiting period
- **Swirlds, Inc.** (the founding company) holds a permanent seat
- Target: up to **39 members** across diverse industries and geographies
### Current Members (2026, selection)
 
**Technology:** Google, IBM, Dell, LG Electronics, Deutsche Telekom, ServiceNow, NVIDIA
 
**Finance:** Standard Bank, DBS Bank, Shinhan Bank, Nomura Holdings, FIS (Worldpay)
 
**Enterprise:** Boeing, FedEx, Tata Communications, Arrow Electronics
 
**Energy:** EDF, Blockchain for Energy
 
**Gaming / Entertainment:** Ubisoft, McLaren Racing
 
**Academic:** London School of Economics (LSE), University College London (UCL)
 
**Legal / Professional:** DLA Piper
 
**Web3 / Blockchain:** Chainlink Labs, Swirlds
 
### Why This Matters for Developers
 
When you build on Hedera, the nodes processing your transactions are operated by Google, IBM, Boeing, and similar organizations - not anonymous validators. This has real implications:
 
- **No validator manipulation** - known, accountable entities run the nodes
- **No rogue forks** - the Council controls what constitutes the official network
- **Legal accountability** - Council members sign a formal LLC Agreement
- **Enterprise trust** - regulated industries can use Hedera without worrying about who controls the infrastructure
 
## Mission and Vision
 
Hedera's stated mission is to be the **"trust layer of the internet"** - a public, neutral, and permanent infrastructure layer that any application, company, or government can build on.
 
The founders framed this as a **100-year project**. Not a startup looking for an exit. Not a protocol chasing token price. A long-term infrastructure play designed to outlast any individual organization.
 
Three core principles guide everything Hedera builds:
 
**Performance** - The network must be fast enough and cheap enough for real-world applications. Not "crypto fast" - actually fast. 10,000+ TPS, $0.0001 fees, 3-5 second finality.
 
**Security** - Asynchronous Byzantine Fault Tolerance (aBFT) is the highest security grade achievable in distributed systems. Hedera is one of only a handful of networks to formally prove this property.
 
**Decentralization** - Governance by a global council of independent organizations, with open-source code under neutral Linux Foundation stewardship, and a clear roadmap toward permissionless node operation.
 
## Key Takeaways
 
- Hedera was founded in **2015-2017** by Dr. Leemon Baird (inventor of Hashgraph) and Mance Harmon
- It is **not a blockchain** - it uses a Directed Acyclic Graph (DAG) with the Hashgraph consensus algorithm
- The mainnet launched in **2018 (private)** and **2019 (public)**
- It is governed by a **Council of up to 39 global organizations** including Google, IBM, Boeing, and Deutsche Telekom
- The algorithm is fully **open source** (Apache 2.0) since 2022, and the entire codebase is managed by the **Linux Foundation** as Project Hiero since 2024
- The network processes **10,000+ TPS** with **absolute finality** in 3-5 seconds at **~$0.0001** per transaction
- As of 2026, it has processed **$10B+ in real-world asset settlements** and hosts enterprise applications across supply chain, payments, identity, and AI
 
## References
 
| Resource | URL |
|----------|-----|
| Hedera Journey (Official Timeline) | https://hedera.com/journey |
| Hedera Whitepapers | https://hedera.com/papers |
| Hedera Council FAQ | https://hederacouncil.org/faq |
| Hedera Council About | https://hederacouncil.org/about |
| Project Hiero - Open Source | https://hedera.com/hiero-open-source |
| Hiero on GitHub | https://github.com/hiero-ledger |
| Hedera Wikipedia | https://en.wikipedia.org/wiki/Hedera_(distributed_ledger) |
 
---
 
*Next: [Module 02 - The Hashgraph Algorithm](https://github.com/nemorixgroup/hedera-knowledge-base/tree/main/module-02-hashgraph)*
 
---
 
*Last updated: June 2026*
*Part of the [Hedera Knowledge Base](https://github.com/nemorixgroup/hedera-knowledge-base/)*
