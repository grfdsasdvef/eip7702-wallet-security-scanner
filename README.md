# 🛡️ EIP-7702 Security Threat Model & Wallet Delegation Scanner

[![Security Research](https://img.shields.io/badge/Security-EIP--7702%20Audit-red.svg)](https://7702.vercel.app)
[![Live Scanner](https://img.shields.io/badge/Live%20Scanner-7702.vercel.app-blue.svg)](https://7702.vercel.app)
[![EVM Networks](https://img.shields.io/badge/Supported%20Networks-8%20EVM%20Chains-green.svg)](https://7702.vercel.app)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> **Free Multi-Chain Verification Tool:** [https://7702.vercel.app](https://7702.vercel.app)  
> *Zero sign-in. Zero seed phrase requirements. Zero wallet connections. Instant 5-second verdict.*

---

## 🚨 Executive Summary: The Invisible EOA Hijack

With the rollout of **EIP-7702** (*Set EOA Account Code for One Transaction or Session*), the fundamental security boundary between **Externally Owned Accounts (EOAs)** and **Smart Contract Accounts (SCAs)** has been completely rewritten.

While originally designed to unlock native Account Abstraction (sponsored gas, batched executions, session keys) for standard wallets like MetaMask, Rabby, and Trust Wallet, it introduces a severe security attack surface: **Silent Bytecode Delegation Takeover**.

```
[Attacker Phishing dApp]
         │
         ▼  (Victim signs what looks like a harmless Permit / Sign-In message)
[Signed EIP-7702 Authorization]
         │
         ▼  (EVM assigns malicious contract bytecode pointer to victim's EOA)
[Victim's EOA Address] ────► [Malicious Implementation Proxy]
         │
         ▼  (Inbound transfers / deposits trigger proxy fallback hooks)
[Automatic Atomic Sweep to Attacker's Vault]
```

### The Fatal Trap
1. **The Seed Phrase Remains Uncompromised:** The victim never gives away their 12/24 words or private key.
2. **The Interface Looks Normal:** Standard wallet UIs and classic block explorers still label the address as a healthy EOA.
3. **Execution Logic is Hijacked:** Any native coin (ETH, BNB, MATIC) or ERC-20 token sent to the address can be atomically intercepted and drained by the delegated implementation contract in the exact same transaction block.

---

## 🔬 Architectural Threat Breakdown

### 1. Phishing the Authorization Tuple
Unlike traditional smart contract approvals (`approve(spender, uint256)`), which only give allowance for a single specific token contract, an **EIP-7702 authorization delegates execution rights for the entire EOA identity**.

The user is tricked into signing an authorization payload containing:
* Target chain identifier (`chain_id`)
* Implementation contract address (`address`)
* Nonce sequence (`nonce`)

Because standard wallet sign popups historically treated arbitrary signature requests with generic warnings, victims assume they are merely verifying account ownership.

### 2. The Code Pointer Mutation
Once submitted on-chain via a Type `0x04` transaction, the Ethereum state trie updates the account's code field to a delegation designator:
```
0xef0100 + <20-byte implementation contract address>
```
From that moment onwards, whenever the victim's EOA receives a call, EVM executes the bytecode of the designated contract **in the context of the victim's EOA storage**.

### 3. Exploitation Vectors in the Wild

| Attack Vector | Mechanism | Impact |
| :--- | :--- | :--- |
| **Inbound Deposit Sweeper** | Fallback hook monitors incoming balance and immediately initiates batch transfer to attacker address. | User cannot receive funds, airdrops, or salaries; all deposits are siphoned instantly. |
| **Silent Sponsored Drain** | Attacker sponsors gas fees via a relayer, invoking arbitrary ERC-20 approvals on behalf of the victim's address. | Zero gas needed on the victim's balance; all portfolio tokens drained atomically. |
| **Cross-Chain Poisoning** | If the signature did not specify a unique `chain_id` (or specified 0), the same authorization can be replayed across all EVM networks. | Simultaneous loss of assets across Arbitrum, Base, Optimism, Polygon, and BSC. |

---

## 🔍 How to Audit Your Address: The 7702 Scanner

Because legacy block explorers do not proactively flag delegated bytecode on EOA addresses, we built **7702.vercel.app** — an instant multi-chain state auditor.

### ✨ What It Does:
* **Bytecode Pointer Inspection:** Directly queries the consensus state and RPC bytecode (`eth_getCode`) for the `0xef0100` delegation prefix.
* **5-Second Verification:** Concurrently audits all 8 major EVM networks simultaneously.
* **100% Client-Safe:** Read-only RPC calls. Never asks for private keys, signatures, or wallet connection.

### 🌐 Supported Networks (8 EVM Ecosystems):
1. **Ethereum Mainnet** (ETH)
2. **BNB Smart Chain** (BNB)
3. **Base** (Coinbase L2)
4. **Arbitrum One** (L2)
5. **Optimism** (OP Mainnet)
6. **Polygon PoS** (POL)
7. **Gnosis Chain** (xDAI)
8. **Scroll** (zkEVM L2)

---

## 🛠️ Remediation & Recovery

If the scanner indicates that your address has an active delegation to an unknown or untrusted contract:

1. **Stop Depositing Assets:** Do not send gas, tokens, or NFTs to the compromised address.
2. **Revocation Transaction:** Submit a new EIP-7702 transaction pointing the code pointer to `0x0000000000000000000000000000000000000000` (clearing the bytecode delegation) or incrementing the account nonce.
3. **Migrate Cold Assets:** For maximum security, rotate cold storage and transfer all non-compromised assets to a completely fresh, unassociated keypair.

---

## 🔗 Resources & Links

* 🌐 **Live Web Scanner:** [https://7702.vercel.app](https://7702.vercel.app)
* 📡 **Telegram Security Lab:** [@snowdensay](https://t.me/snowdensay) — Deep dives into EVM mechanics, eBPF tracing, and zero-day threat analysis.
* 🤖 **Architecture Monographs & Source Code:** [@Snowden_book_bot](https://t.me/Snowden_book_bot)
