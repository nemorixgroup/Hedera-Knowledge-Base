# SDK Technical Decisions

This section documents the implementation decisions, technical
rationale, and key references behind **hedera_flutter_sdk**, the
first native Flutter/Dart SDK for Hedera.

Every decision documented here is grounded in official Hedera
sources (docs.hedera.com, the Hedera HAPI Protobuf definitions, and
HIPs). Where relevant, implementation code and test vectors are
included to make each decision fully verifiable.

## Why This Documentation Exists

Building a production-quality SDK for Hedera requires making
decisions that are not always obvious: which cryptographic scheme to
use for a given key type, why a specific field diverges from its
documented name, how a transaction body maps to the underlying
Protobuf.

This documentation answers those questions for:

- **Contributors** who want to understand the rationale before
  submitting a pull request
- **Developers** who want to verify that the SDK follows official
  Hedera standards
- **Auditors** who need a clear trail from specification to
  implementation

## Structure

```
docs-sdk/
  README.md <- You are here
  phase-1/  <- Architecture + Protobuf (summary)
  phase-2/  <- Crypto + Accounts (summary)
  phase-3/  <- HTS Tokens + NFTs (per-version detail)
  phase-4/  <- Mirror Node + HCS
  phase-5/  <- Docs + pub.dev v1.0
  phase-6/  <- Launch + HIP
```

## Phase 1: Architecture + Protobuf

[Phase 1 - Summary](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/docs-sdk/phase-1/README.md)

## Phase 2: Crypto + Accounts

[Phase 2 - Summary](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/docs-sdk/phase-2/README.md)

## Phase 3: HTS Tokens + NFTs

| Version | Description | Status |
|---------|-------------|--------|
| [v0.2.0-dev](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/docs-sdk/phase-3/v0.2.0-dev/README.md) | Fungible token creation | ✅ Done |
| [v0.2.1-dev](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/docs-sdk/phase-3/v0.2.1-dev/README.md) | Account-to-token association | ✅ Done |
| v0.2.2-dev | Token transfers | 🔄 Next |
| v0.2.3-dev | Mint / Burn | ⏳ Pending |
| v0.2.4-dev | NFT support | ⏳ Pending |
| v0.2.5-dev | KYC and freeze | ⏳ Pending |
| v0.2.6-dev | Token queries | ⏳ Pending |
| v0.2.7-dev | Custom fees | ⏳ Pending |

## Phase 4: Mirror Node + HCS

⏳ Pending

## Phase 5: Docs + pub.dev v1.0

⏳ Pending

## Phase 6: Launch + HIP

⏳ Pending

## Related

- [hedera_flutter_sdk](https://github.com/nemorixgroup/hedera-flutter-sdk); the SDK itself
- [Hedera Knowledge Base](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/README.md); official source documentation
