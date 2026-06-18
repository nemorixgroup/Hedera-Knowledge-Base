# Hedera Knowledge Base
 
> A comprehensive, in-depth technical guide to the Hedera network - covering its architecture, consensus algorithm, native services, token economics, development ecosystem, and real-world use cases.
 
## About This Repository
 
This repository is a structured knowledge base built for developers, researchers, and anyone who wants to understand Hedera at a deep technical level - not just at a surface level.
 
The content is organized in progressive modules, starting from foundational concepts and advancing toward hands-on development. Each module is self-contained and can be studied independently, but reading them in order is recommended for a complete understanding.
 
**Who is this for?**
 
- Developers exploring Hedera for the first time
- Engineers evaluating Hedera for enterprise or startup projects
- Blockchain enthusiasts who want to understand how Hashgraph differs from traditional blockchains
- Flutter/Dart developers interested in building on Hedera (see Module 05)
**Languages:** English (primary). Some modules may include Spanish summaries.
 
## Repository Structure
 
```
hedera-knowledge-base/
- README.md                        <- You are here (General Index)
- module-01-foundations/
    - README.md                    <- History, origin, Governing Council
- module-02-hashgraph/
    - README.md                    <- DAG, gossip protocol, virtual voting
- module-03-hbar/
    - README.md                    <- Tokenomics, supply, staking, fees
- module-04-services/
    - README.md                    <- HCS, HTS, HSCS, HFS, HAPI
- module-05-development/
    - README.md                    <- SDKs, transactions, accounts, Flutter
- module-06-ecosystem/
    - README.md                    <- Use cases, projects, comparisons, future
```
 
## Modules
 
| # | Module | Topics |
|---|--------|--------|
| 01 | [Foundations & History](./module-01-foundations/README.md) | Origin, Leemon Baird, Governing Council, mission |
| 02 | [The Hashgraph Algorithm](./module-02-hashgraph/README.md) | DAG vs blockchain, gossip about gossip, virtual voting, BFT |
| 03 | [HBAR Token](./module-03-hbar/README.md) | Tokenomics, supply, fees, staking, security role |
| 04 | [Native Services](./module-04-services/README.md) | HCS, HTS, HSCS, HFS, Mirror Node, HAPI |
| 05 | [Development on Hedera](./module-05-development/README.md) | SDKs, accounts, transactions, Flutter integration |
| 06 | [Ecosystem & Future](./module-06-ecosystem/README.md) | Use cases, enterprise adoption, comparisons, roadmap |
 
## Official Resources
 
### Core
 
| Resource | URL |
|----------|-----|
| Official Website | https://hedera.com |
| Developer Documentation | https://docs.hedera.com |
| Hedera Papers (Whitepapers) | https://hedera.com/papers |
| Developer Tooling Overview | https://hedera.com/developer-tooling |
| Start Building Guide | https://hedera.com/start-building |
 
### Developer Tools
 
| Resource | URL |
|----------|-----|
| Developer Portal (Testnet accounts) | https://portal.hedera.com |
| Hedera Faucet | https://faucet.hedera.com |
| Developer Playground | https://docs.hedera.com/hedera/tutorials/developer-playground |
| HashScan (Network Explorer) | https://hashscan.io |
| Mirror Node REST API | https://mainnet-public.mirrornode.hedera.com |
 
### SDKs (Official)
 
| Language | Repository |
|----------|------------|
| JavaScript / TypeScript | https://github.com/hashgraph/hedera-sdk-js |
| Java | https://github.com/hashgraph/hedera-sdk-java |
| Go | https://github.com/hashgraph/hedera-sdk-go |
| Swift | https://github.com/hashgraph/hedera-sdk-swift |
| Rust | https://github.com/hashgraph/hedera-sdk-rust |
| C++ | https://github.com/hashgraph/hedera-sdk-cpp |
| **Flutter / Dart** | https://github.com/nemorixpay/hedera-flutter-sdk *(community)* |
 
### GitHub Organizations
 
| Organization | URL |
|--------------|-----|
| Hashgraph (core) | https://github.com/hashgraph |
| Hiero (Linux Foundation DLT) | https://github.com/hiero-ledger |
| Awesome Hedera | https://github.com/hashgraph/awesome-hedera |
| Official Docs Repo | https://github.com/hashgraph/hedera-docs |
 
### Community & News
 
| Resource | URL |
|----------|-----|
| Hedera Blog | https://hedera.com/blog |
| Hedera Discord | https://hedera.com/discord |
| Hedera Twitter / X | https://twitter.com/hedera |
| Hedera Reddit | https://reddit.com/r/hashgraph |
| The Hashgraph Association | https://hashgraph.swiss |
 
## Key Technical Facts (Quick Reference)
 
| Property | Value |
|----------|-------|
| Consensus Algorithm | Hashgraph (asynchronous BFT) |
| Transaction Speed | Up to 10,000+ TPS |
| Finality | ~3-5 seconds (absolute finality) |
| Average Transaction Fee | ~$0.0001 USD |
| Native Token | HBAR |
| Total Supply | 50,000,000,000 HBAR (fixed) |
| EVM Compatible | Yes (via JSON-RPC Relay) |
| Open Source | Yes (Apache 2.0 via Hiero project) |
| Governance | Hedera Governing Council (39 term-limited organizations) |
 
## Why Hedera?
 
Traditional blockchains (like Bitcoin or Ethereum) chain blocks of transactions in a linear sequence. Hedera uses a fundamentally different data structure called a **Directed Acyclic Graph (DAG)** combined with the **Hashgraph consensus algorithm** - invented by Dr. Leemon Baird. This enables:
 
- **Speed** - Thousands of transactions per second, settled in seconds
- **Fairness** - Mathematically proven fair ordering of transactions
- **Security** - Asynchronous Byzantine Fault Tolerant (aBFT) - the highest security grade achievable in distributed systems
- **Low cost** - Fixed, predictable fees as low as $0.0001 per transaction
- **Enterprise grade** - Governed by a council of leading global organizations including Google, IBM, Boeing, LG, and others

This is not just another blockchain. Hedera is a **public distributed ledger** that surpasses blockchain technology in key dimensions - which is why some engineers describe it as a next-generation evolution beyond traditional blockchains.
 
---
 
## About This Guide
 
This knowledge base was built through deep research, official Hedera documentation, published whitepapers, and hands-on development experience - including the construction of [`hedera-flutter-sdk`](https://github.com/nemorixpay/hedera-flutter-sdk), the first native Flutter/Dart SDK for the Hedera network.
 
The goal is to provide the most clear, honest, and technically accurate resource available for understanding Hedera - from the inside out.

## Support This Project

If this project is useful to you or your team, consider supporting its development. Every contribution helps cover infrastructure, documentation, and the time invested in building and maintaining this
open source resource for the Hedera blockchain community. Thank you!

[![Buy Me a Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-Support-FFDD00?logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/nemorixgroupllc)
[![Sponsor](https://img.shields.io/badge/Sponsor-GitHub-EA4AAA?logo=github-sponsors&logoColor=white)](https://github.com/sponsors/nemorixgroup)
[![Ko-fi](https://img.shields.io/badge/Ko--fi-Support-FF5E5B?logo=ko-fi&logoColor=white)](https://ko-fi.com/nemorixgroupllc)
 
---
 
*Last updated: June 2026*
*Maintained by: [Miguel Fagundez](https://github.com/miguelfagundez) & [Nemorix Group, LLC](https://github.com/nemorixpay)*
