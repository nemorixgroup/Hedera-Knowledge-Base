# Module 04 - Native Services
 
> Hedera exposes four native Layer 1 services - each with its own dedicated API - plus a read layer via the Mirror Node. This module covers HCS (Hedera Consensus Service), HTS (Hedera Token Service), HSCS (Hedera Smart Contract Service), HFS (Hedera File Service), and the Mirror Node REST API.
 
## Table of Contents
 
- [Services Overview](#services-overview)
- [HCS - Hedera Consensus Service](#hcs---hedera-consensus-service)
- [HTS - Hedera Token Service](#hts---hedera-token-service)
- [HSCS - Hedera Smart Contract Service](#hscs---hedera-smart-contract-service)
- [HFS - Hedera File Service](#hfs---hedera-file-service)
- [Mirror Node: The Read Layer](#mirror-node-the-read-layer)
- [Choosing the Right Service](#choosing-the-right-service)
- [Key Takeaways](#key-takeaways)
- [References](#references)
 
## Services Overview
 
Hedera provides composable services built directly into Layer 1. These are not smart contracts running on a generic protocol - they are native protocol services with dedicated APIs, optimized for their specific use cases.
 
```
┌─────────────────────────────────────────────────────────────────┐
│                      HEDERA NETWORK (L1)                        │
│                                                                 │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌──────────┐  │
│  │    HCS     │  │    HTS     │  │    HSCS    │  │   HFS    │  │
│  │ Consensus  │  │   Token    │  │   Smart    │  │   File   │  │
│  │  Service   │  │  Service   │  │  Contract  │  │ Service  │  │
│  │            │  │            │  │  Service   │  │          │  │
│  │ Timestamps │  │  Fungible  │  │  Solidity  │  │  Store   │  │
│  │ & ordering │  │  + NFTs    │  │  EVM (Besu)│  │  files   │  │
│  └────────────┘  └────────────┘  └────────────┘  └──────────┘  │
│                                                                 │
│               Mirror Node (read / query layer)                  │
│               REST API  |  gRPC  |  JSON-RPC Relay              │
└─────────────────────────────────────────────────────────────────┘
```
 
Hedera offers developers three primary services - Solidity-based smart contracts, consensus, and token services - through an easy-to-use API and officially supported or community-supported SDKs.
 
 
## HCS - Hedera Consensus Service
 
### What is HCS?
 
Hedera Consensus Service acts like a decentralized notary, providing verifiable and immutable timestamps for events without storing the data itself. This ensures no single party controls the process, eliminating front-running and guaranteeing that everyone sees the same transaction order, exactly as it happened.
 
**Key insight:** HCS does not store your data. It stores the cryptographic *proof* of the order and time at which that data existed. Any application that needs to answer "what happened first, and who saw it?" can use HCS as its source of truth.
 
### How HCS works internally
 
HCS applies the same three mechanisms from Module 02 (Hashgraph Algorithm) at the application level:
 
**Gossip about gossip:** Nodes share not just the message content, but also information about who they heard it from and when. This creates a trusted record of how data moves through the network.
 
**Virtual voting:** Nodes do not vote in real time. Instead, each one uses shared data to independently calculate the order of transactions. A consensus timestamp is then assigned based on when a supermajority of nodes first received the message.
 
**Fair ordering:** Transactions are timestamped based on when they are seen by the majority of the network, not by any single node.
 
### Core concepts: Topics and Messages
 
The fundamental unit of HCS is a **Topic** - a named channel for messages. Any application or user can submit messages to a topic, and each message receives:
- A **consensus timestamp** (immutable, agreed by the network)
- A **sequence number** (monotonically increasing, per topic)
- A **running hash** (cryptographic proof of message integrity and order)
```dart
// Create a topic
final txResponse = await TopicCreateTransaction()
    .setTopicMemo('NemorixPay transaction audit log')
    .execute(client);
 
final topicId = txResponse.getReceipt(client).topicId;
 
// Submit a message to the topic
await TopicMessageSubmitTransaction()
    .setTopicId(topicId)
    .setMessage('{"txId":"TX123","amount":100,"currency":"USD","status":"settled"}')
    .execute(client);
 
// The message now has:
// - consensus_timestamp: immutable, network-agreed
// - sequence_number: e.g. 1, 2, 3...
// - running_hash: cryptographic proof of full history
```
 
Messages can be public or encrypted with additional privacy layers. Every consensus result is verifiable and can be independently audited by anyone via the Mirror Node.
 
### HCS use cases
 
| Industry | Use case | What HCS provides |
|----------|----------|-------------------|
| Finance | Remittance audit trail | Immutable proof of settlement order |
| Supply chain | Product lifecycle tracking | Tamper-proof step timestamps |
| Healthcare | Patient data event log | Verifiable access history |
| Energy | Carbon emissions tracking | Immutable sensor data attestation |
| IoT | Machine-to-machine trust | Trusted timestamps for device events |
| Identity | DID operation log | Non-repudiable identity events |
 
**Real-world example:** Hyundai Motor Company and Kia Corporation launched an Integrated Greenhouse Gas Information System using HCS to monitor and record emissions across the full vehicle lifecycle, creating an auditable and immutable emissions record.
 
 
## HTS - Hedera Token Service
 
### What is HTS?
 
On most blockchains, tokens rely on smart contracts. That means higher fees, slower speeds, and added complexity. Hedera is built differently. Tokens are native, with dedicated APIs for fast, low-cost transfers and instant finality. When needed, you can still use EVM-compatible smart contracts for custom logic.
 
This is a fundamental architectural difference. On Ethereum, creating an ERC-20 token requires deploying a smart contract - code running on the EVM that consumes gas and can have bugs. On Hedera, tokens are first-class citizens of the protocol itself.
 
### Token types
 
HTS supports two native token types:
 
**Fungible Tokens**
- Every unit is identical and interchangeable
- Equivalent to ERC-20 on Ethereum
- Use cases: stablecoins, utility tokens, loyalty points, in-game currencies, CBDCs
**Non-Fungible Tokens (NFTs)**
- Each token is unique with its own metadata
- Equivalent to ERC-721 on Ethereum
- Use cases: digital art, certificates, credentials, tokenized real estate, event tickets
### Token lifecycle operations
 
| Operation | Description |
|-----------|-------------|
| `TokenCreateTransaction` | Create a new fungible or non-fungible token |
| `TokenMintTransaction` | Mint new supply (requires supply key) |
| `TokenBurnTransaction` | Burn tokens, reducing supply |
| `TransferTransaction` | Transfer tokens between accounts |
| `TokenAssociateTransaction` | Associate a token with an account before receiving it |
| `TokenDissociateTransaction` | Remove token association from an account |
| `TokenDeleteTransaction` | Permanently delete a token |
| `TokenUpdateTransaction` | Update token properties |
| `TokenPauseTransaction` | Pause all transfers of the token |
| `TokenGrantKycTransaction` | Grant KYC status to an account |
| `TokenRevokeKycTransaction` | Revoke KYC status from an account |
| `TokenFreezeTransaction` | Freeze an account from sending/receiving the token |
 
### Fine-grained key controls
 
HTS provides granular control over tokens through optional configurable keys:
 
| Key | Controls | Typical use |
|-----|----------|-------------|
| Admin Key | Modify token properties | General administration |
| KYC Key | Approve / revoke accounts to hold the token | Financial compliance |
| Freeze Key | Freeze / unfreeze specific accounts | Regulatory control |
| Wipe Key | Remove tokens from a specific account | Legal compliance |
| Supply Key | Mint / burn tokens | Supply management |
| Pause Key | Pause all token transfers | Emergency controls |
| Fee Schedule Key | Modify custom fee schedule | Economic model updates |
 
Keys are optional. A token with no KYC key cannot enforce KYC. A token with no pause key cannot be paused. This design gives token creators precise control over compliance features without building smart contract logic.
 
### Custom fees
 
Custom fees enable tokens on Hedera to be used for things like utility fees in applications that support their own fungible token, or perpetual royalties for NFTs. This functionality is built directly into the Hedera Services code (no smart contract configuration needed), making it easier to implement using the official SDKs.
 
Three fee types:
 
**Fixed Fee** - a fixed amount of HBAR or a specific token charged per transfer:
```dart
final fixedFee = CustomFixedFee()
    .setAmount(10)                    // 10 tinybars per transfer
    .setFeeCollectorAccountId(collectorId);
```
 
**Fractional Fee** - a percentage of the transferred amount:
```dart
final fractionalFee = CustomFractionalFee()
    .setNumerator(1)
    .setDenominator(100)              // 1% of transfer amount
    .setFeeCollectorAccountId(collectorId);
```
 
**Royalty Fee** - for NFTs, a percentage of the exchange value goes to the creator:
```dart
final royaltyFee = CustomRoyaltyFee()
    .setNumerator(5)
    .setDenominator(100)              // 5% royalty to creator
    .setFeeCollectorAccountId(creatorId);
```
 
### Creating a token (Dart example)
 
```dart
// Create a compliance-ready fungible token
final tokenCreateTx = await TokenCreateTransaction()
    .setTokenName('NemorixPay USD')
    .setTokenSymbol('NPUSD')
    .setTokenType(TokenType.fungibleCommon)
    .setDecimals(2)
    .setInitialSupply(1000000)        // 10,000.00 NPUSD
    .setTreasuryAccountId(treasuryId)
    .setAdminKey(adminPublicKey)
    .setKycKey(kycPublicKey)          // KYC enforcement
    .setFreezeKey(freezePublicKey)    // Regulatory freeze
    .setSupplyKey(supplyPublicKey)    // Mint/burn control
    .setPauseKey(pausePublicKey)      // Emergency pause
    .execute(client);
```
 
### HTS performance
 
Tokens on Hedera achieve 10,000 transactions per second, settle with finality immediately, and always cost $0.0001 USD, paid in HBAR, to transfer.
 
### EVM compatibility
 
HTS tokens can be mirrored into the Hedera EVM so they work exactly like ERC-20 tokens in Solidity smart contracts, DeFi platforms, and Ethereum tooling - without sacrificing native speed and efficiency.
 
 
## HSCS - Hedera Smart Contract Service
 
### What is HSCS?
 
Deploy and run smart contracts using the Ethereum Virtual Machine on the Hedera network. Use familiar tools, frameworks, and libraries such as Hardhat or MetaMask, while combining the flexibility of smart contracts with the speed and efficiency of native tokens through Hedera Token Service.
 
### The Hedera EVM: Besu optimized
 
The Hedera Smart Contract Service uses the **Besu EVM** - the same EVM used by Ethereum, specifically optimized for Hashgraph consensus. This optimization delivers:
 
- Hundreds of smart contract transactions per second
- Predictable fees fixed in USD (no gas price volatility)
- Carbon-negative footprint
- Performance at **15 million gas per second**
### Composability with native services
 
Hedera Smart Contract Service lets you build logic-driven workflows such as payments, escrow, and compliance checks. Smart contracts can interact directly with other native services, including Hedera Token Service and Hedera Consensus Service.
 
This means a single Solidity contract on Hedera can:
- Mint / burn HTS tokens natively
- Submit messages to HCS topics
- Implement complex DeFi logic (AMMs, lending protocols, DAOs)
- All in one atomic transaction
### Compatible tooling
 
Any Ethereum developer can deploy on Hedera immediately using existing tools:
 
| Tool | Status |
|------|--------|
| Solidity | Full support |
| Hardhat | Supported |
| Truffle | Supported |
| Foundry | Supported |
| MetaMask | Supported |
| ethers.js | Supported |
| web3.js | Supported |
| The Graph | Supported |
| OpenZeppelin | Supported |
| Remix IDE | Supported |
 
### Real-world deployments
 
SaucerSwap launched its DEX on Hedera leveraging HSCS for token swap logic - becoming the first decentralized exchange on the network. Bonzo Finance deployed a decentralized lending and borrowing protocol with collateralized loans and auto-balancing.
 
 
## HFS - Hedera File Service
 
### What is HFS?
 
HFS allows storing small files directly on the Hedera ledger. Unlike HCS (which only timestamps content), HFS stores the content itself persistently, with controlled read access.
 
### Core operations
 
```dart
// Create a file (up to 6 KB per transaction)
final fileCreateTx = await FileCreateTransaction()
    .setContents(utf8.encode('{"schema":"v1","endpoint":"https://..."}'))
    .setKeys([adminPublicKey])
    .setFileMemo('NemorixPay API config v1')
    .execute(client);
 
// Read a file
final contents = await FileContentsQuery()
    .setFileId(fileId)
    .execute(client);
 
// Append to a file (for content larger than 6 KB)
await FileAppendTransaction()
    .setFileId(fileId)
    .setContents(additionalBytes)
    .execute(client);
 
// Delete a file
await FileDeleteTransaction()
    .setFileId(fileId)
    .execute(client);
```
 
If a file is too large to create with a single FileCreateTransaction, it can be created with the first portion of its contents, then appended as many times as necessary to complete the file.
 
### HFS use cases
 
| Use case | Description |
|----------|-------------|
| Smart contract bytecode | Store compiled contract bytecode before deployment |
| Configuration / schemas | Network-accessible config files with verifiable history |
| Legal documents | Proof of document existence at a specific point in time |
| NFT metadata | Store metadata directly on-chain instead of IPFS |
| Address books | Publish node or service endpoint registries |
 
### Important limits
 
- Max content per `FileCreateTransaction`: **6 KB**
- For larger files: use `FileAppendTransaction` in chunks
- System files (network address book, fee schedules) occupy reserved ID ranges
 
## Mirror Node: The Read Layer
 
### What is the Mirror Node?
 
The Mirror Node is Hedera's historical data layer. Consensus nodes process and validate live transactions; Mirror Nodes receive, validate, and store the complete transaction history, then expose it via REST APIs and gRPC.
 
The mirror node software reduces the processing burden by receiving pre-constructed files from the network, validating them, populating a database, and providing REST APIs. Mirror nodes validate the signature files associated with record and event files from the consensus nodes.
 
### Official public endpoints
 
| Network | Base URL |
|---------|----------|
| Mainnet | `https://mainnet-public.mirrornode.hedera.com` |
| Testnet | `https://testnet.mirrornode.hedera.com` |
| Previewnet | `https://previewnet.mirrornode.hedera.com` |
 
Public mainnet mirror node requests are throttled at 100 requests per second (rps).
 
### Key REST API endpoints
 
```
# Accounts
GET /api/v1/accounts/{idOrAliasOrEvmAddress}
 
# Transactions
GET /api/v1/transactions
GET /api/v1/transactions/{transactionId}
 
# Tokens (HTS)
GET /api/v1/tokens
GET /api/v1/tokens/{tokenId}
GET /api/v1/tokens/{tokenId}/nfts/{serialNumber}
 
# Topics (HCS)
GET /api/v1/topics/{topicId}/messages
GET /api/v1/topics/{topicId}/messages/{sequenceNumber}
 
# Smart Contracts
GET /api/v1/contracts/{contractIdOrEvmAddress}
 
# Blocks
GET /api/v1/blocks
GET /api/v1/blocks/{hashOrNumber}
 
# Network
GET /api/v1/network/nodes
GET /api/v1/network/fees
GET /api/v1/network/supply
```
 
### Dart example: query HCS messages
 
```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
 
Future<void> queryHcsMessages(String topicId) async {
  final uri = Uri.parse(
    'https://testnet.mirrornode.hedera.com'
    '/api/v1/topics/$topicId/messages?order=asc&limit=10',
  );
 
  final response = await http.get(uri);
 
  if (response.statusCode == 200) {
    final data = jsonDecode(response.body) as Map<String, dynamic>;
    final messages = data['messages'] as List<dynamic>;
 
    for (final msg in messages) {
      final seqNum   = msg['sequence_number'];
      final ts       = msg['consensus_timestamp'];
      final content  = utf8.decode(base64.decode(msg['message'] as String));
      print('[$seqNum] $ts - $content');
    }
  }
}
```
 
### Running your own Mirror Node
 
For production applications requiring higher throughput or data sovereignty, you can run your own Mirror Node. Hedera provides official deployment guides for:
- Amazon Web Services (S3)
- Google Cloud Storage (GCS)
See: https://docs.hedera.com/hedera/core-concepts/mirror-nodes
 
 
## Choosing the Right Service
 
| Need | Service | Why |
|------|---------|-----|
| Create fungible tokens (stablecoins, loyalty) | HTS | Native, fast, cheapest |
| Create NFTs | HTS | Native NFT support |
| Token with complex DeFi logic | HSCS + HTS | Composability + native speed |
| Immutable audit log / timestamping | HCS | No data stored, just proof of order |
| Decentralized message ordering | HCS | Fair, verifiable ordering |
| DEX, lending, DAO | HSCS | Needs EVM composability |
| Store config / schemas on-chain | HFS | Persistent file storage |
| Smart contract bytecode storage | HFS | Required before HSCS deployment |
| Query historical transaction data | Mirror Node | REST API, no fees |
| Real-time transaction monitoring | Mirror Node gRPC | Subscribe to topic messages |

 
## Key Takeaways
 
- **HCS** is a decentralized notary - it timestamps and orders messages without storing their content, providing fair and immutable proof of event sequence
- **HTS** offers native token creation (fungible + NFT) without smart contracts - faster, cheaper, and with granular compliance controls (KYC, freeze, pause, wipe, custom fees)
- **HSCS** runs Solidity on a Besu EVM optimized for Hashgraph - fully compatible with all Ethereum tooling, and composable with HTS and HCS natively
- **HFS** stores files (up to 6 KB per transaction, appendable) directly on the ledger - used for smart contract bytecode, schemas, and immutable document proof
- **Mirror Node** is the read layer - free REST API access to the complete transaction history with 100 rps public rate limit, plus self-hosted options
- All services share the same properties: **~$0.0001 USD fees**, **3-5 second finality**, **10,000+ TPS**, and **no gas volatility**
 
## References
 
| Resource | URL |
|----------|-----|
| Hedera Consensus Service (Official) | https://hedera.com/service/consensus-service |
| Hedera Token Service (Official) | https://hedera.com/service/token-service |
| Hedera Smart Contract Service (Official) | https://hedera.com/service/smart-contract-service |
| Services & Products Overview | https://hedera.com/services |
| HCS SDK Docs | https://docs.hedera.com/hedera/sdks-and-apis/sdks/consensus-service |
| HTS SDK Docs | https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service |
| HSCS SDK Docs | https://docs.hedera.com/hedera/sdks-and-apis/sdks/smart-contracts |
| HFS SDK Docs | https://docs.hedera.com/hedera/sdks-and-apis/sdks/file-service |
| Mirror Node Concepts | https://docs.hedera.com/hedera/core-concepts/mirror-nodes |
| Mirror Node REST API | https://docs.hedera.com/guides/docs/mirror-node-api/rest-api |
| Mainnet Mirror Node | https://mainnet-public.mirrornode.hedera.com |
| Testnet Mirror Node | https://testnet.mirrornode.hedera.com |
| Fee Calculator | https://hedera.com/fee-calculator |
 
---
 
*Previous: [Module 03 - HBAR Token](https://github.com/nemorixgroup/Hedera-Knowledge-Base/tree/main/module-03-hbar)*  
*Next: [Module 05 - Development on Hedera](https://github.com/nemorixgroup/Hedera-Knowledge-Base/tree/main/module-05-development)*
 
---
 
*Last updated: June 2026*  
*Part of the [Hedera Knowledge Base](https://github.com/nemorixgroup/Hedera-Knowledge-Base)*
