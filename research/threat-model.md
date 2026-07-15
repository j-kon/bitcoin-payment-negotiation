# Threat Model

This document outlines the conceptual threat model for payment-method selection and negotiation. We analyze the active and passive actors, specific threats to security and privacy, and potential mitigations.

---

## Threat Actors

We consider the following threat actors in our analysis:

- **Curious Payer**: A sender who wishes to discover which payment methods or wallet brands the receiver supports, or probe the receiver's limits without completing a payment.
- **Curious Receiver**: A merchant or recipient who seeks to identify the sender's wallet software, historical UTXOs, or active Lightning node details via the selection or negotiation flow.
- **Network Observer**: An eavesdropper monitoring network traffic (e.g., HTTP, DNS, or Lightning onion routing) between the payer, receiver, and third-party helpers.
- **Malicious Payment-Instruction Host**: An entity hosting BIP 353 DNS records, web servers (Payjoin endpoints), or other external resources that tampers with the served payment instructions.
- **Malicious Directory or Relay**: An intermediary node (e.g., a BIP 77 store-and-forward directory or DNS resolver) that routes negotiation messages and can drop, delay, or manipulate them.
- **Compromised Wallet**: A payer's or receiver's wallet application that has been infected or modified to leak capabilities, private keys, or historical transaction logs.
- **Payment Request Manipulator**: An active attacker capable of altering static QR codes, print media, or digital payment requests in transit (e.g., Man-in-the-Middle) to alter destinations or remove instructions.
- **Passive Blockchain Observer**: An analyst using public blockchain data (heuristics, address reuse, input linkage) to track transaction ownership.

---

## Threat Catalog

### 1. Wallet Fingerprinting & Capability Probing
- **Attacker**: Curious Receiver or Malicious Payment-Instruction Host.
- **Attacker Capability**: Serving a payment request with a wide variety of optional, obscure, or highly specific payment instructions, and observing which specific method the payer selects or queries.
- **Target**: Payer's software identity and device characteristics.
- **Possible Impact**: The receiver uniquely identifies the payer's wallet brand, version, or user settings. This allows target profiling, tracking across different checkouts, or targeted exploit delivery.
- **Existing Mitigations**: Existing behavior and mitigations have not yet been comprehensively surveyed.
- **Open Questions**: Can wallets mask their capability set by randomizing selection or introducing artificial delays? Can we define a baseline standard client policy to prevent fingerprinting?

### 2. Protocol Downgrade (Removal of Preferred Instructions)
- **Attacker**: Payment Request Manipulator or Network Observer.
- **Attacker Capability**: Intercepting a BIP 321 payload and stripping away privacy-enhancing or low-fee instructions (like Silent Payments or BOLT 12), leaving only standard on-chain instructions.
- **Target**: The payment request payload.
- **Possible Impact**: Forces the payer's wallet to fall back to standard on-chain payment method, revealing UTXO ownership.
- **Existing Mitigations**: Authenticated transport (like HTTPS) or a separately defined signed envelope can protect integrity. However, this project has not found a standard BIP 321 signature envelope.
- **Open Questions**: How can a wallet verify that the received payment request has not been stripped of other options if it was scanned from an offline QR code?

### 3. Rare-Feature Identification
- **Attacker**: Network Observer or Passive Blockchain Observer.
- **Attacker Capability**: Observing transactions or network handshakes that utilize rare or newly introduced features.
- **Target**: Transaction privacy.
- **Possible Impact**: A very small subset of users utilize these features, making their transactions stand out from the general transaction pool, facilitating tracking. We reframe this threat around network behavior, implementation quirks, unsupported error messages, or distinguishable transaction structures where evidence exists. Note that BIP 352 Silent Payments are explicitly designed so that resulting transaction outputs blend in with ordinary Taproot (P2TR) outputs, making them indistinguishable on-chain.
- **Existing Mitigations**: Encouraging wider adoption of advanced features to enlarge the anonymity set.
- **Open Questions**: How do we trace feature adoption metrics safely?

### 4. Replay of Expired Instructions
- **Attacker**: Curious Payer or Payment Request Manipulator.
- **Attacker Capability**: Capturing and re-submitting an old, expired payment request.
- **Target**: Receiver's accounting system and liquidity.
- **Possible Impact**: Expired BOLT 11 invoices should be rejected by the recipient's node. However, ordinary on-chain addresses do not inherently expire on the protocol level, leading to potential accounting confusion, loss of funds to outdated keys, or address reuse.
- **Existing Mitigations**: Wallet-side warnings, server-side database validation of transaction hashes, and invoice expiry parameters.
- **Open Questions**: How should a wallet handle selection when some instructions within a payload have expired but others are still valid?

### 5. Correlation between Negotiation and Settlement
- **Attacker**: Network Observer or Curious Receiver.
- **Attacker Capability**: Correlating the IP addresses and timestamps of negotiation steps (like fetching a BOLT 12 invoice or querying a Payjoin endpoint) with the final broadcast of an on-chain transaction or Lightning routing.
- **Target**: Transaction anonymity.
- **Possible Impact**: Connects the user's real-world identity/IP address to specific on-chain UTXOs or Lightning node IDs.
- **Existing Mitigations**: Distinguish the metadata visible to a BOLT 12 receiver (who sees only the blinded path/payer key) from metadata visible to routing nodes or network observers (who might see timing patterns).
- **Open Questions**: Does interactive negotiation inherently leak more timing correlation data than local selection?

### 6. False Security-Property Advertisements
- **Attacker**: Malicious Payment-Instruction Host or Receiver.
- **Attacker Capability**: Falsely claiming that a specific payment instruction supports a higher security level or custody model than it actually does.
- **Target**: Payer trust and funds safety.
- **Possible Impact**: Payer sends funds to an address believing it is a self-custodial multi-sig script, but it is actually a single-key custodial endpoint. This is a prospective threat only if a future negotiation format introduces self-reported security metadata.
- **Existing Mitigations**: Client-side verification of addresses, scripts, and public keys. Note that transaction outputs cannot always prove custody or trust models (e.g. multi-sig vs. custodial wallets).
- **Open Questions**: How can a wallet safely represent and compare trust and custody models without misleading the user?
