# v0.2.0-dev: Fungible Token Creation

Status: ✅ Done  
Files:
  - [Token Create Transaction](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/transactions/token_create_transaction.dart)
  - [Token Create Example](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/example/phase3/token_create_example.dart)
  - [Token Create Test](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/test/unit/transactions/token_create_transaction_test.dart)

## Summary

The first Phase 3 release implements `TokenCreateTransaction`,
covering fungible (and non-fungible) token creation on the Hedera
Token Service (HTS), with all 22 fields from the official token
creation reference.

## Files Added / Modified

### Added

- `lib/src/transactions/token_create_transaction.dart`;
  `TokenCreateTransaction`, all 22 fields
- `example/phase3/token_create_example.dart`; live testnet example
- `test/unit/transactions/token_create_transaction_test.dart`;
  66 unit tests

### Modified

- `lib/src/transactions/transaction.dart`; `executeGrpc()` signature
  change (`ClientChannel` instead of `CryptoServiceClient`)
- `lib/src/transactions/account_create_transaction.dart`;
  `lib/src/transactions/account_update_transaction.dart`;
  `lib/src/transactions/account_delete_transaction.dart`;
  `lib/src/transactions/crypto_transfer_transaction.dart`; updated
  `executeGrpc()` to match the new signature
- `lib/src/models/token_id.dart`; added `toProto()`
  
## Official References

- [Create and Transfer Your First Fungible Token](https://docs.hedera.com/hedera/tutorials/token/create-and-transfer-your-first-fungible-token)
- [Define a Token (SDK reference)](https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/define-a-token)

## Key Decisions

### All 22 fields modeled from the start

`TokenCreateTransaction` exposes every field from the official
reference, not just the three required ones (`tokenName`,
`tokenSymbol`, `treasuryAccountId`). This was a deliberate choice:
Hedera's token keys (admin, KYC, freeze, wipe, supply, pause, fee
schedule, metadata) are set once at creation. If a key is not set
here, it can never be added later; if a key is set here, it can
never be removed later. Omitting a field from the initial model
would mean a developer could not get it right on the first try.

### Proto field names diverge from documented setter names

Verifying `TokenCreateTransactionBody` against the generated
`token_create.pb.dart` (not just the docs) surfaced several
mismatches between the documented setter names and the actual
Protobuf field names:

| Documented name | Actual proto field |
|---|---|
| `treasuryAccountId` | `treasury` |
| `expirationTime` | `expiry` |
| `autoRenewAccountId` | `autoRenewAccount` |
| `tokenMemo` | `memo` |

The SDK's public API keeps the documented, more descriptive names
(`setTreasuryAccountId()`, `setExpirationTime()`, etc.), mapping to
the correct underlying proto field internally.

### Token keys accept any HederaKey

Admin, KYC, freeze, wipe, supply, pause, fee schedule, and metadata
keys all accept `HederaKey`, the same interface introduced in
v0.1.4-dev for multi-signature accounts. This means any of them can
be a single `PublicKey`, a `HederaKeyList` (N-of-N), or a
`HederaThresholdKey` (M-of-N), enabling multi-signature control over
token administration, not just account administration.

### executeGrpc() breaking change

`Transaction.executeGrpc()` previously received a pre-built
`CryptoServiceClient`, hardcoding every transaction to the Crypto
Service. Token transactions require `TokenServiceClient` instead.
Fixed by changing the method to receive the raw `ClientChannel`,
with each subclass constructing the specific gRPC service client it
needs. This also unblocks Hedera Consensus Service (HCS)
transactions in a later phase.

### ECDSA token key fix

`PublicKey.toProtoKey()` (via `HederaKey`) now correctly encodes
ECDSA keys with the `eCDSASecp256k1` Protobuf field for token keys,
closing the same class of bug fixed for account keys in v0.1.4-dev.

### Custom fees deferred to v0.2.7-dev

`setCustomFees()` is not included in this version. Custom fees
(fixed, fractional, royalty) are a self-contained subsystem, and
royalty fees specifically only apply to non-fungible tokens, which
are not implemented until v0.2.4-dev. Deferred to its own version
after NFT, KYC, freeze, and query support are in place.

## Verified

Live on Hedera testnet:

- Created a treasury account and a fungible token ("USD Bar" / USDB,
  2 decimals, 100 initial supply), confirmed `SUCCESS` status and
  correct sequential token ID via HashScan.
- Confirmed token creation requires a higher `maxTransactionFee`
  than account creation: the inherited 2 HBAR default triggers
  `INSUFFICIENT_TX_FEE`, since token creation includes a
  `CryptoTransfer` to move the initial supply to the treasury.
  30 HBAR, matching Hedera's own example fee override, is used in
  `example/phase3/token_create_example.dart`.

## Tests

66 unit tests in `token_create_transaction_test.dart`: defaults,
setters, `toBytes()` serialization for all 22 fields (including a
`HederaThresholdKey` as a token key), and `buildBody()` integration
confirming the `tokenCreation` oneof field routing and required-field
validation.

## Related

- [Phase 2: Crypto + Accounts](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/docs-sdk/phase-2/README.md)
