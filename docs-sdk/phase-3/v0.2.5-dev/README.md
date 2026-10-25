# v0.2.5-dev: KYC and Freeze

Status: ✅  Done

## Summary

Implements the two compliance controls an HTS token issuer can apply to
a single holder: KYC (`TokenGrantKycTransaction` and
`TokenRevokeKycTransaction`) and freeze (`TokenFreezeTransaction` and
`TokenUnfreezeTransaction`). Also fixes wrong numeric values in
`HederaStatusCode` found while planning the live example, and makes
`HederaStatusCode.fromCode()` resolve any network status through the
generated protobuf enum.

## Files Added / Modified

### Added

- `lib/src/transactions/tokens/token_grant_kyc_transaction.dart`;
  `TokenGrantKycTransaction`
- `lib/src/transactions/tokens/token_revoke_kyc_transaction.dart`;
  `TokenRevokeKycTransaction`
- `lib/src/transactions/tokens/token_freeze_transaction.dart`;
  `TokenFreezeTransaction`
- `lib/src/transactions/tokens/token_unfreeze_transaction.dart`;
  `TokenUnfreezeTransaction`
- `example/phase3/token_kyc_freeze_example.dart`; live testnet example
- `test/unit/transactions/tokens/token_grant_kyc_transaction_test.dart`,
  `token_revoke_kyc_transaction_test.dart`,
  `token_freeze_transaction_test.dart` and
  `token_unfreeze_transaction_test.dart`; 52 unit tests in total

### Modified

- `lib/src/core/hedera_status_code.dart`; five corrected status codes,
  seven new constants, and the `ResponseCodeEnum` fallback in
  `fromCode()`
- `test/unit/core/hedera_status_code_test.dart`; rewritten, now with a
  test that checks every named constant against the protobuf enum
- `lib/hedera_flutter_sdk.dart`; exports the four new classes
- `example/hedera_flutter_sdk_example.dart`; wires the new example

## Official References

- https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/enable-kyc-account-flag
- https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/disable-kyc-account-flag
- https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/freeze-an-account
- https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/unfreeze-an-account
- https://github.com/hashgraph/hedera-protobufs/blob/main/services/response_code.proto

## Key Decisions

### What KYC and freeze do

KYC ("Know Your Customer") lets an issuer restrict a token to approved
holders: while the token has a KYC key, an account cannot transact with
the token until KYC is granted to it. Freeze is a separate control: a
frozen account can neither send nor receive the token. They are
independent, they use different keys, and a token can have either, both,
or neither. Both keys can only be set when the token is created, which
is why `TokenCreateTransaction` has supported `kycKey`, `freezeKey` and
`freezeDefault` since v0.2.0-dev.

Each of the four transactions takes the same two required fields,
`tokenId` and `accountId`. The token's KYC key (for grant and revoke) or
freeze key (for freeze and unfreeze) must sign together with the
transaction fee payer, and the account must already be associated with
the token.

### Four independent classes with the official names

The classes use the names from the official documentation
(`TokenGrantKycTransaction`, `TokenRevokeKycTransaction`,
`TokenFreezeTransaction`, `TokenUnfreezeTransaction`), the same
approach used for the other token transactions. They are four
independent classes, as with `TokenAssociateTransaction` and
`TokenDissociateTransaction`, rather than subclasses of a shared base.
The four are almost identical, but each needs its own dartdoc,
`applyToBody()` and `executeGrpc()`, so a shared base would save very
little and break the symmetry with the existing classes.

### Protobuf names differ from the documentation

Verifying against the generated protobufs (and the upstream
`hedera-protobufs`) showed several names that do not match the
documented ones:

- The freeze and unfreeze bodies are
  `TokenFreezeAccountTransactionBody` and
  `TokenUnfreezeAccountTransactionBody`; the KYC bodies are
  `TokenGrantKycTransactionBody` and `TokenRevokeKycTransactionBody`.
- The account field in every body is named `account`, not `accountId`,
  the same pattern seen earlier with `treasury` and `autoRenewAccount`.
- The `TransactionBody` oneof fields are `tokenFreeze` (31),
  `tokenUnfreeze` (32), `tokenGrantKyc` (33) and `tokenRevokeKyc` (34).
- The `TokenServiceClient` methods are `freezeTokenAccount`,
  `unfreezeTokenAccount`, `grantKycToTokenAccount` and
  `revokeKycFromTokenAccount`.

### Local validation is limited to the required fields

Each class throws `ArgumentError` from `toBytes()` and `buildBody()` if
`tokenId` or `accountId` is missing. The SDK does not check whether the
token has the needed key. As with the supply key in mint and burn, the
network resolves it (`TOKEN_HAS_NO_KYC_KEY`, `TOKEN_HAS_NO_FREEZE_KEY`).

### Freeze and unfreeze are idempotent

The protobuf definitions state that freezing an account that is already
frozen succeeds with no change, and the documentation says the same
about unfreezing. The SDK does not add any local state to track this.

### Status code numbers were wrong, and are now checked automatically

Planning the live example showed that it would report some network
errors incorrectly. Comparing `HederaStatusCode` with the official
`response_code.proto` found five constants with a wrong number:

| Constant | Number in the SDK | Official number |
|---|---|---|
| `insufficientAccountBalance` | 18 | 28 |
| `accountFrozenForToken` | 178 | 165 |
| `accountKycNotGrantedForToken` | 181 | 176 |
| `invalidTokenId` | 150 | 167 |
| `tokenIsPaused` | 260 | 265 |

The old numbers belonged to other statuses (`RECEIPT_NOT_FOUND`,
`INSUFFICIENT_TOKEN_BALANCE`, `TOKEN_HAS_NO_WIPE_KEY`,
`INVALID_TOPIC_ID`, `SERIAL_NUMBER_LIMIT_REACHED`), so a real KYC or
freeze rejection would have been shown as `UNKNOWN_STATUS` or under the
wrong name. The constant names are unchanged.

Fixing only the numbers would have left the root cause in place: a
hand-typed table that silently drifts from the protocol. Two changes
address that:

- `fromCode()` now falls back to the generated `ResponseCodeEnum` for
  any code without a named constant, so `name` shows the official
  status name. `UNKNOWN_STATUS` remains only for codes that are not
  defined in the protobuf either, and the original code is preserved.
- A unit test checks every named constant against `ResponseCodeEnum`
  (code and name), so a wrong number now fails the test suite.

Seven constants were added for the errors these transactions can
return: `insufficientTxFee`, `accountDeleted`, `insufficientTokenBalance`,
`tokenHasNoFreezeKey`, `tokenHasNoKycKey`, `tokenHasNoSupplyKey` and
`tokenWasDeleted`.

### Observed behavior of the treasury account

In the live run, the treasury of a token created with a KYC key and a
freeze key transferred tokens without being granted KYC and without
being unfrozen, so the treasury is not subject to these controls. This
is an observation from testnet, not something the pages consulted for
this version state.

### Folder structure stays flat for now

The four classes live in `lib/src/transactions/tokens/` next to the
other token transactions, which now number nine. Regrouping them by
theme is planned as a separate version (v0.2.8-dev), once the full set
of Phase 3 classes is known, so files are moved only once. It is a
mechanical change with no effect on the public API.

## Verified

Live on Hedera testnet: created a token with a KYC key and a freeze key,
then created and associated a second account. A transfer to it before
KYC was rejected with `ACCOUNT_KYC_NOT_GRANTED_FOR_TOKEN`; after
granting KYC it succeeded; while the account was frozen it was rejected
with `ACCOUNT_FROZEN_FOR_TOKEN`; after unfreezing it succeeded; and
after revoking KYC it was rejected again with
`ACCOUNT_KYC_NOT_GRANTED_FOR_TOKEN`. The grant, freeze, unfreeze and
revoke transactions all returned `SUCCESS`. The status names appearing
correctly in this run depend on the `HederaStatusCode` fix above.

## Tests

78 new unit tests (699 to 777 total): 13 for each of the four new
classes (defaults, setters, `toBytes()` validation and serialization,
`buildBody()` integration confirming the oneof routing, and signing),
and 26 more in `hedera_status_code_test.dart`, which now has 41 tests,
including one per named constant checked against `ResponseCodeEnum`.

## Out of Scope for v0.2.5-dev

- Reading KYC and freeze status back. These transactions have no query
  counterpart in the consensus node API, so the example checks them by
  behavior. Token relationship queries are planned for v0.2.6-dev,
  designed around the Mirror Node REST API.
- Wipe, pause and unpause, token deletion and `TokenUpdateTransaction`.
- A live example using `freezeDefault`.
- Regrouping the token classes by theme; planned as v0.2.8-dev.

## Related

- [v0.2.4-dev: NFT Support](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/docs-sdk/phase-3/v0.2.4-dev/README.md)
