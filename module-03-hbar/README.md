# Module 03 - HBAR: The Native Token
 
> HBAR is the native cryptocurrency of the Hedera network. It serves two inseparable roles: it is the fuel that powers every operation on the network, and the shield that protects the network's security through proof-of-stake consensus. This module covers HBAR's denominations, dual roles, staking mechanics, tokenomics, supply distribution, and its distinction from tokens built on top of Hedera.
 
## Table of Contents
 
- [What is HBAR?](#what-is-hbar)
- [Symbol and Denominations](#symbol-and-denominations)
- [The Two Roles of HBAR](#the-two-roles-of-hbar)
- [Transaction Fees: Predictable and USD-Denominated](#transaction-fees-predictable-and-usd-denominated)
- [How Fee Distribution Works](#how-fee-distribution-works)
- [Staking: Security Without Lock-up](#staking-security-without-lock-up)
- [Tokenomics: Fixed Supply of 50 Billion](#tokenomics-fixed-supply-of-50-billion)
- [Supply Allocation Categories](#supply-allocation-categories)
- [Released vs Unreleased Supply](#released-vs-unreleased-supply)
- [Why a Gradual Release Schedule?](#why-a-gradual-release-schedule)
- [HBAR vs Tokens on Hedera (HTS)](#hbar-vs-tokens-on-hedera-hts)
- [Wrapped HBAR (WHBAR)](#wrapped-hbar-whbar)
- [HBAR as a Digital Commodity (2026)](#hbar-as-a-digital-commodity-2026)
- [Key Takeaways](#key-takeaways)
- [References](#references)
 
## What is HBAR?
 
HBAR is the native, energy-efficient cryptocurrency of the public Hedera network. HBARs are used to pay application transaction fees and to protect the network from attack through proof-of-stake.
 
Unlike most Layer 1 tokens where the native coin is primarily a speculative asset, HBAR has deeply embedded utility baked into the protocol itself. Without HBAR, no transaction can be submitted, no consensus node can earn compensation, and no staking weight can be applied to the network's security model. The token and the network are architecturally inseparable.
 
## Symbol and Denominations
 
The official HBAR currency symbol is **ℏ** - the reduced Planck constant from quantum physics. This is a deliberate nod to the scientific rigor behind the project. The corporate logo form is **Ħ** (uppercase).
 
The written form is **hbars** (lowercase), equivalent to writing "US dollars" instead of "USD".
 
HBAR is divisible into the following denominations:
 
| Name | Symbol | Equivalence |
|------|--------|-------------|
| gigabar | Gℏ | 1 Gℏ = 1,000,000,000 ℏ |
| megabar | Mℏ | 1 Mℏ = 1,000,000 ℏ |
| kilobar | kℏ | 1 kℏ = 1,000 ℏ |
| hbar | ℏ | 1 ℏ = 1 ℏ |
| millibar | mℏ | 1,000 mℏ = 1 ℏ |
| microbar | μℏ | 1,000,000 μℏ = 1 ℏ |
| tinybar | tℏ | 100,000,000 tℏ = 1 ℏ |
 
The **tinybar** is the smallest indivisible unit of HBAR. All amounts in the Hedera SDKs and APIs are expressed internally in tinybars - similar to how Bitcoin uses satoshis.
 
```dart
// All Hedera SDK amounts are expressed in tinybars
// 1 HBAR = 100,000,000 tinybars
 
final oneHbar         = Hbar.fromTinybars(100000000); // 1 ℏ
final oneTinybar      = Hbar.fromTinybars(1);          // 0.00000001 ℏ
final transactionFee  = Hbar.fromTinybars(10000);      // ~$0.0001 USD
```
 
## The Two Roles of HBAR
 
HBAR serves two fundamental and inseparable roles in the Hedera network:
 
```
┌─────────────────────────────────────────────┐
│                    HBAR                     │
│                                             │
│   Role 1: FUEL          Role 2: SHIELD      │
│   Transaction fees  +   Proof-of-Stake      │
│   Pay for compute,      Secure the network  │
│   bandwidth, storage    against attacks     │
└─────────────────────────────────────────────┘
```
 
Neither role can exist without the other. The same token that pays for a token transfer also secures the network that processes it.
 
## Transaction Fees: Predictable and USD-Denominated
 
Every operation on Hedera requires payment in HBAR. This includes:
 
- Transferring HBAR between accounts
- Minting fungible or non-fungible tokens (HTS)
- Calling smart contracts (HSCS)
- Logging messages to the consensus service (HCS)
- Storing files (HFS)
- Creating or updating accounts
**What makes Hedera's fee model unique:**
 
Hedera avoids gas-based pricing. Fees are predictable, fixed in USD, and converted to HBAR at the time of transaction. There are no surprises, just simple and reliable costs for every operation.
 
This is a fundamental difference from Ethereum and similar networks where:
 
| | Ethereum | Hedera |
|--|---------|--------|
| Fee denomination | Gas (ETH) | Fixed USD, paid in HBAR |
| Fee variability | Can spike 10x-100x during congestion | Constant and predictable |
| Minimum fee (transfer) | ~$0.50-$5.00 (varies heavily) | ~$0.0001 USD |
| Developer predictability | Low - gas estimation required | High - fees are known in advance |
| MEV risk | High (miners reorder for profit) | None (fair ordering guaranteed) |
 
A fee of **~$0.0001 USD** per transaction makes Hedera one of the only networks where micropayments, IoT data logging, and high-frequency financial applications are economically viable.
 
## How Fee Distribution Works
 
When a user submits a transaction, the fee paid in HBAR is automatically split among three destinations:
 
```
Transaction Fee (in HBAR)
         │
         ├──> Node Fee        → The specific node that received and submitted the transaction
         │                      (compensates for bandwidth, compute, storage)
         │
         ├──> Network Fee     → Hedera Treasury (account 0.0.98)
         │                      (supports network operations and governance)
         │
         └──> Service Fee     → Staking Rewards account (0.0.800)
                                 (distributed to accounts staking HBAR)
```
 
Once per day, at the end of each staking period, a single synthetic transaction distributes the accumulated fees from account 0.0.802 to their appropriate destinations, including the Staking Rewards account (0.0.800).
 
Node operators receive HBAR as direct compensation for operating infrastructure. They can exchange earned HBAR for other currencies or fiat through exchanges.
 
## Staking: Security Without Lock-up
 
Staking is the mechanism by which HBAR holders contribute to the network's security while earning rewards.
 
### How staking works
 
Every node on Hedera's proof-of-stake network has a weighted influence on consensus - that weight is proportional to the number of HBARs staked to a node. Transactions achieve consensus after being processed by nodes representing an aggregate stake of over two-thirds of the network's total HBAR supply.
 
This directly implements the aBFT security guarantee covered in Module 02: a malicious actor would need to control and stake more than 1/3 of the total HBAR supply to compromise network consensus - making such an attack economically unfeasible.
 
### What makes Hedera staking different
 
Hedera's staking model is notably more accessible and lower-risk than most proof-of-stake networks:
 
| Feature | Hedera | Ethereum | Typical PoS |
|---------|--------|----------|-------------|
| Lock-up period | None - fully liquid at all times | 27+ hours unbonding | Days to weeks |
| Slashing risk | None | Yes (up to 50% of stake) | Yes |
| Minimum amount | No minimum | 32 ETH minimum (to be validator) | Varies |
| Change node | Anytime | Complex | Varies |
| Amount to stake | Entire account balance (automatic) | Must specify amount | Must specify |
| APY (approximate) | Up to 2.5% | ~3-4% | Varies |
 
The absence of slashing and lock-up periods is a deliberate design decision. Technical details are explained in the official Hedera blog post: "Why is there no Slashing in Hedera's Proof of Stake?"
 
### Indirect staking
 
Hedera supports a unique indirect staking model: if account A stakes to account B, and account B stakes to a node, the stake from both A and B increases the node's consensus weight. However, rewards for both accounts are paid to account B. An account can also optionally decline to earn rewards while still contributing its balance to the node's stake.
 
## Tokenomics: Fixed Supply of 50 Billion
 
The Hedera network was launched with a total supply of **50,000,000,000 HBAR**. The total supply of HBAR may not be modified without the unanimous consent of the members of the Hedera Council (LLCA § 8.4).
 
This is a hard cap encoded in the legal operating agreement of the governing entity - not just in code. There is no inflation mechanism, no additional minting possible, and no governance vote that could change this without unanimous Council agreement.
 
**Comparison with other major networks:**
 
| Network | Supply Model | Notes |
|---------|-------------|-------|
| Bitcoin | Fixed at 21M BTC | Hard cap in code |
| Ethereum | No hard cap, issuance continues | EIP-1559 burns some ETH |
| XRP | Fixed at 100B XRP | But large portion held by Ripple |
| Stellar | Fixed at ~50B XLM | Inflation mechanism removed in 2019 |
| **HBAR** | **Fixed at 50B HBAR** | **Hard cap in legal agreement + code** |
 
## Supply Allocation Categories
 
As of the Treasury Management Report updated June 2026, all 50 billion HBAR have been allocated across four categories:
 
| Allocation Category | HBAR Allocated | % of Total |
|--------------------|---------------|-----------|
| Initial Development Costs and Licensing | 3,882,948,559 | 7.77% |
| Purchase Agreements (SAFTs / TPAs) | 12,698,348,449 | 25.40% |
| Network Governance and Operations | 8,116,201,648 | 16.23% |
| Ecosystem and Open Source Development | 25,302,501,344 | 50.61% |
| **TOTAL** | **50,000,000,000** | **100%** |
 
### Category descriptions
 
**Initial Development Costs and Licensing (7.77%):** HBAR allocated to license and deploy the Hashgraph technology originally engineered and patented by Swirlds, Inc. All IP was purchased and open-sourced in 2022, ending the need for further allocations in this category.
 
**Purchase Agreements (25.40%):** All HBAR sold to users under regulated sales contracts - Simple Agreements for Future Tokens (SAFTs) sold before network launch, and Token Purchase Agreements (TPAs) sold after launch to support ongoing operations.
 
**Network Governance and Operations (16.23%):** Compensation to founders, executives, employees, and contractors. Includes the original Coin Plan and current Council Operations structure.
 
**Ecosystem and Open Source Development (50.61%):** The largest category by far - HBAR released to the community for furthering decentralization and development. Includes community incentive programs, developer grants, and the Hedera Foundation's ecosystem development mandate.
 
The fact that over **half** of the entire HBAR supply is designated for ecosystem and open-source development reflects Hedera's strategy to grow the developer community rather than concentrate tokens with insiders.
 
## Released vs Unreleased Supply
 
All HBAR are initially considered **Unreleased Supply** until they are transferred to a User Account - an account operationally controlled by anyone other than the Hedera Council.
 
```
Total Supply: 50,000,000,000 HBAR
│
├── Unreleased Supply   → Still in Hedera Council treasury accounts
│   ├── Unallocated     → No specific purpose assigned yet
│   └── Allocated       → Purpose assigned but not yet distributed
│
└── Released Supply     → Transferred to user accounts
                          (one possible measure of circulating supply)
```
 
Hedera does not use or define the term "Circulating Supply" officially, as different market data platforms apply different definitions. For the most current and accurate data, the Treasury Management Report at https://hedera.com/treasury-management-report is the authoritative source, updated quarterly.
 
## Why a Gradual Release Schedule?
 
The gradual release of HBAR from the treasury serves a specific security purpose:
 
Hedera's coin release schedule was designed to maintain the security of the network and correlate with expected activity on the network. With a mission of being a 100-year company, Hedera is focused on the long-term well-being and growth of the network.
 
The reasoning is mathematical: if an attacker needed to compromise the aBFT guarantee by controlling >1/3 of staked HBAR, a smaller circulating supply makes that attack proportionally more expensive. As the network grows in activity and value, more supply is gradually released - always keeping the cost of an attack economically unfeasible relative to the value of the network.
 
## HBAR vs Tokens on Hedera (HTS)
 
A critical distinction for developers:
 
| Property | HBAR | HTS Tokens |
|----------|------|------------|
| Layer | Native Layer 1 coin | Tokens created on Hedera |
| Primary purpose | Fees + PoS security | Represent any digital asset |
| Created by | Hedera Council at genesis | Any developer via HTS |
| Transfer fee | ~$0.0001 USD | ~$0.0001 USD |
| Throughput | 10,000+ TPS | 10,000+ TPS |
| Finality | 3-5 seconds | 3-5 seconds |
| Examples | HBAR itself | USDC on Hedera, custom stablecoins, NFTs, reward tokens |
| EVM compatible | Via WHBAR wrapper | Natively via HTS-ERC20 bridge |
 
HTS tokens use HBAR to pay for their own creation, minting, and transfer. So even application-layer tokens depend on HBAR as the underlying fee mechanism.
 
## Wrapped HBAR (WHBAR)
 
Wrapped HBAR (WHBAR) is an ERC-compatible wrapper that follows the ERC-20 standard for Hedera's native HBAR token. Built on widely adopted wrapper contract principles, WHBAR makes it easier for developers and users to integrate Hedera's native token into decentralized applications (dApps).
 
How it works:
 
```
Deposit:  User sends HBAR → Contract mints equivalent WHBAR (1:1 parity)
Withdraw: User calls withdraw(amount) → Contract burns WHBAR + releases HBAR
```
 
Each unit of HBAR (represented in tinybars with 8 decimals) is matched with one unit of WHBAR. WHBAR enables:
 
- Integration with DeFi protocols that expect ERC-20 tokens
- Compatibility with the broader Web3 ecosystem
- Use in Solidity smart contracts on Hedera that expect ERC-20 interfaces
For more details, see the official documentation: https://docs.hedera.com/hedera/core-concepts/smart-contracts/wrapped-hbar-whbar
 
## HBAR as a Digital Commodity (2026)
 
In 2026, both the SEC and CFTC classified HBAR as a **digital commodity** - not as a security. This regulatory classification has significant implications:
 
- HBAR is not subject to securities regulations in the U.S.
- Applications built on Hedera that use HBAR for payments or fees operate in a clearer regulatory environment
- Fintech projects integrating HBAR face reduced regulatory uncertainty compared to networks whose tokens remain unclassified or are classified as securities
This classification aligns with Hedera's long-standing design philosophy: HBAR is infrastructure fuel, not an investment contract.
 
## Key Takeaways
 
- HBAR is the **native coin** of Hedera with a **fixed supply of 50 billion** - no inflation possible
- The symbol is **ℏ** and the smallest unit is the **tinybar** (1 ℏ = 100,000,000 tℏ)
- HBAR has two inseparable roles: **fuel** (transaction fees) and **shield** (proof-of-stake security)
- Fees are **fixed in USD** (~$0.0001 per transaction) and paid in HBAR at the current exchange rate - no gas volatility
- Staking on Hedera is **liquid with no lock-up and no slashing risk** - a major differentiator from Ethereum
- Stakers earn up to **2.5% APY** as a share of transaction fees
- Over **50% of the total supply** is allocated to ecosystem and open-source development
- **Released Supply** is the operative measure of circulating HBAR - tracked in real-time via the Treasury Management Report and HashScan
- In 2026, HBAR was classified as a **digital commodity** by the SEC and CFTC

## References
 
| Resource | URL |
|----------|-----|
| HBAR Official Page | https://hedera.com/hbar |
| HBAR Denominations (Official Help) | https://help.hedera.com/hc/en-us/articles/360000674317 |
| Treasury Management Report (live, updated quarterly) | https://hedera.com/treasury-management-report |
| Hedera Staking Program (Docs) | https://docs.hedera.com/learn/core-concepts/staking/staking |
| Why No Slashing in Hedera PoS? (Official Blog) | https://hedera.com/blog/why-is-there-no-slashing-in-hederas-proof-of-stake |
| Native Staking Phase 1 Announcement | https://hedera.com/blog/introducing-native-staking-phase-1-on-the-hedera-network |
| Wrapped HBAR (WHBAR) Docs | https://docs.hedera.com/hedera/core-concepts/smart-contracts/wrapped-hbar-whbar |
| Hedera Fee Calculator | https://hedera.com/fee-calculator |
| HashScan Network Explorer | https://hashscan.io |
 
---
 
*Previous: [Module 02 - The Hashgraph Algorithm](https://github.com/nemorixgroup/hedera-knowledge-base/tree/main/module-02-hashgraph)*  
*Next: [Module 04 - Native Services](https://github.com/nemorixgroup/hedera-knowledge-base/tree/main/module-04-services)*
 
---
 
*Last updated: June 2026*
*Part of the [Hedera Knowledge Base](https://github.com/nemorixgroup/hedera-knowledge-base/)*
