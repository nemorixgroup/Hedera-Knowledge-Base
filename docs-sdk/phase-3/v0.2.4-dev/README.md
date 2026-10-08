# v0.2.4-dev: NFT Support

Status: ✅  Done

## Summary

Adds NFT support to the Hedera Token Service layer: minting NFTs,
burning NFTs, transferring NFTs, and reading the serial numbers of
minted NFTs from the receipt. Following the official Hedera SDKs, NFTs
reuse the same classes already used for fungible tokens
(`TokenCreateTransaction`, `TokenMintTransaction`,
`TokenBurnTransaction`, `CryptoTransferTransaction`); there are no
separate NFT transaction classes.

## Files Added / Modified

### Added

- `lib/src/models/nft_id.dart`; `NftId` (token ID plus serial number)
- `example/phase3/token_nft_example.dart`; live testnet example
- `test/unit/models/nft_id_test.dart`;
  `test/unit/transactions/transaction_receipt_test.dart`;
  `test/unit/transactions/tokens/token_create_nft_test.dart`

### Modified

- `lib/src/transactions/tokens/token_mint_transaction.dart`;
  `addMetadata()`, `setMetadata()`, exactly-one-of validation
- `lib/src/transactions/tokens/token_burn_transaction.dart`;
  `addSerial()`, `setSerials()`, exactly-one-of validation
- `lib/src/transactions/tokens/token_create_transaction.dart`; local
  validation for `NON_FUNGIBLE_UNIQUE` tokens
- `lib/src/transactions/crypto_transfer_transaction.dart`;
  `addNftTransfer()`, `nftTransferCount`, fungible/NFT mixing check
- `lib/src/transactions/transaction.dart` and
  `lib/src/queries/transaction_receipt_query.dart`;
  `TransactionReceipt.serialNumbers`
- `lib/hedera_flutter_sdk.dart`; exports `NftId`, `TokenType` and
  `TokenSupplyType`
- `test/unit/transactions/tokens/token_mint_transaction_test.dart`,
  `token_burn_transaction_test.dart` and
  `test/unit/transactions/crypto_transfer_transaction_test.dart`;
  new NFT tests added
- `example/hedera_flutter_sdk_example.dart`; wires the new example
- `example/phase3/token_create_example.dart` and
  `token_associate_example.dart`; no longer print private keys

## Official References

- https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/mint-a-token
- https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/burn-a-token
- https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/transfer-tokens

## Key Decisions

### Same classes as fungible tokens

The official documentation has no separate NFT mint, burn, or transfer
classes: the same `TokenMintTransaction`, `TokenBurnTransaction`, and
`TransferTransaction` handle both token kinds, and the protobuf
bodies carry both sets of fields (`amount` plus `metadata` for mint,
`amount` plus `serialNumbers` for burn, `transfers` plus
`nftTransfers` for transfer lists). This SDK follows the same shape.
It also keeps the public API smaller and matches what developers
coming from other Hedera SDKs expect.

### NftId

An NFT is identified by its collection (the token ID) and a serial
number. `NftId` is an immutable value object with value equality.
`toString()` returns `tokenId/serial`, and `NftId.fromString()`
accepts both `/` and `@` as separators, since both notations appear in
Hedera tooling and documentation.

### Mint and burn accept exactly one mode

The protobuf definitions state that mint and burn must not specify
both a fungible quantity and NFT data in the same transaction.
Instead of letting the network reject it, `TokenMintTransaction` and
`TokenBurnTransaction` throw an `ArgumentError` when built with
neither `amount` nor `metadata`/`serials`, or with both.
`setAmount(0)` remains valid, since zero-unit operations are allowed
(HIP-564).

The real protobuf field for burn is `serialNumbers`; the SDK exposes
it as `addSerial()` and `setSerials()`, as in the documentation.

### Mint metadata

Each metadata entry mints one NFT. The official limit is 100 bytes per
entry, typically an IPFS URI (HIP-412). The SDK copies the bytes on
`addMetadata()` so later changes to the caller's list do not alter the
transaction. The receipt returns the serial numbers in the same order
as the metadata entries.

### Transfers: fungible and NFT never mix per token

`TokenTransferList` in the protobuf contains either `transfers`
(fungible) or `nftTransfers` (NFT), never both. If the same token is
used with `addTokenTransfer()` and `addNftTransfer()` in one
`CryptoTransferTransaction`, the SDK throws an `ArgumentError`. The
current owner of the NFT signs, and the receiver must already be
associated with the token. `expectedDecimals` is not used for NFTs.

### Minimal local validation in TokenCreateTransaction

Only one rule was added: a `NON_FUNGIBLE_UNIQUE` token with `decimals`
or `initialSupply` different from 0 is rejected locally, since that
combination can never succeed and the message points to
`TokenMintTransaction.addMetadata()`. Everything else is left to the
network (supply key, `FINITE` supply type, `maxSupply`), consistent
with how the SDK treats other business rules.

### No hardcoded batch limit

The number of NFTs per mint or burn is limited by network properties
(`tokens.nfts.maxBatchSizeMint` and `tokens.nfts.maxBatchSizeBurn`).
Because these can change, the SDK does not hardcode a limit and lets
the network respond.

### Receipt: serialNumbers

`TransactionReceipt` now exposes `serialNumbers` (the protobuf
receipt field of the same name), populated from `getReceipt()` and
`TransactionReceiptQuery`. `newTotalSupply` is not exposed yet.

## Verified

Live on Hedera testnet: created an NFT collection with a supply key,
minted 3 NFTs (`SUCCESS`, serials 1, 2 and 3), associated a second
account and transferred serial 1 to it (`SUCCESS`), then burned
serial 3 from the treasury (`SUCCESS`), confirmed on HashScan. Run
both as the individual example and as part of the combined example,
with the same results.

## Tests

55 new unit tests (644 to 699 total) covering `NftId` (parsing,
equality, proto conversion), receipt `serialNumbers`,
`TokenCreateTransaction` NFT validation, mint and burn (setters,
exactly-one-of validation, serialization, `buildBody()` routing), and
`CryptoTransferTransaction` NFT transfers (validation, mixing check,
serialization).

## Out of Scope for v0.2.4-dev

- BUSY retry in `RetryPolicy`; planned as its own later version.
- `isApproval` on NFT transfers and `newTotalSupply` on the receipt.
- `setHighVolume()` (HIP-1313); same open question as in v0.2.1-dev
  and v0.2.3-dev.
- NFT and token balance queries; planned for v0.2.6-dev. The consensus
  node `AccountBalanceQuery` is deprecated for token balances
  (HIP-367), so these queries will be designed around the Mirror Node
  REST API.
- KYC and freeze; v0.2.5-dev.

## Related

- [v0.2.3-dev: Fungible Token Mint / Burn](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/docs-sdk/phase-3/v0.2.3-dev/README.md)
