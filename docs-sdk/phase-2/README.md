# Phase 2: Crypto + Accounts

Status: ✅ Done (v0.0.4-dev to v0.1.4-dev)  
Files:  
  - [Hedera Client](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/client/hedera_client.dart)
  - [Hedera Node](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/client/hedera_node.dart)
  - [Retry Policy](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/client/retry_policy.dart)
  - [Transaction](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/transactions/transaction.dart)
  - [Create Transaction](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/transactions/account_create_transaction.dart)
  - [Crypto Transfer Transaction](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/transactions/crypto_transfer_transaction.dart)
  - [Query](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/queries/query.dart)
  - [Hedera Key](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/crypto/hedera_key.dart)
  - [Hedera Key List](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/crypto/hedera_key_list.dart)
  - [Private Key](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/crypto/private_key.dart)
  - [Public Key](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/crypto/public_key.dart)

## Summary

Phase 2 built the cryptographic and account-management core of the
SDK: wallet generation, both Hedera key types (ED25519 and ECDSA),
the full account lifecycle, HBAR transfers, network resilience
(multi-node load balancing and retry/failover), and multi-signature
account support.

## Key Decisions

### Cryptography and wallets

- **BIP-39 mnemonics in English and Spanish**, including the first
  native Spanish mnemonic wordlist among Hedera SDKs, aimed at
  NemorixPay's LATAM remittance use case.
- **`PrivateKey`/`PublicKey`** support both Hedera key types:
  ED25519 (recommended for native Hedera accounts) and ECDSA
  secp256k1 (for EVM wallet compatibility).
- **ECDSA signing uses Keccak-256, not SHA-256**, per HIP-222.
  This was caught as a bug during implementation: the initial ECDSA
  signing code hashed the message with SHA-256, which would have
  produced signatures the network silently rejects. Fixed with
  pointycastle's `KeccakDigest(256)`.
- **`HederaKey`, `HederaKeyList`, `HederaThresholdKey`** model
  Hedera's key structures (a single key, an N-of-N list, or an
  M-of-N threshold), so any of them can be used interchangeably
  wherever an account or token key is required, including
  multi-signature account control.

### Accounts and transactions

- **`Transaction<T>`/`Query<R,T>`** are generic base classes using
  the Generic Self-Type pattern, so fluent setters from the base
  class and a subclass can be chained in any order while still
  returning the concrete subclass type.
- **Full account CRUD**: `AccountCreateTransaction`,
  `AccountUpdateTransaction`, `AccountDeleteTransaction`, plus
  `CryptoTransferTransaction` for HBAR transfers with sum-zero
  validation.
- **`signWith()` and `setPayerAccountId()`** enable non-operator
  signing and custom fee payers, required for flows where the
  account acting is not the client's configured operator.

### Network resilience

- **Multi-node load balancing**: `HederaClient` fetches the live
  consensus node list from the Mirror Node REST API
  (`/api/v1/network/nodes`), cached for 24 hours, and rotates
  through it with round-robin selection instead of a single
  hardcoded node.
- **`RetryPolicy`**: retries transient node/network failures
  (`UNAVAILABLE`, `deadlineExceeded`, `internal`) with exponential
  backoff; never retries business errors like `INVALID_SIGNATURE`,
  since those fail identically against any node.
- **Critical fix**: the gRPC connection used to submit a transaction
  must match the specific node declared in that transaction's
  `nodeAccountID`, not a generic fixed endpoint. A mismatch (which
  could occur once node rotation was introduced) causes
  `INVALID_NODE_ACCOUNT`. Fixed via `HederaClient.channelFor()` and
  `resolveNode()`.

## Verified

- 502 unit tests passing at the close of Phase 2.
- Multiple accounts, transfers, and a 2-of-3 multi-signature account
  verified live against Hedera testnet, visible on HashScan.

## Related

- [Phase 1: Architecture + Protobuf](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/docs-sdk/phase-1/README.md)  
- [SDK Technical Decisions](https://github.com/nemorixgroup/Hedera-Knowledge-Base/tree/main/docs-sdk)
