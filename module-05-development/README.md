# Module 05 - Development on Hedera
 
> A developer-focused guide to building on the Hedera network. Covers the two development paths (native SDKs and EVM), official SDK languages, account model, key types, transaction lifecycle, advanced features like batch and scheduled transactions, and a dedicated section on the Flutter/Dart SDK.
 
## Table of Contents
 
- [Two Paths to Build on Hedera](#two-paths-to-build-on-hedera)
- [Official SDKs (Hiero)](#official-sdks-hiero)
- [Accounts: The Foundation](#accounts-the-foundation)
- [Account ID Format](#account-id-format)
- [Account Properties](#account-properties)
- [Keys and Signatures: ECDSA vs Ed25519](#keys-and-signatures-ecdsa-vs-ed25519)
- [Key Structures: Simple, KeyList, Threshold](#key-structures-simple-keylist-threshold)
- [Transaction Lifecycle](#transaction-lifecycle)
- [Transaction ID](#transaction-id)
- [Transaction Properties and Limits](#transaction-properties-and-limits)
- [Batch Transactions](#batch-transactions)
- [Scheduled Transactions](#scheduled-transactions)
- [Developer Portal and Testnet Setup](#developer-portal-and-testnet-setup)
- [Flutter / Dart SDK](#flutter--dart-sdk)
- [Key Takeaways](#key-takeaways)
- [References](#references)
 
## Two Paths to Build on Hedera
 
Hedera supports two distinct development approaches, each suited for different use cases:
 
```
Path 1: Hedera Native SDKs
  - Languages: Java, JavaScript, Go, Swift, Rust, C++, Dart/Flutter
  - Direct access to all native services: HCS, HTS, HFS, HSCS
  - No Solidity or EVM knowledge required
  - Best for: fintech apps, enterprise integrations, IoT, mobile
 
Path 2: EVM (Ethereum-compatible)
  - Languages: Solidity
  - Tools: Hardhat, Foundry, Remix, ethers.js, MetaMask
  - Same tooling as Ethereum, running on Hedera's Besu EVM
  - Best for: DeFi protocols, DAOs, NFT marketplaces, migrating from Ethereum
```
 
Both paths can be combined. A native SDK can call smart contracts. A Solidity contract can interact with HTS tokens and HCS topics natively.
 
Developers choose Hedera for its fast, final, and fair transactions (no MEV), predictable fees, stable governance, negative carbon footprint, and its thriving ecosystem.
 
## Official SDKs (Hiero)
 
Hedera and the developer community contribute to and maintain the Hiero Consensus Node SDKs across various languages. All SDKs implement the Hedera APIs in a consistent way, so switching between languages is straightforward.
 
| Language | Repository | Status |
|----------|------------|--------|
| JavaScript / TypeScript | `hiero-ledger/hiero-sdk-js` | Official |
| Java | `hiero-ledger/hiero-sdk-java` | Official |
| Go | `hiero-ledger/hiero-sdk-go` | Official |
| Swift | `hiero-ledger/hiero-sdk-swift` | Official |
| Rust | `hiero-ledger/hiero-sdk-rust` | Official |
| C++ | `hiero-ledger/hiero-sdk-cpp` | Official |
| Flutter / Dart | `nemorixpay/hedera-flutter-sdk` | Community / Nemorix Group, LLC |
 
**React Native note:** The Hiero JavaScript SDK supports React Native with Expo on Android devices and Android emulators. It does not currently support React Native Bare.
 
**Swift 6 migration (2026):** The Hiero Swift SDK officially supports Swift 6.0, 6.1, and 6.2 as of version v0.47.0 (February 2, 2026). Developers on Swift 5.x have until August 3, 2026 to migrate, after which Swift 5.x will no longer be supported.
 
**Community SDKs** are maintained by the Hedera developer community and include support for languages and frameworks not covered by the official SDK set. These are listed in the Hedera developer tooling page and licensed under Apache 2.0.
 

## Accounts: The Foundation
 
Accounts are the central starting point when interacting with the Hedera network. A Hedera account is an entity stored in the ledger that holds tokens. Accounts can hold:
 
- **HBAR** - the native Hedera token used to pay transaction fees
- **Fungible Tokens** - custom tokens created via HTS
- **Non-Fungible Tokens (NFTs)** - unique tokens created via HTS
You interact with the network by submitting transactions that modify state (write operations) or queries that read state. Most transactions and queries carry a fee charged in HBAR.
 
**Account creation** requires an existing funded account to pay the creation fee. For development:
- Use the Hedera Developer Portal to create free testnet accounts
- Use Auto Account Creation for programmatic account generation in apps
- Use supported wallets for mainnet account creation

 
## Account ID Format
 
Every Hedera entity (accounts, tokens, topics, contracts, files) is identified by a three-part ID:
 
```
<shardNum>.<realmNum>.<entityNum>
 
Examples:
  0.0.123       - Account 123 on mainnet (shard 0, realm 0)
  0.0.100       - Token ID 100
  0.0.54321     - Topic ID 54321
  0.100.456     - Account in realm 100 (private networks)
```
 
Each realm maintains a single counter for all entity types. This means if file `0.1.2` exists, no account or smart contract can have ID `0.1.2` in the same realm. Entity numbers are unique, immutable, and assigned by the network upon creation.
 
### Account aliases
 
In addition to the numeric ID, accounts can have an **alias** - an alternate identifier used in specific transaction types:
 
| Alias type | Format | Description |
|------------|--------|-------------|
| Account Number Alias | `0x000...0a` | Hex-encoded account number prefixed with 20 zero bytes |
| Public Key Alias | `0.0.<pubkeyHex>` | The ECDSA or Ed25519 public key bytes |
| EVM Address Alias | `0.0.<evmAddress>` | Rightmost 20 bytes of Keccak-256(ECDSA pubkey) |
 
The EVM address alias is the most important for modern development: it enables compatibility with MetaMask, Solidity's `msg.sender`, and all EVM-native tooling.
 
 
## Account Properties
 
Key properties every developer should understand:
 
| Property | Description | Notes |
|----------|-------------|-------|
| Account ID | `shard.realm.num` format | Immutable |
| Account Alias | EVM address or public key alias | Immutable once set |
| Key | ECDSA or Ed25519 (or key structure) | Required; mutable |
| HBAR Balance | Current HBAR balance in tinybars | Mutable |
| Token Balances | List of associated tokens and balances | Mutable |
| Max Auto Associations | Max tokens auto-associated without manual approval | -1 = unlimited |
| Staked Node ID | Node this account's HBAR is staked to | Mutable |
| Memo | Up to 100 characters, visible on explorers | Mutable; never store private data |
| EVM Nonce | Count of EthereumTransaction submissions | Auto-managed |
| Auto Renew Period | Interval for renewal fees (30 days - 3 months) | Not yet enabled |
 
**Automatic Token Associations:** Accounts can automatically receive up to 5,000 tokens without manually preauthorizing each one. Setting `maxAutoAssociations = -1` allows unlimited automatic associations (default for accounts created via auto account creation). This is enabled as part of frictionless airdrops (HIP-904).
 
 
## Keys and Signatures: ECDSA vs Ed25519
 
Key selection is one of the most important decisions in Hedera development. The official documentation is explicit:
 
**ECDSA secp256k1 is the recommended key type for all new accounts and applications on Hedera.**
 
### ECDSA secp256k1 (Recommended)
 
ECDSA is the same cryptographic curve used by Ethereum. This means:
 
- Full compatibility with MetaMask, Hardhat, Foundry, ethers.js
- Full `msg.sender` resolution in Solidity smart contracts
- Native support for `ECRECOVER` precompile
- EVM address (alias) derived from the public key
The EVM alias is the rightmost 20 bytes of the 32-byte Keccak-256 hash of the ECDSA public key, as described in the Ethereum Yellow Paper. The recovery ID is not included in the hash.
 
```javascript
// JavaScript SDK example (official)
const ecdsaPrivateKey = PrivateKey.generateECDSA();
const ecdsaPublicKey  = ecdsaPrivateKey.publicKey;
 
// Set EVM alias at account creation (recommended)
const txCreateAccount = new AccountCreateTransaction()
    .setKeyWithAlias(ecdsaPublicKey)
    .setInitialBalance(new Hbar(1));
```
 
**Critical warning:** Do not use `setKeyWithAlias()` if you plan to rotate the account key in the future. The EVM alias is permanently bound to the original ECDSA public key and does not update when keys are rotated. Use `setKeyWithoutAlias()` for accounts that will undergo key rotation.
 
### Ed25519 (Supported, not recommended for new development)
 
Ed25519 is supported but does not produce an EVM address. It cannot interact directly with Solidity smart contracts via `msg.sender` or `ECRECOVER`. It can still interact with contracts via HIP-632 system contract functions (`isAuthorized` / `isAuthorizedRaw`).
 
Ed25519 is appropriate for HCS-only applications and Hedera-native workflows where EVM compatibility is not needed.
 
### Comparison table
 
| Property | ECDSA secp256k1 | Ed25519 |
|----------|-----------------|---------|
| Recommendation | Recommended for all new development | Supported only |
| EVM Address | Yes - set via `setKeyWithAlias()` | No |
| Smart contract compatibility | Full - `msg.sender`, `ECRECOVER`, Solidity | Limited - HIP-632 only |
| Wallet support | MetaMask + all EVM wallets + HashPack | HashPack and Hedera-native only |
| Key rotation | Supported (avoid EVM alias if rotation needed) | Supported |
| Use cases | dApps, DeFi, payments, EVM integrations | HCS, Hedera-native only |
 
 
## Key Structures: Simple, KeyList, Threshold
 
Hedera supports three key structures, far more expressive than Ethereum's single-key model:
 
### Simple Key
A single key controls the account. One signature required.
 
```
Account Key = { Key1 }
Required signatures: Key1
```
 
### Key List
All keys in the list must sign. N-of-N multisig.
 
```
Account Key = KeyList { Key1, Key2, Key3 }
Required signatures: Key1 AND Key2 AND Key3
```
 
### Threshold Key
A minimum number (threshold) of keys must sign. M-of-N multisig.
 
```
Account Key = ThresholdKey(2/3) { Key1, Key2, Key3 }
Required signatures: any 2 of { Key1, Key2, Key3 }
```
 
Key structures can be **nested** - a ThresholdKey can contain KeyLists, and vice versa. This enables complex enterprise governance models, such as requiring board approval for large transactions.
 
All transaction types that include a key field support these three structures.
 
 
## Transaction Lifecycle
 
Every interaction with the Hedera network follows this lifecycle:
 
```
Step 1 - CREATE
  Client builds the transaction object
  Sets: parameters, maxFee, validDuration, memo
 
Step 2 - SIGN
  The fee-paying account signs (mandatory)
  Additional signatures per transaction type requirements
 
Step 3 - SUBMIT
  Transaction sent to a specific node via gRPC
  Node validates the payer's signature
 
Step 4 - CONSENSUS
  Node gossips the transaction to the network
  Hashgraph algorithm assigns a consensus timestamp
  Valid duration checked: if expired, rejected
 
Step 5 - EXECUTE
  Transaction applied to ledger state
  Receipt generated (success / failure status)
  Record generated (full details, logs, changes)
 
Step 6 - QUERY
  Client requests the receipt (free) or record (small fee)
  Confirms finality - 3-5 seconds after submission
```
 
The lifecycle begins when a client creates a transaction. Once created, it is cryptographically signed at minimum by the account paying for the fees. Transactions are requests submitted to the network with the expectation that they are processed into consensus order and applied to state.
 
 
## Transaction ID
 
A transaction ID is composed of the payer account ID and the timestamp in seconds.nanoseconds format:
 
```
0.0.9401@1602138343.335616988
 ^^^^^^^  ^^^^^^^^^^^^^^^^^^^^
payer ID   valid start timestamp
```
 
The SDKs generate transaction IDs automatically when submitting transactions - you do not need to create them manually in most cases.
 
Special transaction ID formats:
- **Scheduled:** `0.0.9401@1602138343.335616988?scheduled`
- **Child tx:** `0.0.9401@1602138343.335616988/1` (nonce suffix)
The transaction ID is used for:
- Retrieving receipts and records
- Detecting duplicate transactions
- Referencing the parent transaction from child transactions

 
## Transaction Properties and Limits
 
| Property | Description | Notes |
|----------|-------------|-------|
| Transaction ID | Unique ID: `payerAccount@timestamp` | Auto-generated by SDK |
| Max Transaction Fee | Maximum HBAR the payer agrees to pay | SDK sets a safe default |
| Valid Duration | Seconds the tx is valid (from valid start time) | Default: 120s |
| Memo | Optional string up to 100 bytes | Public - visible on explorers |
| Node Account ID | The specific node to submit to | SDK auto-selects |
| Max Size | **6 KiB** total transaction size | Hard limit |
 
**Queries** are different from transactions - they are processed by a single node and return current state data. Some queries are free (account balance); most carry a small fee.
 
 
## Batch Transactions
 
Batch transactions allow multiple transactions to execute atomically in a single network transaction.
 
A batch transaction ensures that all inner operations either succeed together or fail together, providing ACID properties (atomicity, consistency, isolation, and durability). This enables complex transaction flows without requiring smart contracts, simplifying development and offering better user experiences.
 
All inner transactions must execute within the standard transaction valid duration (typically 3 minutes).
 
```javascript
// Conceptual batch: transfer + log in HCS (atomic)
const batchTx = new BatchTransaction()
    .addInnerTransaction(
        new TransferTransaction()
            .addHbarTransfer(senderAccount, new Hbar(-10))
            .addHbarTransfer(receiverAccount, new Hbar(10))
            .batchKey(batchPublicKey)
    )
    .addInnerTransaction(
        new TopicMessageSubmitTransaction()
            .setTopicId(auditTopicId)
            .setMessage('{"event":"transfer","amount":10}')
            .batchKey(batchPublicKey)
    );
```
 
Practical use case for NemorixPay-type applications: atomically transfer stablecoin and log the remittance event to HCS in a single transaction - guaranteed to both succeed or both fail.
 
 
## Scheduled Transactions
 
Scheduled transactions allow queuing a transaction for future execution while collecting required signatures on-chain. Unlike a batch transaction (immediate, atomic), a scheduled transaction waits until all required signatures are collected, then executes.
 
Key properties:
 
| Property | Details |
|----------|---------|
| Maximum future time | 62 days (5,356,800 seconds) |
| Admin key | Optional - allows deletion of the schedule |
| Signature collection | Any party can submit a ScheduleSignTransaction |
| Auto-execution | Executes automatically once threshold signatures are met |
| Wait for expiry | Can optionally wait until expiration time regardless of signatures |
 
The transaction ID of a scheduled transaction inherits `transactionValidStart` and `accountID` from the `ScheduleCreateTransaction` that created it, plus a `?scheduled` flag.
 
This is ideal for multi-party workflows such as escrow, DAO votes, or corporate fund releases requiring approval from multiple executives.
 
 
## Developer Portal and Testnet Setup
 
### Step-by-step to start developing
 
```
1. Create a testnet account
   - Go to: https://portal.hedera.com
   - Generate a key pair
   - Receive free testnet HBAR
 
2. Set up the SDK in your language
   JavaScript: npm install @hashgraph/sdk
   Java:       <dependency> in pom.xml
   Go:         go get github.com/hiero-ledger/hiero-sdk-go/v2
   Dart:       hedera_flutter_sdk (pub.dev)
 
3. Initialize the client
   const client = Client.forTestnet();
   client.setOperator(accountId, privateKey);
 
4. Submit your first transaction
   const tx = await new TransferTransaction()...execute(client);
   const receipt = await tx.getReceipt(client);
```
 
### Available development environments
 
| Environment | Purpose | HBAR |
|-------------|---------|------|
| Testnet | Primary development and testing | Free via portal/faucet |
| Previewnet | Early access to upcoming features | Free |
| Mainnet | Production | Real HBAR required |
| Local Node | Fully offline development | Simulated |
 
**Local Node:** Hedera provides an official Docker-based local node for fully offline development. See: https://docs.hedera.com/hedera/tutorials/local-node
 
**Developer Playground:** An in-browser coding environment at `portal.hedera.com/playground` supporting Java and JavaScript - no local setup required. Built in collaboration with Kabila.
 
 
## Flutter / Dart SDK
 
### What is hedera_flutter_sdk?
 
`hedera_flutter_sdk` is the first native Flutter/Dart SDK for the Hedera network, built by Nemorix Group, LLC. Unlike wrappers that delegate to another language's SDK, this package implements the Hedera protocol directly in Dart, making it suitable for production mobile applications on iOS and Android.
 
Published at: https://pub.dev/packages/hedera_flutter_sdk
 
### Why a native Dart SDK matters
 
- Flutter is the primary framework for cross-platform mobile development
- Native Dart means no platform-channel overhead or FFI complexity
- Full pub.dev distribution - `flutter pub add hedera_flutter_sdk`
- Type-safe Dart APIs consistent with official SDK design patterns
- Suitable for production apps like NemorixPay

### Current implementation (v0.0.5-dev) - June, 2026
 
Phase 2 of development is in progress, covering cryptographic primitives:
 
```dart
import 'package:hedera_flutter_sdk/hedera_flutter_sdk.dart';
 
// Mnemonic (BIP-39) - English and Spanish wordlists
final mnemonic  = await Mnemonic.generate();        // 24 words
final restored  = Mnemonic.fromWords(wordList);     // restore from words
 
// PrivateKey
final edKey     = PrivateKey.generateED25519();     // Ed25519
final ecKey     = PrivateKey.generateECDSA();       // ECDSA secp256k1
 
// PublicKey derived from PrivateKey
final pubKey    = edKey.publicKey;
 
// DER encoding constants (centralized in HederaConstants)
// HederaConstants.ed25519PrivateKeyPrefix
// HederaConstants.ecdsaPrivateKeyPrefix
```
 
### Development roadmap
 
| Phase | Scope | Status |
|-------|-------|--------|
| Phase 1 | Project structure, Client, network config, base types | Completed |
| Phase 2 | Mnemonic BIP-39, PrivateKey (ED25519 + ECDSA), PublicKey, HederaConstants | In progress (v0.0.5-dev) |
| Phase 3 | HD key derivation (`Mnemonic.toPrivateKey()`), Account Management transactions | Planned |
| Phase 4 | HTS - Token Service transactions | Planned |
| Phase 5 | HCS - Consensus Service | Planned |
| Phase 6 | HFS - File Service, Smart Contract queries | Planned |
 
### pana score and quality
 
The SDK follows Dart/Flutter best practices enforced by `pana`:
- Current score: 140/160 on pub.dev
- All tests passing
- Conventional Commits for version history
- `pre_commit.ps1` script for pre-publish validation

 
## Key Takeaways
 
- Hedera supports **two development paths**: native SDKs (Java, JS, Go, Swift, Rust, C++, Dart) and EVM-compatible Solidity - both can interoperate
- **ECDSA secp256k1 is the recommended key type** for all new accounts - it provides EVM address compatibility, MetaMask support, and full Solidity integration; Ed25519 is supported but limited to non-EVM workflows
- **Key structures** (Simple, KeyList, ThresholdKey, nested) enable enterprise-grade multisig without smart contracts
- The **transaction lifecycle** has 6 steps: create, sign, submit, consensus (3-5s), execute, query - finality is absolute, not probabilistic
- **Transaction IDs** combine payer account ID and timestamp and are auto-generated by the SDK
- **Batch transactions** provide ACID atomicity across multiple operations without smart contracts
- **Scheduled transactions** allow multi-party signature collection on-chain, with execution up to 62 days in the future
- The **Developer Portal** provides free testnet accounts and HBAR; the **Playground** allows in-browser development with no local setup
- `hedera_flutter_sdk` is the **first native Dart/Flutter SDK** for Hedera, published on pub.dev, with Phase 1 completed and Phase 2 in progress.

 
## References
 
| Resource | URL |
|----------|-----|
| Start Building (Official) | https://hedera.com/start-building |
| Developer Tooling | https://hedera.com/developer-tooling |
| SDKs Overview (Docs) | https://docs.hedera.com/hedera/sdks-and-apis/sdks |
| Accounts - Core Concepts | https://docs.hedera.com/hedera/core-concepts/accounts |
| Account Properties | https://docs.hedera.com/hedera/core-concepts/accounts/account-properties |
| Keys and Signatures | https://docs.hedera.com/hedera/core-concepts/keys-and-signatures |
| Transactions and Queries | https://docs.hedera.com/hedera/core-concepts/transactions-and-queries |
| Transaction ID | https://docs.hedera.com/hedera/sdks-and-apis/sdks/transactions/transaction-id |
| Batch Transactions | https://docs.hedera.com/hedera/sdks-and-apis/sdks/transactions/create-a-batch-transaction |
| Scheduled Transactions | https://docs.hedera.com/hedera/core-concepts/scheduled-transaction |
| Developer Portal (Testnet) | https://portal.hedera.com |
| Developer Playground | https://portal.hedera.com/playground |
| Hedera Faucet | https://faucet.hedera.com |
| Local Node Setup | https://docs.hedera.com/hedera/tutorials/local-node |
| hedera_flutter_sdk (pub.dev) | https://pub.dev/packages/hedera_flutter_sdk |
| hedera_flutter_sdk (GitHub) | https://github.com/nemorixpay/hedera-flutter-sdk |
| hedera_flutter_sdk (Docs) | https://github.com/nemorixgroup/Hedera-Flutter-SDK-Docs |
| Hiero SDK JavaScript | https://github.com/hiero-ledger/hiero-sdk-js |
| Hiero SDK Java | https://github.com/hiero-ledger/hiero-sdk-java |
| Hiero SDK Swift | https://github.com/hiero-ledger/hiero-sdk-swift |
 
---
 
*Previous: [Module 04 - Native Services](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/module-04-services/README.md)*  
*Next: [Module 06 - Ecosystem & Future](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/module-06-ecosystem/README.md)*
 
---
 
*Last updated: June 2026*  
*Part of the [Hedera Knowledge Base](https://github.com/nemorixgroup/Hedera-Knowledge-Base)*
