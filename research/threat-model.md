# Threat Model

This document outlines the conceptual threat model for payment-method selection and negotiation. We analyze the active and passive actors, specific threats to security and privacy, and potential mitigations.

---

## Threat Actors

We consider the following threat actors in our analysis:

- **Curious Payer**: A sender who wishes to discover which payment methods or wallet brands the receiver supports, or probe the receiver's limits without completing a payment.
- **Curious Receiver**: A merchant or recipient who seeks to identify the sender's wallet software, historical UTXOs, or active Lightning node details via the selection or negotiation flow.
- **Network Observer**: An eavesdropper monitoring network traffic (e.g., HTTP, DNS, Nostr, or Lightning onion routing) between the payer, receiver, and third-party helpers.
- **Malicious Payment-Instruction Host**: An entity hosting BIP 353 DNS records, web servers (LNURL/Payjoin endpoints), or other external resources that tampers with the served payment instructions.
- **Malicious Directory or Relay**: An intermediary node (e.g., a Nostr relay or DNS resolver) that routes negotiation messages and can drop, delay, or manipulate them.
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
- **Existing Mitigations**: Minimal. Currently, wallets default to local selection heuristics, but these heuristics are often unique to each wallet brand (e.g., one wallet always prefers Lightning if present, while another always prefers Payjoin).
- **Open Questions**: Can wallets mask their capability set by randomizing selection or introducing artificial delays? Can we define a baseline "standard client policy" to prevent fingerprinting?

### 2. Protocol Downgrade (Removal of Preferred Instructions)
- **Attacker**: Payment Request Manipulator or Network Observer.
- **Attacker Capability**: Intercepting a BIP 321 payload and stripping away privacy-enhancing or low-fee instructions (like Silent Payments or BOLT 12), leaving only standard on-chain instructions.
- **Target**: The payment request payload.
- **Possible Impact**: Forces the payer's wallet to fall back to a less-private, higher-fee standard on-chain payment method, revealing UTXO ownership.
- **Existing Mitigations**: End-to-end cryptographic signatures on the payment request (e.g., BIP 322 or SSL/TLS for web endpoints).
- **Open Questions**: How can a wallet verify that the received payment request has not been stripped of other options if it was scanned from a offline QR code?

### 3. Rare-Feature Identification
- **Attacker**: Network Observer or Passive Blockchain Observer.
- **Attacker Capability**: Observing transactions on-chain or network handshakes that utilize rare or newly introduced features (e.g., Silent Payments, Payjoin v2).
- **Target**: Transaction privacy.
- **Possible Impact**: A very small subset of users utilize these features, making their transactions highly stand out from the general transaction pool, facilitating tracking.
- **Existing Mitigations**: Encouraging wider adoption of advanced features to enlarge the anonymity set.
- **Open Questions**: Should wallets avoid using a superior payment method if its anonymity set in the active block space is too small?

### 4. Replay of Expired Instructions
- **Attacker**: Curious Payer or Payment Request Manipulator.
- **Attacker Capability**: Capturing and re-submitting an old, expired payment request (e.g., a BOLT 11 invoice or a single-use on-chain address).
- **Target**: Receiver's accounting system and liquidity.
- **Possible Impact**: Double-payments, payment correlation, locking up receiver's channels, or routing fees incurred by accidental re-settlement.
- **Existing Mitigations**: Expiration timestamps within the payment request parameters (e.g., `exp` or standard Lightning expiry fields).
- **Open Questions**: How should a wallet handle selection when some instructions within a BIP 321 payload have expired but others are still valid?

### 5. Correlation between Negotiation and Settlement
- **Attacker**: Network Observer or Curious Receiver.
- **Attacker Capability**: Correlating the IP addresses and timestamps of negotiation steps (like fetching a BOLT 12 invoice or querying a Payjoin endpoint) with the final broadcast of an on-chain transaction or Lightning onion route.
- **Target**: Transaction anonymity.
- **Possible Impact**: Connects the user's real-world identity/IP address to specific on-chain UTXOs or Lightning node IDs.
- **Existing Mitigations**: Routing negotiation traffic over Tor/VPN; delaying the broadcast of transactions relative to the negotiation handshake.
- **Open Questions**: Does interactive negotiation inherently leak more timing correlation data than local selection?

### 6. False Security-Property Advertisements
- **Attacker**: Malicious Payment-Instruction Host or Receiver.
- **Attacker Capability**: Falsely claiming that a specific payment instruction supports a higher security level or custody model than it actually does.
- **Target**: Payer trust and funds safety.
- **Possible Impact**: Payer sends funds to an address believing it is a self-custodial multi-sig script, but it is actually a single-key custodial endpoint, or sending funds to a counterfeit lightning node.
- **Existing Mitigations**: Client-side verification of addresses, scripts, and public keys.
- **Open Questions**: How can a wallet safely represent and compare trust and custody models without misleading the user?

---

## Important Architectural Notes

> [!WARNING]
> **No Custody or Settlement Proofs via Negotiation**
> 
> Capability negotiation or instruction selection protocols **cannot** cryptographically prove the custody model, trust assumptions, or settlement guarantees of the target payment method. A receiver's node or server can lie about its security properties or custody status. 
> 
> The selection layer must treat all self-reported capabilities and security properties as unverified claims, and must rely on cryptographic verification of the transaction outputs themselves.
