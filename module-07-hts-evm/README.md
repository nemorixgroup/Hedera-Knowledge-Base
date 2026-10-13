# Module 07 - HTS x EVM: Hybrid Tokenization

> Create, mint, and burn native Hedera NFTs from a Solidity smart contract by calling the Hedera Token Service (HTS) system contract, and compare that flow with the native SDK path.

---

## Table of Contents

1. [The Three Tokenization Models](#1-the-three-tokenization-models)
2. [The HTS System Contract](#2-the-hts-system-contract)
3. [Tutorial: NFT Collection with Royalty, Mint, and Burn](#3-tutorial-nft-collection-with-royalty-mint-and-burn)
4. [The ERC-721 Facade (HIP-218 and HIP-376)](#4-the-erc-721-facade-hip-218-and-hip-376)
5. [Native SDK vs Hybrid Contract](#5-native-sdk-vs-hybrid-contract)
6. [Hedera-Specific Considerations](#6-hedera-specific-considerations)
7. [Official Sources](#7-official-sources)

---

## 1. The Three Tokenization Models

Hedera supports three tokenization approaches:

| Feature | HTS Native | ERC/EVM | Hybrid (HTS + EVM) |
|---|---|---|---|
| Smart contracts required | No | Yes | Optional |
| Cost | Low, fixed fees | Higher gas fees | Lower than full EVM, uses HTS efficiencies |
| Compliance | Built-in (KYC, freeze, pause) | Must be coded in Solidity | HTS tokens act as ERC-20/ERC-721 |
| Flexibility | Limited customization | Highly customizable | Combines both |
| Typical use cases | Enterprise, regulated assets | DeFi, dApps, ERC-3643 | Interoperability, cost-effective DeFi |

In the **hybrid model**, HTS handles the core token operations (mint, burn, transfer) at low cost, while the smart contract adds custom logic such as governance or multi-signature approvals.

---

## 2. The HTS System Contract

System contracts expose Hedera API functionality at reserved EVM addresses. When a deployed contract calls one of these addresses, it executes as though a contract existed there.

| System contract | Address |
|---|---|
| Hedera Token Service | `0x167` |
| Exchange Rate | `0x168` |
| PRNG | `0x169` |
| Hedera Account Service | `0x16a` |
| Hedera Schedule Service | `0x16b` |

Key points about the HTS system contract (`0x167`):

- Tokens created through it are real HTS tokens and can also be managed with the native HTS APIs and SDKs.
- Every function returns a `responseCode`; `SUCCESS` is `22`.
- The Solidity interfaces are maintained in the [`hiero-contracts`](https://github.com/hiero-ledger/hiero-contracts) repository.
- Related HIPs: HIP-206, HIP-358, HIP-376, HIP-514, HIP-719.

---

## 3. Tutorial: NFT Collection with Royalty, Mint, and Burn

This section follows the official **HTS x EVM - Part 1** tutorial. The complete, maintained code lives in the official [hedera-code-snippets repository](https://github.com/hedera-dev/hedera-code-snippets/tree/main/hts-evm-mint-nfts).

### Prerequisites

- An **ECDSA** account from the [Hedera Portal](https://portal.hedera.com/)
- Basic Solidity knowledge

### Tooling

| Item | Value |
|---|---|
| Framework | Hardhat 3 (TypeScript, Mocha, Ethers.js) |
| Solidity | 0.8.28 |
| Dependencies | `@openzeppelin/contracts`, `@hiero-ledger/hiero-contracts` |
| RPC URL (Hedera testnet) | `https://testnet.hashio.io/api` |

```bash
mkdir hts-evm-mint-nfts && cd hts-evm-mint-nfts
npx hardhat --init
npx hardhat keystore set HEDERA_RPC_URL
npx hardhat keystore set HEDERA_PRIVATE_KEY
npm install @openzeppelin/contracts
npm install @hiero-ledger/hiero-contracts
```

> Hardhat 3 renamed `compile` to `build` and uses `npx hardhat --init` to bootstrap a project.

### The contract: `MyHTSToken.sol`

The contract inherits from `HederaTokenService`, `KeyHelper`, and `Ownable`.

**a) Create the collection** (`createNFTCollection`, owner only, callable once):

- The **treasury** is the contract itself.
- The **SUPPLY** and **ADMIN** keys are set to the contract ID, so minting, burning, and admin updates can only happen through the contract.
- A **10% royalty** with a **1 HBAR fallback fee** is paid to the owner.

```solidity
keys[0] = getSingleKey(KeyType.SUPPLY, KeyValueType.CONTRACT_ID, address(this));
keys[1] = getSingleKey(KeyType.ADMIN, KeyValueType.CONTRACT_ID, address(this));

(int rc, address created) = createNonFungibleTokenWithCustomFees(
    token,
    fixedFees,
    royaltyFees
);
require(rc == HederaResponseCodes.SUCCESS, "HTS: create NFT failed");
```

**b) Mint** (`mintNFT`):

1. Mint the serial to the treasury with `mintToken(tokenAddress, 0, metadata)`.
2. Transfer it to the recipient through the ERC-721 facade with `IERC721(tokenAddress).transferFrom(...)`.

**c) Burn** (`burnNFT`):

1. Check the caller is the owner or an approved operator.
2. If the NFT is not in the treasury, pull it back (the contract must be approved).
3. Burn it with `burnToken(tokenAddress, 0, serials)`.

**d) HBAR handling:** `receive()`, `fallback()`, and an owner-only `withdrawHBAR()`.

### The scripts

| Script | What it does |
|---|---|
| `deploy.ts` | Deploys the contract and calls `createNFTCollection()` sending **15 HBAR** |
| `mintNFT.ts` | Associates the signer by calling `associate()` on the token address, then mints with IPFS metadata (max 100 bytes) and checks `balanceOf` |
| `burnNFT.ts` | Approves the contract for the `tokenId`, calls `burnNFT()`, and checks the balance drops to 0 |

```bash
npx hardhat build
npx hardhat run scripts/deploy.ts --network testnet
npx hardhat run scripts/mintNFT.ts --network testnet
npx hardhat run scripts/burnNFT.ts --network testnet
```

> For most HTS system contract calls, gas covers the cost. Expensive operations such as creating an NFT collection are paid with the HBAR value sent in the call.

### Tests (optional)

```bash
npx hardhat test solidity   # local Solidity tests
npx hardhat test mocha      # runs against Hedera testnet
```

System contracts are not available on the local Hardhat network, so integration tests must run against Hedera testnet.

---

## 4. The ERC-721 Facade (HIP-218 and HIP-376)

A **facade contract** is a built-in adapter that lets an HTS token behave like a standard ERC-20 or ERC-721 token. EVM tools can call `transfer()`, `approve()`, or `transferFrom()` on the token address, and the call is delegated to the native Token Service.

| Concept | Detail |
|---|---|
| Token address | The HTS token exposed as an EVM address (for example `0x000...0068D3eF`) |
| Supported interfaces | ERC-20 for fungible tokens, ERC-721 for NFTs |
| Event logs | HTS tokens do not emit native logs; Mirror Nodes generate **synthetic events** for `CryptoTransfer`, `TokenMint`, `TokenBurn`, and others |

---

## 5. Native SDK vs Hybrid Contract

According to the official documentation, NFTs on Hedera can be created, minted, burned, and transferred **without deploying any smart contract**, using only HTS and an SDK.

The same NFT lifecycle with [`hedera_flutter_sdk`](https://pub.dev/packages/hedera_flutter_sdk) (Dart/Flutter):

```dart
// Create the NFT collection
final collectionTx = TokenCreateTransaction()
    .setTokenName('Demo Art')
    .setTokenSymbol('DART')
    .setTokenType(TokenType.NON_FUNGIBLE_UNIQUE)
    .setTreasuryAccountId(treasuryAccountId)
    .setSupplyKey(supplyPublicKey)
    .setMaxTransactionFee(Hbar(30));

// Mint 3 NFTs (one per metadata entry)
final mintTx = TokenMintTransaction()
    .setTokenId(tokenId)
    .addMetadata(_metadata('ipfs://bafyDemoArt1'))
    .addMetadata(_metadata('ipfs://bafyDemoArt2'))
    .addMetadata(_metadata('ipfs://bafyDemoArt3'))
    .setMaxTransactionFee(Hbar(20));

// Transfer serial 1 to Alice (Alice must be associated first)
final transferTx = CryptoTransferTransaction()
    .addNftTransfer(NftId(tokenId, 1), treasuryAccountId, aliceAccountId);

// Burn a serial (must be in the treasury)
final burnTx = TokenBurnTransaction()
    .setTokenId(tokenId)
    .addSerial(burnSerial)
    .setMaxTransactionFee(Hbar(20));
```

The full runnable example is [`example/phase3/token_nft_example.dart`](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/example/phase3/token_nft_example.dart).

### Side-by-side comparison

| Operation | Native SDK (`hedera_flutter_sdk`) | Hybrid contract (`0x167`) |
|---|---|---|
| Create collection | `TokenCreateTransaction` with `TokenType.NON_FUNGIBLE_UNIQUE` | `createNonFungibleTokenWithCustomFees()` |
| Mint | `TokenMintTransaction.addMetadata()` | `mintToken()` |
| Burn | `TokenBurnTransaction.addSerial()` | `burnToken()` |
| Associate | `TokenAssociateTransaction` | `associate()` on the token address |
| Transfer | `CryptoTransferTransaction.addNftTransfer()` | `IERC721.transferFrom()` (facade) |
| Supply key holder | An account (private key) | The contract (`CONTRACT_ID`) |
| Custom logic | No | Yes (Solidity rules) |

### When to use each

- **Native SDK:** you only need to issue and move NFTs; simpler and cheaper.
- **Hybrid contract:** you need programmable rules (who can mint, burn conditions, governance) on top of native HTS performance.

---

## 6. Hedera-Specific Considerations

1. **Token association is required.** The receiver must be associated with the token (or have auto-association slots) before receiving it; otherwise the transfer reverts.
2. **NFT metadata is limited to 100 bytes** per serial.
3. **Token IDs are HTS serial numbers**, starting at 1.
4. **Expensive operations are paid with HBAR sent in the call**, such as collection creation (15 HBAR in the tutorial).
5. **ECDSA accounts** are required on the EVM side.
6. **NFTs can only be burned from the treasury**, which is why the contract pulls the NFT back before burning.
7. **System contracts do not exist on the local Hardhat network**; integration tests run on Hedera testnet.

---

## 7. Official Sources

| Source | Link |
|---|---|
| HTS x EVM - Part 1: How to Mint NFTs | [docs.hedera.com](https://docs.hedera.com/evm/tutorials/hedera/hts-evm/part1-mint-nfts) |
| Tokenization on Hedera | [docs.hedera.com](https://docs.hedera.com/learn/core-concepts/tokens) |
| System Smart Contracts | [docs.hedera.com](https://docs.hedera.com/evm/hedera-services/system-contracts) |
| ERC/EVM-Compatible Tokenization | [docs.hedera.com](https://docs.hedera.com/evm/hedera-services/hybrid/erc-compatibility) |
| Official code repository | [hedera-code-snippets](https://github.com/hedera-dev/hedera-code-snippets/tree/main/hts-evm-mint-nfts) |
| Solidity interfaces | [hiero-contracts](https://github.com/hiero-ledger/hiero-contracts) |

> **Next in this series:** Part 2 (KYC and token update) and Part 3 (pause, freeze, wipe, and delete) may be added in future updates.
