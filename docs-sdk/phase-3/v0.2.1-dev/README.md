# v0.2.1-dev: Account-to-Token Association

Status: ✅ Done  
Files:  
- [TokenAssociateTransaction](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/transactions/token_associate_transaction.dart)
- [TokenDissociateTransaction](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/transactions/token_dissociate_transaction.dart)
- [Example](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/example/phase3/token_associate_example.dart)

## Summary

Implements `TokenAssociateTransaction` and `TokenDissociateTransaction`,
the required step before an account can send or receive an HTS token
that it did not create as treasury.

## Files Added / Modified

### Added

- `lib/src/transactions/token_associate_transaction.dart`;
  `TokenAssociateTransaction`
- `lib/src/transactions/token_dissociate_transaction.dart`;
  `TokenDissociateTransaction`
- `example/phase3/token_associate_example.dart`; live testnet example
- `test/unit/transactions/token_associate_transaction_test.dart`;
  `test/unit/transactions/token_dissociate_transaction_test.dart`;
  34 unit tests total

### Modified

None. Both transactions were implemented directly against the
`ClientChannel`/`TokenServiceClient` pattern established in
v0.2.0-dev; no changes were needed to `Transaction` or any existing
transaction class.

## Official References

- https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/associate-tokens-to-an-account
- https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/dissociate-tokens-from-an-account

## Key Decisions

### Minimal field set, confirmed against the regenerated protobuf

Both transactions expose only `accountId` (required) and a list of
`tokenIds` (required, at least one), matching
`TokenAssociateTransactionBody`/`TokenDissociateTransactionBody`
exactly. `addTokenId()` follows the same accumulator pattern already
used by `CryptoTransferTransaction.addHbarTransfer()`, rather than
the single `setTokenIds(List)` call shown in the official docs.

### The setHighVolume() question

The official docs list a `setHighVolume()` property on
`TokenAssociateTransaction`, tied to HIP-1313 (a high-volume entity
creation lane shipped on mainnet/testnet in network release v0.73,
May 2026). The SDK's Protobuf definitions were regenerated from the
official Hedera protobufs source to check for a corresponding
`high_volume` field on `TokenAssociateTransactionBody`; none was
found. Cross-checking the actively maintained
`hiero-ledger/hiero-sdk-java` reference implementation showed the
same: its `TokenAssociateTransaction.build()` does not reference a
`highVolume` field either.

Working theory: HIP-1313 has been confirmed for `FileAppend` and
`ConsensusCreateTopic` specifically, but may not yet extend to
`TokenAssociateTransactionBody`. This mirrors an earlier pattern
seen in v0.2.0-dev, where the docs referenced `setCustomFees()`
ahead of a full evaluation. `setHighVolume()` is deliberately left
out of the SDK for now, to be added once confirmed on the protobuf
side.

## Verified

Live on Hedera testnet:

- Created a treasury account and a fungible token ("Demo Coin" /
  DEMO)
- Created a second account (Bob), not associated with the token
- Associated Bob's account with the token (`SUCCESS`)
- Dissociated Bob's account from the token with a zero balance
  (`SUCCESS`)
- Confirmed no `maxTransactionFee` override was needed for either
  transaction, unlike `TokenCreateTransaction` in v0.2.0-dev

## Tests

34 unit tests across `token_associate_transaction_test.dart` and
`token_dissociate_transaction_test.dart`: defaults, setters,
`toBytes()` validation and serialization, and `buildBody()`
integration confirming `tokenAssociate`/`tokenDissociate` oneof
field routing.

## Related

- [v0.2.0-dev: Fungible Token Creation](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/docs-sdk/phase-3/v0.2.0-dev/README.md)
