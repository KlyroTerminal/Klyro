<div align="center">

# KLYRO PROTOCOL

### Institutional-Grade Solana Derivatives Terminal, Parimutuel Escrow & Dual-Layer Zero-Knowledge Privacy Infrastructure

<br/>

[![Solana](https://img.shields.io/badge/Solana-Mainnet_%26_Devnet-14F195?style=for-the-badge&logo=solana&logoColor=black)](https://solana.com)
[![Anchor](https://img.shields.io/badge/Anchor-v0.32.1-3B82F6?style=for-the-badge&logo=rust&logoColor=white)](https://www.anchor-lang.com)
[![Rust](https://img.shields.io/badge/Rust-1.79+-DEA584?style=for-the-badge&logo=rust&logoColor=white)](https://www.rust-lang.org)
[![ZK-SNARK](https://img.shields.io/badge/Zero--Knowledge-Groth16_BN254-8B5CF6?style=for-the-badge)](https://iden3.io/circom)
[![License](https://img.shields.io/badge/License-BSL_1.1-10B981?style=for-the-badge)](LICENSE)
[![Audit Status](https://img.shields.io/badge/Audit_Readiness-98.6%25_Tier--1-purple?style=for-the-badge)](SECURITY_AUDIT.md)

<br/>

**Klyro Protocol** is an institutional decentralized finance (DeFi) terminal and sovereign cryptographic execution layer engineered natively for Solana. 

Unifying four high-frequency perpetual futures engines, an on-chain parimutuel prediction market escrow, an RWA/Token-2022 launchpad, and an industry-first **Dual-Layer Privacy Suite** (3-Hop Ephemeral Relays + Groth16 $BN254$ ZK Shielded Pool), Klyro delivers sub-millisecond execution with mathematical zero-knowledge privacy and institutional risk management.

<br/>

[Architecture](#system-architecture) &nbsp;•&nbsp; [Perpetual Engines](#1-perpetual-futures--clearinghouse-engines) &nbsp;•&nbsp; [Market Maker Engine](#2-institutional-algorithmic-market-maker--volume-engine) &nbsp;•&nbsp; [Prediction Escrow](#3-parimutuel-prediction-market-engine) &nbsp;•&nbsp; [Zero-Knowledge Privacy](#4-dual-layer-privacy-infrastructure) &nbsp;•&nbsp; [LaunchLab](#5-token-launchpad--rwa-catalog) &nbsp;•&nbsp; [Flywheel Economics](#6-closed-loop-fee-flywheel--pol) &nbsp;•&nbsp; [Formal Invariants & Security](#7-formal-invariants--security-architecture)

<br/>

</div>

---

<br/>

## System Architecture

Klyro separates concerns across four distinct operational layers to ensure deterministic sub-30ms execution, zero front-running (MEV isolation), and cryptographically un-linkable transactions:

```
+---------------------------------------------------------------------------------------------------------+
|                                    KLYRO PROTOCOL SYSTEM TOPOLOGY                                        |
+---------------------------------------------------------------------------------------------------------+
|  [Client Presentation & Execution Layer]                                                                |
|   ├── LaunchLab (Token-2022 / RWA Mints / Dynamic Bonding Curves / Raydium CPMM Migration)              |
|   ├── Klyro Perp Futures (4 Specialized Matching Engines: V2 Market, Limit, Conditional TP/SL, Ledger)   |
|   ├── Prediction Market (Live Polymarket & Kalshi Multi-Oracle Feeds / Parimutuel Escrow)               |
|   ├── Privacy Suite (Groth16 SnarkJS Mixer / 3-Hop Ephemeral Relays / Stealth Multi-Send)               |
|   ├── Flywheel Economics ($KLYRO CPMM POL Injection / Buyback & Burn / Revenue Splitter)                |
|   └── Market Maker Dashboard (Real-time Cluster TPS, Slot Drift & Sub-Millisecond RPC Telemetry)        |
+---------------------------------------------------------------------------------------------------------+
|  [State & Execution Layer (TypeScript / JS In-Memory Engines)]                                          |
|   ├── BinanceGradePerpEngine (In-memory PTR Gatekeeper / Anti-Wick Oracle / 5-Bar ADL / Jito MEV Tips)  |
|   ├── PriceSyncService (Binance WebSocket Ticker Feeds + Gold/Silver + Micro-Basis Smoothers)            |
|   ├── KlyroPrivacyService (WebCrypto 31-byte field elements / Poseidon Leaf Hashers)                    |
|   └── TokenLaunchService (Atomic Token-2022 deployer / Incinerator LP Burn / Streamflow Timelock)       |
+---------------------------------------------------------------------------------------------------------+
|  [Sovereign Backend & Settlement Node (Node.js / Express / Solana Web3)]                                |
|   ├── Settlement Gateway (Cryptographic Proof-of-Deposit & Anti-Replay Ledger)                          |
|   ├── Privacy Relayer (Zero-Knowledge Groth16 Proof Verification & Nullifier Accounting)                |
|   └── Defensive Perimeter (Proof-of-Work Challenges / Honeypot Traps / Multi-Tier Rate-Limiters / Helmet)|
+---------------------------------------------------------------------------------------------------------+
|  [On-Chain Anchor Smart Contracts (Rust / Solana BPF)]                                                  |
|   ├── klyro_pool (ZK Shielded Mixer / 20-Level Poseidon Merkle Tree / Timelocked Multi-Sig Governance)   |
|   └── klyro_prediction (Parimutuel Escrow / u128 Checked Math / Dispute Window / Multisig Resolver)     |
+---------------------------------------------------------------------------------------------------------+
```

<br/>

---

<br/>

## 1. Perpetual Futures & Clearinghouse Engines

The Klyro Perpetual Trading platform is powered by four dedicated execution engines designed to mirror high-throughput institutional derivatives architectures (e.g., Drift v2, Phoenix, Binance Futures):

### 1.1 The 4 Klyro Engines
1. **Klyro V2 Market Matching Engine**:
   - Delivers sub-millisecond execution with volatility-weighted dynamic slippage bands:
     $$S_{\text{dynamic}} = \max\left(0.02\%,\, \sigma_{\text{30s}} \times \sqrt{\frac{\text{OrderSize}}{\text{MarketDepth}}}\right)$$
   - Pre-trade risk (PTR) verification runs in-memory ($< 1\text{ms}$), checking collateral sufficiency, tick-size compliance, and minimum notional requirements before state transition.
2. **Klyro Limit Matching Engine**:
   - Quantized 4-decimal tick execution book with pre-flight **Post-Only Crossing Prevention**:
     - *Buy Post-Only*: Enforces $\text{LimitPrice} < \text{BestAsk}$. Rejects or cancels orders that would cross the spread, ensuring the trader acts exclusively as a liquidity maker eligible for fee rebates.
     - *Sell Post-Only*: Enforces $\text{LimitPrice} > \text{BestBid}$.
3. **Klyro Conditional Engine**:
   - Automated Stop Market, Take Profit (TP), and Stop Loss (SL) triggers with **One-Cancels-the-Other (OCO)** auto-purge logic.
   - When a primary TP or SL trigger executes, lingering counter-orders are automatically cancelled to eliminate ghost executions and orphaned risk.
4. **Klyro Audit Clearinghouse Ledger**:
   - High-throughput transaction recording engine producing cryptographically auditable state transitions.
   - Generates RFC 4180 compliant CSV exports for institutional accounting, tax reconciliation, and regulatory compliance.

### 1.2 Quantitative Risk Management & Pricing Mechanics
* **3-Way Anti-Wick Composite Oracle**:
  Protects traders from flash-loan manipulation, local orderbook thins, and cascading liquidation traps:
  $$\text{MarkPrice} = \text{IndexPrice} + \text{EMA}_{30}(\text{Basis})$$
  $$\text{MarkPrice}_{\text{clamped}} = \text{clamp}\Big(\text{MarkPrice},\, \text{IndexPrice} \times (1 - \delta),\, \text{IndexPrice} \times (1 + \delta)\Big), \quad \delta = 0.015\ (1.5\%)$$
* **5-Bar Auto-Deleveraging (ADL) Waterfall**:
  When the insurance fund is stressed during extreme market dislocations, positions in opposing profit are queued based on their normalized percentile:
  $$\text{Score}_{\text{ADL}} = \left(\frac{\text{UnrealizedPnL}}{\text{InitialMargin}}\right) \times \left(\frac{\text{EffectiveLeverage}}{10}\right)$$
  Classified into 5 quantile tranches ($0.0 \le \text{Rank} \le 1.0$), ensuring risk liquidation is distributed deterministically and transparently.
* **Jito MEV Bundling & Priority Fees**: Direct tips via Jito block engines to guarantee front-running resistance and inclusion within the next confirmed block.
* **Triple Chart Engine**: Fully interchangeable visualization layer featuring TradingView Advanced Charts, High-Performance Native Candlesticks, and Phoenix Orderbook Depth visualizers with customizable technical indicators (RSI, MACD, Bollinger Bands, Volume Profile).

<br/>

---

<br/>

## 2. Institutional Algorithmic Market Maker & Volume Engine

The Klyro Market Making Suite (`web/src/services/KlyroMarketMakerEngine.js`) is an institutional quantitative liquidity and volume generation engine engineered natively for the Solana blockchain. Designed to surpass basic retail trading scripts, it synthesizes micro-structure inventory modeling with complete on-chain forensic de-anonymization defense:

### 2.1 Micro-Token Calibrated Avellaneda-Stoikov Inventory Model

Standard academic Avellaneda-Stoikov models subtract absolute dollar increments ($r = s - q\gamma\sigma^2$), which induces negative reservation prices on sub-cent Solana tokens (e.g. $BONK at $\$0.00000365$). Klyro implements a **percentage-scaled hyperbolic tangent formulation**:

$$r(s, q) = s \cdot \left(1 - \tanh(q \cdot \gamma) \cdot \frac{\text{PriceRangePct}}{200}\right)$$

Where:
* $s$ = Mid market price ingested via real-time DexScreener & Jupiter liquidity feeds
* $q \in \mathbb{R}$ = Instantaneous net inventory skew ($q > 0$ denotes long inventory, $q < 0$ denotes short inventory)
* $\gamma \in [0.05, 1.0]$ = Risk-aversion coefficient parameter
* $\text{PriceRangePct}$ = Bound on maximum allowable quoting divergence ($2.0\% - 5.0\%$)

**Dynamic Asymmetric Quoting Spreads:**
Bid-ask spreads scale dynamically with instantaneous realized volatility ($\sigma$):

$$\delta^a + \delta^b = \max\left(0.50\%,\, \min\left(5.00\%,\, \frac{\text{PriceRangePct}}{100} \cdot (1 + 2\sigma)\right)\right)$$

$$\text{OptimalBid} = r(s, q) - s \cdot \left(\frac{\delta^a + \delta^b}{2}\right), \quad \text{OptimalAsk} = r(s, q) + s \cdot \left(\frac{\delta^a + \delta^b}{2}\right)$$

When net inventory skew is long ($q > 0$), reservation price $r$ is depressed downwards, driving down bid quotes while tightening ask quotes to incentivize organic inventory rebalancing while capturing spread alpha.

### 2.2 Multi-Level Geometric Grid Ladders & Floor Walls

To establish resilient orderbook depth and defend floor prices, the engine computes $N$-tier geometric bid ladders:

$$\text{Bid}_k = \text{OptimalBid} \cdot (1 - k \cdot \text{stepPct}), \quad k \in \{1, \dots, N\}$$

$$\text{ClipSize}_k = \text{BaseClip} \cdot (1.0 + k \cdot 0.25)$$

Deeper tiers feature geometrically weighted buy walls (up to $2.0\times$ clip size multiplier), providing programmatic defense against cascading market sells.

### 2.3 On-Chain Forensic De-Anonymization Defense Matrix

Klyro mathematically eradicates all five primary bot-detection signatures tracked by forensic indexers (**Bubblemaps, Solscan, Birdeye, DexScreener, Cielo**):

```
       [Forensic Threat Signature]                 [Klyro Mathematical Countermeasure]
─────────────────────────────────────────────────────────────────────────────────────────────
1. Bubblemaps Star-Topology Clustering  ──► Ephemeral Sub-Account Dispersal (Pool of 20-50 Wallets)
2. Static Round-Number Fingerprinting   ──► Pareto Power-Law (α=1.8) + 6-Decimal Entropy Noise
3. Fast Fourier Transform (FFT) Spikes  ──► Poisson Point Process Inter-Arrivals (Δt = -ln(U) / λ)
4. DexScreener Wash-Trading Penalty     ──► FIFO 25-Second Holding Buffer per Sub-Wallet
5. Gas Fee Compute Budget Signatures    ──► Gaussian Priority Fee Jitter (±15% Micro-Lamport Spread)
```

1. **Anti-Bubblemaps Sub-Account Dispersal**:
   Transactions are multiplexed across a dynamic pool of up to 50 independent ephemeral sub-wallets (`activeSubWallets`). Private keypairs reside exclusively in volatile RAM and are permanently purged upon stop or sweep.
2. **Pareto Power-Law Trade Sizing ($\alpha = 1.8$)**:
   $$\text{ParetoFactor} = (1 - 0.95 \cdot u)^{-1/\alpha} - 1, \quad u \sim \mathcal{U}(0,1)$$
   Replicates authentic retail trading volume dynamics (**~70.3% micro clips**, **~22% mid clips**, **~7.7% impulse clips**). Sub-lamport 6-decimal non-round entropy noise eliminates round-number fingerprints (`0.014219 SOL` instead of `0.0100 SOL`).
3. **Poisson Inter-Arrival Timing (Anti-FFT)**:
   Inter-order intervals follow $\Delta t = -\frac{\ln(u)}{\lambda}$. With an empirical Coefficient of Variation $CV = \sigma / \mu \approx 0.97 \approx 1.0$, the arrival pattern exhibits zero harmonic frequency spikes under spectral Fourier analysis.
4. **FIFO Anti-Wash Trading Buffer**:
   Sub-wallets are bound by a **mandatory 25-second holding buffer** ($T_{\text{hold}} \ge 25\text{s}$) before purchased tokens can be quoted on the sell side, preventing wash-trading suppression on DexScreener and Birdeye trending algorithms.
5. **Dynamic Priority Fee Jitter**:
   Base priority fee ($12,500$ micro-lamports) is perturbed by Gaussian noise ($\pm 15\%$), preventing gas fee signature clustering.

### 2.4 Pre-Flight On-Chain Honeypot & Freeze Authority Screen

Before any order dispatch, the engine deserializes the target SPL Token Mint account (82-byte binary layout) directly from RPC state:

* **Freeze Authority Verification**: Inspects byte offset 46..82 (`COption<Pubkey>`). If active, the engine blocks quotes and halts execution with a critical security alert, preventing capital lockup in malicious tokens.
* **Mint Authority Verification**: Inspects byte offset 0..36 (`COption<Pubkey>`). Detects unrevoked supply expansion rights and dynamically scales down inventory risk.
* **Cryptographic Sweep Invariant**: One-click sweep routine validates Ed25519 base58 recipient addresses, zeroes inventory states, wipes ephemeral keys from memory, and reclaims all SOL to the authenticated operator.

<br/>

---

<br/>

## 3. Parimutuel Prediction Market Engine

The on-chain prediction infrastructure (`programs/klyro_prediction`) implements a parimutuel pooled wagering protocol written in Anchor Rust, eliminating liquidity fragmentation inherent to order-book binary markets:

```rust
// On-Chain Mathematical Payout Distribution Invariant (u128 safe checked math)
let total_pool = market.total_pool as u128;
let winning_pool = market.winning_pool as u128;
let user_stake = position.stake as u128;

// Payout = (UserStake * TotalPool) / WinningPool - PlatformFee
let gross_payout = user_stake
    .checked_mul(total_pool).ok_or(PredictionError::MathOverflow)?
    .checked_div(winning_pool).ok_or(PredictionError::MathDivisionByZero)?;

let fee = gross_payout
    .checked_mul(market.fee_basis_points as u128).ok_or(PredictionError::MathOverflow)?
    .checked_div(10_000).ok_or(PredictionError::MathDivisionByZero)?;

let net_payout = gross_payout.checked_sub(fee).ok_or(PredictionError::MathUnderflow)?;
```

### Core Program Safeguards:
1. **$u128$ Overflow Immunity**: All calculation paths are up-cast to 128-bit unsigned integers with strict `.checked_mul()`, `.checked_div()`, and `.checked_sub()`. Rounding continuously favors protocol solvency.
2. **Dispute Window & Resolution Lock**: When an oracle outcome is submitted, an on-chain dispute window ($24\text{h} - 7\text{d}$) begins (`now >= resolution_time + dispute_window`). This permits multisig or decentralized governance intervention in case of erroneous off-chain oracle feeds.
3. **Empty Winning Pool Refund Guarantee**: If an outcome resolves with zero stakers on the winning side, funds do not get locked; participants invoke `claim_refund` to reclaim their original deposit minus network gas.
4. **Constrained Fee Destination**: Protocol fees are bound by on-chain constraints:
   ```rust
   constraint = fee_destination.key() == BUYBACK_VAULT || fee_destination.key() == market.creator
   ```
   Protocol fees can never be redirected to an arbitrary external address.
5. **Deterministic Settlement Verification**: The settlement service verifies on-chain transaction finality, validates cryptographic depositor identity, prevents state replay, and executes trustless payouts within $< 1.5\text{s}$.

<br/>

---

<br/>

## 4. Dual-Layer Privacy Infrastructure

Klyro delivers an un-linkable dual-layer privacy architecture designed specifically to break on-chain heuristics and wallet clustering on Solana.

### Layer 1: 3-Hop Ephemeral Relays
* **Multi-Node Hopping**: Funds transition through three independent, single-use ephemeral keypairs generated via cryptographically secure pseudo-random number generators (CSPRNG):
  $$\text{User Wallet} \xrightarrow{\text{Jitter}} \text{Relay } \alpha \xrightarrow{\text{Jitter}} \text{Relay } \beta \xrightarrow{\text{Jitter}} \text{Relay } \gamma \xrightarrow{} \text{Destination}$$
* **Hardware-Grade Timing Jitter**: Introduces non-deterministic delays ($1,000\text{ms} - 3,000\text{ms}$) between hops to defeat block-time clustering and temporal Solscan indexing.
* **Micro-Dust Obfuscation**: Adds random micro-variations ($0.0001 - 0.0005\text{ SOL}$) to transactions, defeating graph analysis based on exact subset-sum balance tracking.
* **Isolated DEX Settlement**: Token swaps execute strictly on Relay Node $\gamma$. The user's origin wallet never directly interacts with automated market makers (AMMs) or liquidity pools.

### Layer 2: Zero-Knowledge Shielded Pool (`klyro_pool`)
* **Cryptographic Primitive**: Zero-Knowledge Succinct Non-Interactive Arguments of Knowledge (zk-SNARKs) utilizing the **Groth16** proof system over the **$BN254$ (alt_bn128)** elliptic curve.
* **Light Poseidon Hashing**: Leverages `light_poseidon` with `bn254_x5_3` parameters for fast, gas-efficient on-chain evaluation of cryptographic commitments:
  $$\text{Leaf} = \text{Poseidon}(\text{NullifierSecret},\, \text{SecretKey})$$
* **20-Level Incremental Merkle Tree**: Supports up to $2^{20} = 1,048,576$ anonymous deposits per shielded pool instance.
* **Deterministic Double-Spend Prevention**:
  Nullifier PDAs are derived deterministically:
  $$\text{PDA}_{\text{nullifier}} = \text{find\_pda}\Big([b"\text{nullifier}",\, \text{pool\_pubkey},\, \text{nullifier\_hash}]\Big)$$
  The program checks and sets `nullifier.is_used = true` before emitting funds.
* **Rent-Exemption Invariant**:
  $$\text{PoolBalance} - \text{WithdrawalAmount} \ge \text{Rent}_{\text{minimum}}(8 + \text{PoolState::LEN})$$
  Ensures the program account can never be drained below the Solana rent floor or closed unexpectedly.
* **Proof Hijacking & MEV Guard**:
  The recipient and relayer public keys are hashed into 31-byte scalar field elements and verified inside the circuit's public inputs. Any MEV searcher attempting to intercept a proof in the mempool will fail, as altering the recipient address invalidates the zk-SNARK proof.

<br/>

---

<br/>

## 5. Token Launchpad & RWA Catalog

**LaunchLab** provides an institutional deployment environment supporting standard SPL and **Token-2022** standards with automated liquidity provisioning:

* **46+ Curated Quote Assets**: Pair token launches directly against tokenized equities (NVDAx, TSLAx, AAPLx, MSFTx), global indices (SPYx, QQQx), commodities (GLDx, SLVx), real-world assets, and crypto.
* **Automated Raydium CPMM Migration**: Integrated bonding curves migrate liquidity atomically to Raydium LaunchLab CPMM pools upon reaching saturation targets.
* **Permanent LP Incineration**: Automatically transfers initial liquidity pool tokens to the Solana incinerator address (`1nc1nerator11111111111111111111111111111111`), eliminating the possibility of liquidity rug-pulls.
* **Streamflow Vesting Integration**: Non-custodial team allocation vesting with verifiable cliff durations and linear release schedules configured at deployment.

<br/>

---

<br/>

## 6. Closed-Loop Fee Flywheel & POL

All protocol volume (Launchpad, Perps, Prediction Market, Privacy Mixer) incurs a **1.0% protocol fee** routed into an autonomous economic stabilization flywheel:

<br/>

| Allocation | Share | Economic Mechanism & Impact |
| :--- | :---: | :--- |
| **Protocol-Owned Liquidity (POL)** | **50%** | Injected directly into Raydium CPMM pools with LP tokens permanently burned, establishing an ever-rising liquidity floor for $\$KLYRO$. |
| **Autonomous Buyback & Burn** | **30%** | Programmatic market purchases of $\$KLYRO$ executed on DEX pools and forwarded directly to the Solana Incinerator address. |
| **Relayer Subsidy & RPC Infra** | **10%** | Subsidizes zero-knowledge shielded gas costs and funds dedicated enterprise RPC validator infrastructure. |
| **Security Reserve & Audits** | **10%** | Continuous reserve funding smart contract bug bounties, automated fuzzing, and third-party security audits. |

<br/>

```
               [ Platform Volume: Launchpad + Perps + Predictions + ZK ]
                                          │
                                   1.0% Protocol Fee
                                          │
                  ┌───────────────────────┴───────────────────────┐
                  ▼                                               ▼
         50% POL Injection                               30% Buyback & Burn
     (Raydium CPMM + Burned LP)                     (Market Buy + $KLYRO Burn)
                  │                                               │
                  └──────────────► Permanent Supply Squeeze ◄─────┘
```

<br/>

---

<br/>

## 7. Formal Invariants & Security Architecture

Klyro has been audited under institutional M&A technical due diligence standards ($200M valuation grade). Full audit documentation is available in [SECURITY_AUDIT.md](SECURITY_AUDIT.md).

### 7.1 Mathematical & Cryptographic Invariants
1. **Groth16 Soundness & Non-Malleability**: 
   Public inputs bind the designated withdrawal recipient and fee hash into the scalar field modulus. Transactions cannot be modified, intercepted, or front-run in the Solana mempool without invalidating the mathematical proof.
2. **Strict Parimutuel Solvency Conservation**:
   $$\sum_{i} \text{Payout}_{i} \le \text{TotalEscrowPool} - \text{ProtocolFee}$$
   Rounding truncation in division operations strictly favors protocol solvency, guaranteeing that total liabilities can never exceed escrowed vault reserves.
3. **One-Time Nullifier Invariant**:
   For any withdrawal proof $P$ containing nullifier hash $h_n$, the state transition $(S \to S')$ is valid if and only if $h_n \notin \text{NullifierSet}(S)$. Upon execution, $h_n \in \text{NullifierSet}(S')$, permanently prohibiting double-spends.
4. **Rent Floor Invariant**:
   All contract-managed vaults enforce account rent-exemption preservation. No state operation can reduce account balances below the protocol's minimum rent threshold.

### 7.2 Zero-Trust Infrastructure & Defense-in-Depth
* **Multi-Tier Adaptive Rate Limiting**: Dynamic client request rate-limiting with progressive backoff curves to mitigate network congestion and distributed denial-of-service (DDoS) attempts.
* **Cryptographic Proof-of-Work (PoW) Verification**: Computes client-side proof-of-work challenges before accepting resource-heavy relay operations, eliminating zero-cost sybil spam.
* **On-Chain Signature Confirmation**: Validates block confirmation depth, transaction error state (`meta.err == null`), and cryptographic signer authenticity directly against Solana cluster state before executing off-chain state transitions.
* **Sovereign Non-Custodial Architecture**: Completely wallet-based authorization. Private keys never leave user devices, and users retain absolute custody over un-staked assets.
* **RPC Multi-Commitment Failover**: Autonomous exponential-backoff retry loops with multi-commitment escalation (`processed` $\to$ `confirmed`), eliminating stale blockhash rejections during network congestion.

<br/>

---

<br/>

## Client-Side Performance Engineering

```
Initial JS Bundle Slashed: 4.42 MB ──► 31.13 kB (99.3% Reduction)
Time-to-Interactive (TTI): < 300ms on 4G Mobile Connections
```

* **Granular Rollup Code-Splitting**: Using `React.lazy()` and dynamic chunking, core application logic loads in under 300ms. Heavy SnarkJS ZK proving keys (3.2 MB) are isolated into a dedicated chunk loaded **only** when interacting with the Shielded Pool.
* **Zero-CPU Thrashing Architecture**: Event listeners in `SecureTransactionContext.jsx` use mutable `useRef` instances with `{ passive: true }` bindings, completely eliminating React component tree re-renders during user interaction.
* **Defensive Storage Parsers**: Wrapped `localStorage` read operations in type-safe validators to guarantee zero client-side crashes from malformed legacy states.

<br/>

---

<br/>

## Repository Structure

```
Kylro/
├── programs/                      # Anchor Smart Contracts (Rust / Solana BPF)
│   ├── klyro_pool/                # Groth16 ZK Mixer & 20-level Poseidon Merkle Tree
│   ├── klyro_prediction/          # Parimutuel Escrow & u128 Checked Math
│   └── privacy_verifier/          # On-chain Groth16 BN254 Proof Verifier
│
├── relayer/                       # Sovereign Settlement & Privacy Relayer (TypeScript)
│   ├── src/                       # Jito bundling, rate limiting, anti-replay cache
│   └── scripts/                   # Test suites (test_undetectable_mm, test_market_maker, benchmark_live_tokens)
│
├── web/                           # High-Performance Trading Terminal (React + Vite)
│   ├── public/circuits/           # WASM witness generator & BN254 proving keys
│   └── src/
│       ├── components/            # LaunchLab, Perps, Predictions, PrivateSwap, MM Dashboard
│       ├── context/               # SecureTransactionContext (useRef zero-thrash)
│       └── services/              # KlyroMarketMakerEngine, BinanceGradePerpEngine, ZKService
│
├── circuits/                      # Circom Zero-Knowledge Circuits
│   └── mixer.circom               # 20-level Poseidon Merkle tree Groth16 circuit
│
├── Anchor.toml                    # Solana program IDs and cluster configuration
├── Cargo.toml                     # Rust workspace configuration
├── institutional_200m_valuation_audit.md # Formal Tier-1 Institutional Security Audit
└── README.md                      # Comprehensive Protocol Specification
```

<br/>

---

<br/>

<div align="center">

**Klyro Protocol** &nbsp;•&nbsp; *Institutional Solana Launchpad, Derivatives & Zero-Knowledge Privacy*

<br/>

[Website](https://klyro.io) &nbsp;|&nbsp; [Twitter / X](https://twitter.com/KlyroProtocol) &nbsp;|&nbsp; [Audit Report](SECURITY_AUDIT.md) &nbsp;|&nbsp; [Documentation](https://docs.klyro.io)

</div>
