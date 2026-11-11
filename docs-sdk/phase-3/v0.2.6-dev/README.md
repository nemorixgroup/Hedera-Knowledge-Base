# v0.2.6-dev: Token Queries (Mirror Node)

Status: ✅  Done

## Summary

Adds the way to read token data back from the network: account and token
balances (with KYC and freeze status), token information, and NFT
information. The reads use the Mirror Node REST API, through a new
`MirrorNodeClient` and four query classes. The version also deprecates
the Phase 2 `AccountBalanceQuery`, which Hedera removed from consensus
nodes in release v0.77, and closes the gap left by v0.2.5-dev, where KYC
and freeze could only be checked by behavior.

## Files Added / Modified

### Added

- `lib/src/client/mirror_node_client.dart`; `MirrorNodeClient`
- `lib/src/core/mirror_node_exception.dart`; `MirrorNodeException`
- `lib/src/queries/mirror/mirror_node_account_balance.dart` and
  `mirror_node_account_balance_query.dart`; HBAR balance
- `lib/src/queries/mirror/mirror_node_token_balance.dart` and
  `mirror_node_token_balance_query.dart`; token balances, plus the
  `KycStatus` and `FreezeStatus` enums
- `lib/src/queries/mirror/mirror_node_token_info.dart` and
  `mirror_node_token_info_query.dart`; token information, plus the
  `PauseStatus` enum
- `lib/src/queries/mirror/mirror_node_nft_info.dart` and
  `mirror_node_token_nft_info_query.dart`; NFT information
- `lib/src/queries/mirror/mirror_node_key.dart`; `MirrorNodeKey` and
  `MirrorNodeKeyType`
- `lib/src/queries/mirror/mirror_node_custom_fees.dart`; read models for
  custom fees
- `lib/src/queries/mirror/mirror_json.dart`; internal JSON helpers (not
  exported)
- `example/phase3/token_queries_example.dart`; live testnet example
- Unit tests under `test/unit/core/`, `test/unit/client/` and
  `test/unit/queries/mirror/`; 173 new tests in total

### Modified

- `lib/src/client/hedera_client.dart`; owns a lazily created
  `http.Client`, adds `mirrorNode`, `mirrorNodeBaseUrl` and
  `setHttpClient()`; the node list fetch now uses that client
- `lib/src/queries/account_balance_query.dart`; `@Deprecated`
- `lib/src/queries/query.dart`; documentation states the real status
- `example/phase2/query_example.dart`; reads a real balance from the
  Mirror Node
- `lib/hedera_flutter_sdk.dart`; exports the new classes
- `example/hedera_flutter_sdk_example.dart` and
  `example/phase3/token_transfer_example.dart`; wire the new example and
  update stale comments
- `test/unit/queries/account_balance_query_test.dart`; documented
  `ignore_for_file` for the intentional use of the deprecated class

## Official References

- https://docs.hedera.com/hedera/sdks-and-apis/sdks/cryptocurrency/get-account-balance
- https://docs.hedera.com/native/tokens/get-balance
- https://docs.hedera.com/native/queries
- https://docs.hedera.com/hedera/sdks-and-apis/rest-api
- https://hedera.com/blog/migrating-from-accountbalancequery-what-you-need-to-know
- https://hips.hedera.com/hip/hip-367
- https://github.com/hiero-ledger/hiero-mirror-node/blob/main/rest/api/v1/openapi.yml

## Key Decisions

### Why the Mirror Node, and not consensus queries

The consensus node `AccountBalanceQuery` was removed in release v0.77
(testnet on September 15, 2026; mainnet on September 29, 2026), and
returning token balances from consensus nodes has been deprecated since
Services release 0.50 (HIP-367). The official documentation now points
to the Mirror Node for these reads, and its SDKs expose
`MirrorNodeAccountBalanceQuery` and `MirrorNodeTokenBalanceQuery` for it.

Two options were analyzed: Mirror Node REST only, or also implementing
the consensus queries. The consensus path was left out because
`Query.execute()` was never implemented in this SDK (it throws
`UnimplementedError`), so those queries would first need gRPC execution
and a signed payment attached to each query. That is infrastructure of
its own and is planned as v0.3.0-dev, together with `TokenInfoQuery`,
`TokenNftInfoQuery` and `AccountInfoQuery` execution. Mirror Node reads
are free and need no operator account, which also makes them the better
fit for a wallet-style app.

### Naming: MirrorNode prefix on every result and fee model

Result models are named `MirrorNodeTokenInfo`, `MirrorNodeNftInfo` and so
on, so the names `TokenInfo` and `TokenNftInfo` stay free for the
consensus queries. The read-only fee models are named
`MirrorNodeFixedFee`, `MirrorNodeFractionalFee` and `MirrorNodeRoyaltyFee`
so the names for the custom fees set at token creation stay free for
v0.2.7-dev. The public protobuf enums `TokenType` and `TokenSupplyType`
are reused instead of duplicated.

### No shared base query class

The four queries are independent classes that call `MirrorNodeClient`.
They do not extend `Query`, since that class models consensus queries
(protobuf request, node, payment). Sharing it would drag unused concepts
into a plain REST call, and the REST queries need very little shared
code.

### MirrorNodeClient

`MirrorNodeClient` offers two operations:

- `getJson(path, {queryParameters, notFoundStatus})`; one request, one
  JSON object.
- `getAll(path, {listKey, ...})`; follows the `links.next` relative path
  until it is absent, with a `maxPages` safety limit (100) and a default
  `limit` of 100.

Retry follows the client `RetryPolicy`. It retries HTTP 5xx, HTTP 429
(the Mirror Node can rate limit), timeouts, `SocketException` and
`http.ClientException`. Any other 4xx is never retried, since it would
fail identically. A `maxAttempts` below 1 is treated as 1. The delay
function and the `http.Client` can be injected, so the retry loop is
tested without waiting.

The Mirror Node error body is `{"_status":{"messages":[{"message":...}]}}`
(checked with real requests: `Not found` for a 404 and
`Invalid parameter: tokenid` for a 400), and `MirrorNodeException`
carries the request path, the status code and that message. When the
body is not in that shape, the raw body is used, truncated to 200
characters.

### HederaClient owns the HTTP client

`HederaClient` now creates one `http.Client` lazily, shares it with the
Mirror Node client and the node list fetch, and closes it in `close()`.
A client passed to `setHttpClient()` is not closed by the SDK, since the
caller owns it. `HederaClient.mirrorNode` returns a `MirrorNodeClient`
bound to the network's `mirrorNodeBaseUrl` and the client retry policy.
The node list fetch stays on the plain `http.Client` instead of
`MirrorNodeClient`, to avoid a slower fallback path when the node list
is needed to send a transaction.

### Endpoints

| Query | Endpoint |
|---|---|
| `MirrorNodeAccountBalanceQuery` | `GET /api/v1/balances?account.id={id}` |
| `MirrorNodeTokenBalanceQuery` | `GET /api/v1/accounts/{id}/tokens` (`token.id` when a token is set) |
| `MirrorNodeTokenInfoQuery` | `GET /api/v1/tokens/{tokenId}` |
| `MirrorNodeTokenNftInfoQuery` | `GET /api/v1/tokens/{tokenId}/nfts/{serial}` |

HBAR comes from `/balances`: without a `timestamp` parameter it returns
the latest balance. Token balances come from `/accounts/{id}/tokens`
and not from `/balances`, because the OpenAPI definition limits the
token list of `/balances` to 50 tokens and recommends against using it
for tokens. The token relationship endpoint also carries what the
balances endpoint lacks: KYC status, freeze status and whether the
association was automatic.

### Error mapping

An account, token or NFT that does not exist resolves to a
`HederaStatusException` with the matching official status, so existing
error handling keeps working:

| Unknown | Status | Code |
|---|---|---|
| Account | `INVALID_ACCOUNT_ID` | 15 |
| Token | `INVALID_TOKEN_ID` | 167 |
| NFT (token or serial) | `INVALID_NFT_ID` | 226 |

The first two are named constants. `INVALID_NFT_ID` is resolved through
`HederaStatusCode.fromCode(226)`, using the protobuf enum fallback added
in v0.2.5-dev (verified in `response_code.proto`). Every other HTTP
failure, and any response with an unexpected shape, is a
`MirrorNodeException`; the models throw `FormatException` internally and
the queries convert it to `MirrorNodeException` with the message
`Unexpected response: ...`.

For the HBAR balance, a response without a matching entry is also mapped
to `INVALID_ACCOUNT_ID`. That is an inference from the response shape,
covered by unit tests but not exercised against a real unknown account.

A token balance query that finds no relationship returns an empty list,
which means the account is not associated with the token. That is not an
error, and the live example checks it.

### Models

- Identifiers use the SDK types (`TokenId`, `AccountId`, `NftId`).
- The Mirror Node returns decimals and supplies as strings; the models
  expose them as `int`.
- Timestamps stay in the Mirror Node format (`seconds.nanoseconds`) as
  `String`, except `expiryTimestamp`, which the API gives in nanoseconds
  and is an `int`.
- `metadata` is base64 in JSON and is decoded to bytes.
- Keys are `MirrorNodeKey` with a `MirrorNodeKeyType`
  (`ECDSA_SECP256K1`, `ED25519`, `ProtobufEncoded`), and absent keys are
  `null`.
- `KycStatus`, `FreezeStatus` and `PauseStatus` map the API values
  (`GRANTED`, `REVOKED`, `NOT_APPLICABLE`; `FROZEN`, `UNFROZEN`,
  `NOT_APPLICABLE`; `PAUSED`, `UNPAUSED`, `NOT_APPLICABLE`) and have an
  `unknown` fallback, so a value added by a future Mirror Node release
  does not break parsing. Each has an `apiValue`.

### The NFT spender field

The OpenAPI schema for an NFT names the field `spender`, while its
example uses `spender_id`. The parser accepts both. In the live run
`spender` was `null` because no allowance exists, so the real field name
was not confirmed. Confirming it needs an NFT allowance, which the SDK
cannot create yet.

### Eventual consistency

The Mirror Node ingests records a few seconds after consensus, as the
official documentation states, so a read right after a transaction can
fail or return the previous state. The SDK does not retry on stale data,
since it cannot know what the caller expects. The live example shows the
pattern: a small helper retries a read until the result is what the
example expects, with a limit.

### AccountBalanceQuery deprecated, not removed

`AccountBalanceQuery` is marked `@Deprecated` with a message pointing to
the Mirror Node queries, and so is its constructor (the
`deprecated_consistency` lint requires it). Its `toBytes()` is unchanged
so the existing serialization tests keep their meaning. Removing it
would be a breaking change in a `-dev` line where it never executed
against the network; deprecation tells users the truth without breaking
them. The unit test file for the class uses a documented
`ignore_for_file`, since using the deprecated class there is intentional.
The `Query` documentation now says that consensus execution is not
implemented and lists the Mirror Node alternatives, and
`example/phase2/query_example.dart` reads a real balance instead of
describing a flow that cannot run.

## Verified

Live on Hedera testnet: created a treasury, a fungible token with a KYC
key and a freeze key, and an NFT collection with two NFTs; created and
associated a second account, granted KYC and moved tokens and one NFT.
The four queries returned the expected data:

- The HBAR balance of the new account was visible immediately.
- The token balance showed 1000 units (10.00 at 2 decimals), KYC
  `GRANTED` and freeze `UNFROZEN`; the NFT collection showed
  `NOT_APPLICABLE` for both.
- Token information returned name, symbol, type, decimals, supply,
  treasury, memo, pause status and the presence of each key.
- Each NFT returned its owner (the receiver for the transferred serial
  and the treasury for the other) and its metadata.
- An account that is not associated returned an empty list.
- An unknown token resolved to `INVALID_TOKEN_ID` and an unknown NFT
  serial to `INVALID_NFT_ID`.

The Mirror Node reflected the changes about 2 seconds after consensus;
only one read needed a retry.

## Tests

173 new unit tests (777 to 950 total), covering `MirrorNodeClient`
(pagination, the retry rules for 5xx, 429, timeouts and network errors,
non-retry of other 4xx, error body parsing), the `HederaClient` HTTP
client ownership, the four queries (requests, parameters, error mapping,
empty results), the models and enums (including the `unknown` fallback
and the `spender` / `spender_id` case) and `MirrorNodeException`.

## Out of Scope for v0.2.6-dev

- Consensus query execution (gRPC and the signed query payment),
  `TokenInfoQuery`, `TokenNftInfoQuery` and `AccountInfoQuery`
  execution; planned as v0.3.0-dev.
- The optional `timestamp` parameter for historical reads.
- Custom fees as input when creating a token; planned as v0.2.7-dev.
- NFT allowances, needed to confirm the `spender` field name.
- Regrouping the token classes by theme; planned as v0.2.8-dev.
- WebSocket subscriptions on the Mirror Node client.

## Related

- [v0.2.5-dev: KYC and Freeze](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/docs-sdk/phase-3/v0.2.5-dev/README.md)
