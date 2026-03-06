<div align="center">

# Klyro Protocol

**Institutional-Grade Privacy Infrastructure for Solana**

[![Solana](https://img.shields.io/badge/Solana-black?style=for-the-badge&logo=solana&logoColor=14F195)](https://solana.com)
[![Rust](https://img.shields.io/badge/Anchor%20·%20Rust-black?style=for-the-badge&logo=rust&logoColor=white)](https://www.anchor-lang.com)
[![ZK-SNARKs](https://img.shields.io/badge/Groth16%20ZK--SNARKs-blueviolet?style=for-the-badge)](https://iden3.io/circom)
[![React](https://img.shields.io/badge/React%20·%20Vite-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://reactjs.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white)](https://typescriptlang.org)
[![License](https://img.shields.io/badge/License-BSL%201.1-green?style=for-the-badge)](LICENSE)

---

Klyro is a **zero-knowledge privacy protocol** built natively on the Solana blockchain.  
It enables private token launches, shielded swaps, and anonymous transactions  
at the speed of Solana — utilizing **Groth16 ZK-SNARK** proofs verified on-chain.

[Documentation](#architecture) · [Security Model](#security-architecture) · [Deploy](#deployment) · [API Reference](#api-reference)

</div>

---

## Why Klyro?

Today, every Solana transaction is fully transparent. Wallet addresses, balances, and trade history are open to the world. This creates critical vulnerabilities for professional market participants:

| Problem | Impact |
|---------|--------|
| **Wallet surveillance** | Anyone can track your on-chain activity and strategies |
| **Front-running & MEV** | Bots extract value from your pending transactions |
| **Launch sniping** | Team wallets are identified and replicated within seconds |
| **Address clustering** | Transfer patterns link isolated wallets to real identities |

Klyro solves these issues by combining **ZK-SNARK cryptographic proofs** with **Jito atomic bundles** and a **non-custodial relay architecture** — making it possible to transact privately without placing trust in any centralized third party.

---

## Core Technology

### Zero-Knowledge Proofs (Groth16 on BN254)

Klyro employs **Groth16 ZK-SNARK proofs** over the BN254 elliptic curve — an industry-standard cryptographic framework widely adopted across blockchain privacy protocols. Proofs are:

- **Generated client-side** — cryptographic secrets never leave the user's local execution environment
- **Verified on-chain** — Anchor smart contract mathematically validates proofs using `arkworks` (Rust)
- **Publicly auditable** — the verification key is embedded in the program binary accompanied by an integrity hash

```
User Browser                    Solana Program
┌─────────────────┐            ┌─────────────────┐
│ circom circuit   │            │ arkworks BN254   │
│ snarkjs.groth16  │ ──proof──▶│ Groth16::verify  │
│ witness(secret,  │            │ prepare_vk()     │
│  nullifier, path)│            │ public_inputs[]  │
└─────────────────┘            └─────────────────┘
```

### Merkle Tree Mixer

The privacy pool implements a **Merkle tree of commitments**. When a user deposits funds, their commitment (the hash of a secret paired with a nullifier) is appended as a leaf node. When withdrawing, they prove membership within the tree without revealing which specific leaf node belongs to them, completely breaking the on-chain link.

```
Deposit:  commitment = hash(secret, nullifier) → Merkle tree leaf
Withdraw: ZK proof of (secret, nullifier, path) → fresh wallet receives funds
```

The nullifier strictly prevents double-spending; each valid deposit can mathematically only be successfully withdrawn once.

### Jito Bundle Integration

Token launches exclusively use **Jito bundles** for atomic execution. All transactions within a bundle either succeed collectively or fail entirely — eliminating vulnerabilities such as front-running, sandwich attacks, and partial execution failures.

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                        KLYRO PROTOCOL                            │
├──────────────┬──────────────────┬────────────────────────────────┤
│  Smart       │  Relayer         │  Frontend                     │
│  Contract    │  (TypeScript)    │  (React + Vite)               │
│              │                  │                               │
│  ┌────────┐  │  ┌────────────┐  │  ┌──────────────────────┐     │
│  │Pool    │  │  │HMAC Auth   │  │  │PrivateSwap.jsx       │     │
│  │State   │  │  │Circuit Brkr│  │  │BundleLauncher.jsx    │     │
│  │Multisig│  │  │Monitoring  │  │  │AntiPhishing.jsx      │     │
│  │Timelock│  │  │Relay       │  │  │KlyroAI.jsx           │     │
│  └────────┘  │  │SecKeystore │  │  │TokenLauncher.jsx     │     │
│              │  └────────────┘  │  └──────────────────────┘     │
│  Anchor/Rust │  Express + TLS   │  WebCrypto + snarkjs          │
├──────────────┴──────────────────┴────────────────────────────────┤
│           Solana Blockchain (Devnet / Mainnet)                   │
└──────────────────────────────────────────────────────────────────┘
```

### Project Structure

```
klyro/
├── programs/
│   ├── klyro_pool/src/lib.rs       # Privacy pool (deposit, withdraw, pause, multisig, timelock)
│   └── privacy_verifier/           # ZK proof verifier program
├── relayer/src/
│   ├── index.ts                    # Express server with HTTPS + HSTS
│   ├── relay.ts                    # Withdrawal relay handler
│   ├── bundle-launcher.ts          # Klyro Bundle Launch handler
│   ├── hmac-auth.ts                # HMAC-SHA256 request authentication
│   ├── circuit-breaker.ts          # Rate limiter + IP blacklisting
│   ├── monitoring.ts               # Balance alerts + webhook notifications
│   ├── secure-keystore.ts          # AES-256-GCM encrypted key storage
│   ├── merkle.ts                   # Server-side Merkle tree
│   └── logger.ts                   # Privacy-safe structured logging
├── web/src/
│   ├── components/
│   │   ├── PrivateSwap.jsx         # ZK deposit/withdraw UI
│   │   ├── BundleLauncher.jsx      # Private token launch UI
│   │   ├── AntiPhishing.jsx        # Anti-phishing verification banner
│   │   ├── KlyroAI.jsx             # AI trading assistant
│   │   └── ...                     # 15+ modular components
│   ├── hooks/useKlyroPrivacy.js    # ZK proof generation hook
│   └── services/                   # Trading engine, Jupiter, analysis
├── circuits/
│   └── mixer.circom                # Groth16 mixer circuit (Circom)
├── SECURITY.md                     # Full security audit documentation
└── Anchor.toml                     # Solana program configuration
```

---

## Security Architecture

Klyro has undergone **comprehensive security hardening** across all protocol layers. Complete details can be found in [`SECURITY.md`](SECURITY.md).

### Smart Contract (Anchor/Rust)

| Control | Description |
|---------|-------------|
| **Emergency Pause** | `pause()` / `unpause()` instructions — instantly freeze all deposits and withdrawals |
| **2/3 Multisig** | All admin actions require 2 of 3 authorized signers |
| **48h Upgrade Timelock** | Program upgrades must be proposed 48 hours in advance; users can exit before changes apply |
| **Reentrancy Guard** | `pool.locked` flag prevents re-entrant calls during state mutations |
| **Overflow Protection** | All arithmetic strictly utilizes `checked_add` / `checked_sub` / `checked_mul` |
| **Proof Malleability** | G1/G2 affine point infinity checks prevent malleated proofs |
| **Input Bounds** | Deposit range strictly bounded: 0.1–100 SOL. Fee cap: 1%. Proof size: ≤256 bytes |
| **Circuit Breaker** | Automatically pauses operations if >10 withdrawals occur within a 60-second window |
| **Nullifier PDA** | Robust double-spend prevention through PDA-seeded nullifier accounts |

### Relayer (TypeScript)

| Control | Description |
|---------|-------------|
| **HMAC-SHA256 Auth** | Every request mandates a valid signature combined with a timestamp (30s freshness limit) |
| **TLS / HSTS** | HTTPS strictly enforced with automatic HTTP→HTTPS redirection |
| **Circuit Breaker** | Internal relayer-side rate limiter matched with IP blacklisting protocols |
| **Balance Monitoring** | Real-time automated alerts for low relayer/pool balances triggered via webhook |
| **Encrypted Keystore** | AES-256-GCM encrypted key storage paired with PBKDF2 (100,000 algorithmic iterations) |
| **Rate Limiting** | Strict Express rate limiter: maximum 10 requests/minute per IP address |
| **TX Simulation** | Every constructed transaction is simulated safely before network broadcast |

### Frontend (React)

| Control | Description |
|---------|-------------|
| **Content Security Policy** | Strict CSP headers definitively blocking XSS and injection attack vectors |
| **Anti-Phishing** | User-defined verification phrase persistently displayed on every active page load |
| **Note Encryption** | AES-256-GCM + PBKDF2 high-grade encryption applied to withdrawal notes |
| **Dependency Pinning** | All package versions are locked to absolute exacts — strictly no `^` or `~` ranges |
| **Non-Custodial** | All cryptographic signing occurs isolated within Phantom/Solflare — no private keys interact with the server |

---

## Klyro AI

The platform seamlessly integrates a sophisticated **AI trading assistant** powered by a multi-tiered dual-engine architecture:

| Engine | Latency | Purpose |
|--------|---------|---------|
| **Fast Action** | <100ms | Instant trade execution, rapid wallet queries, real-time command processing |
| **Deep Research** | 2–10s | Comprehensive contract audits, complex market analysis, statistical trend forecasting |

Supported Natural Language Command structures:
```
"Acquire 5 SOL of $WIF"
"Perform an analysis of the top 10 trending tokens"
"Execute a security audit on this contract"
"Provide current market sentiment analysis on $BONK"
```

---

## Deployment

### Prerequisites

- **Node.js** ≥ 18.0
- **Rust** + **Anchor CLI** ≥ 0.30
- **Solana CLI** (for robust key management and deployment procedures)
- Institutional Wallet (**Phantom** or **Solflare**)

### Quick Start Initialization

```bash
# Clone Repository
git clone https://github.com/KlyroTerminal/Klyro.git
cd Klyro

# Compile Smart Contract
anchor build -- --tools-version v1.43

# Build Relayer
cd relayer && npm install && npm run build

# Initialize Frontend
cd ../web && npm install && npm run dev
```

### Environment Variable Requirements

| Variable | Description |
|----------|-------------|
| `VITE_RPC_URL` | Solana RPC endpoint connection string (e.g. Helius, Alchemy) |
| `KLYRO_HMAC_SECRET` | HMAC cryptographic signing secret for relayer authentication |
| `SSL_CERT_PATH` / `SSL_KEY_PATH` | Discrete TLS certificate paths strictly for HTTPS termination |
| `ALERT_WEBHOOK_URL` | Integration webhook endpoint for automated monitoring alerts |
| `KEYSTORE_PATH` | Absolute path to the securely encrypted relayer keystore |

---

## API Reference

### Relayer Endpoints

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/status` | GET | — | Core service health diagnostic check |
| `/health` | GET | — | Advanced monitoring status and circuit breaker state query |
| `/merkle-proof` | GET | — | Retrieve the corresponding Merkle proof for an active commitment |
| `/relay` | POST | HMAC | Submit ZK withdrawal proof payload for execution relay |
| `/klyro-bundle-launch` | POST | HMAC | Execute private token launch utilizing sequential Jito bundles |

### Frontend API Proxy

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/ipfs` | POST | Securely upload token metadata to IPFS |
| `/api/trade` | POST | Algorithmically construct trade transactions |
| `/api/launch-with-bundler` | POST | Safely complete bundled token launch protocols |
| `/api/analyze-token` | POST | Trigger advanced bundle detection and quantitative market analysis |

---

## Technical Workflows

### Private Swap (Mixer)

```
1. DEPOSIT                          2. WITHDRAW
   ┌──────────┐                        ┌──────────┐
   │ User     │                        │ User     │
   │ Wallet A │                        │ (proves  │
   └────┬─────┘                        │  secret) │
        │                              └────┬─────┘
        │ SOL + commitment                  │ ZK proof
        ▼                              ▼
   ┌──────────┐                   ┌──────────┐
   │ Klyro    │                   │ Relayer  │──▶ Verifies cryptographic proof
   │ Pool     │                   │          │──▶ Submits sequentially to Solana
   │ (PDA)    │                   │          │──▶ Pays network gas fees
   └──────────┘                   └────┬─────┘
                                       │
                                       ▼
                                  ┌──────────┐
                                  │ Fresh    │
                                  │ Wallet B │ ← Cleanly receives funded assets
                                  └──────────┘
```

Absolutely no on-chain link exists between Wallet A and Wallet B.

### Private Token Launch

```
1. User strictly configures token parameters → 2. ZK-shields liquidity capital via mixer structure
→ 3. Dynamically generates N anonymous operational wallets → 4. Distributes SOL implementing randomized chronological delays
→ 5. Atomic Jito bundle deployment: simultaneous minting and acquisition across all wallets
→ 6. Result: No discernible public link between the primary team wallet and the asset launch
```

---

## Technology Stack

| Layer | Technology | Infrastructure Purpose |
|-------|-----------|-----|
| **Consensus** | Solana (400ms finality) | Industry-leading L1 for high-throughput ZK verification |
| **Smart Contract** | Anchor (Rust) + arkworks | Highly-optimized native BN254 curve arithmetic operations on-chain |
| **ZK Proving** | Circom + snarkjs (Groth16) | Enterprise-grade SNARK framework executing as a client-side prover |
| **Relayer** | Express + TypeScript | Secure, privacy-preserving transaction abstraction layer |
| **Frontend** | React 18 + Vite | Sub-second HMR accompanied by WebCrypto API for deep client-side encryption |
| **Bundling** | Jito Block Engine | Guarantees atomic bundle execution delivering complete MEV protection |
| **AI** | Advanced LLM Integration | Dual-engine computational intelligence optimized for trading velocity and comprehensive research |

---

## Contributing Guidelines

The Klyro Protocol is currently operating in locked closed development preceding a comprehensive professional audit. 

Security researchers interested in conducting independent analyses of the protocol architecture may initiate contact through the official channels listed below.

## License Compliance

Business Source License 1.1 — Reference [LICENSE](LICENSE) for exact limitations and parameters.

---

<div align="center">

**Klyro Protocol** — *Uncompromising Privacy Infrastructure for Solana, Powered by Zero-Knowledge Cryptography.*

[Website](https://klyro.io) · [Twitter](https://twitter.com/KlyroProtocol) · [Discord](https://discord.gg/klyro)

</div>
