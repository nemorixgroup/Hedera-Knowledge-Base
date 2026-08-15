# Phase 1: Architecture + Protobuf

Status: ✅ Done (v0.0.1-dev to v0.0.3-dev)  
Files:
  - [Protobuf](https://github.com/nemorixgroup/hedera-flutter-sdk/tree/main/lib/src/proto)
  - [Hedera Network](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/client/hedera_network.dart)
  - [Hbar.dart](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/models/hbar.dart)
  - [Transaction ID](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/models/transaction_id.dart)
  - [Hedera Status Exception](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/core/hedera_status_exception.dart)

## Summary

Phase 1 established the foundational architecture of
hedera_flutter_sdk: a pure Dart SDK with no platform channels,
built directly on the official Hedera HAPI Protobuf definitions.

## Key Decisions

- **335 Dart classes generated from 104 Hedera HAPI `.proto` files.**
  Rather than hand-writing serialization logic, the SDK generates
  its Protobuf layer directly from Hedera's official protocol
  definitions, keeping the wire format authoritative and easy to
  regenerate as HAPI evolves.
- **`HederaStatusException` and `HederaStatusCode`** provide typed
  error handling over the raw numeric response codes returned by
  the network (`response_code.proto`), instead of exposing integers
  directly to SDK consumers.
- **`HederaConstants`** centralizes protocol-level values: ports,
  default fees, endpoints, and other network parameters, so they
  are defined once and referenced everywhere instead of scattered
  as magic numbers.
- **Pana score raised from 50/160 to 130/160** by fixing Protobuf
  generation script gaps (missing `auxiliary/` and `state/`
  subdirectories), declaring explicit platform support (excluding
  Web, due to the `grpc` package's `dart:io` dependency), and
  excluding `lib/src/proto/**` from static analysis, since it is
  generated code.

## Verified

- 60 unit tests passing at the close of Phase 1.
- GitHub Actions CI/CD pipeline and `pre_commit.ps1` local
  verification script established, used in every version since.

## Related

- [Phase 2: Crypto + Accounts](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/docs-sdk/phase-2/README.md)
