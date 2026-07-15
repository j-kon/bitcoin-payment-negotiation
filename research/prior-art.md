# Prior-Art Review Framework

This document catalogs existing Bitcoin payment standards, protocols, and specifications. We examine each standard's capabilities, payment details, and how it relates to payment-method selection.

> [!IMPORTANT]
> **Evidence Policy**: Absence of evidence is not evidence of non-support. A feature is marked Unsupported only when official documentation or source code explicitly establishes non-support.

> **Research Finding:** We have not identified a general cross-method selection policy in the specifications and implementations surveyed so far. This is classified as a **research inference**.

---

## Existing Protocols & Standards

### BIP 21 (URI Scheme)
- **Primary Source**: [BIP 21 Specification](https://github.com/bitcoin/bips/blob/master/bip-0021.mediawiki)
- **Specification Status**: Closed and superseded by BIP 321.
- **What it specifies**: A URI scheme representing a destination address, an amount, and optional label/message fields. It allowed unknown optional query parameters.
- **What it does not specify**: Did not standardize the modern set of payment-instruction keys now documented by BIP 321. It does not define rules or algorithms for how a wallet chooses between multiple payment options.
- **Request-integrity properties**: None natively.
- **Relevance to selection**: Historical baseline. In practice, custom query parameters (e.g. `lightning=`) were appended, but selection behavior was left to implementation-level heuristics.
- **Remaining uncertainty**: None.

### BIP 321 (Payment Request URI)
- **Primary Source**: [BIP 321 Specification](https://github.com/bitcoin/bips/blob/master/bip-0321.mediawiki)
- **Specification Status**: Complete (replaces BIP 21).
- **What it specifies**:
  - The URI path may contain an on-chain address.
  - Payment instructions may appear in query parameters.
  - Current standardized keys include `lightning` (BOLT 11), `lno` (BOLT 12), `pay` (BIP 351), and `sp` (BIP 352).
  - Multiple parameters with the same key may be used for payment-instruction keys.
  - Unknown parameters prefixed with `req-` invalidate the URI.
  - Clients must obtain user authorization before acting on a URI.
- **What it does not specify**: The specification does not define a cross-method priority or selection algorithm.
- **Request-integrity properties**: None natively.
- **Relevance to selection**: BIP 321 is the primary vehicle for representing multi-instruction payment requests.
- **Remaining uncertainty**: None.

### BIP 353 (DNS Payment Instructions)
- **Primary Source**: [BIP 353 Specification](https://github.com/bitcoin/bips/blob/master/bip-0353.mediawiki)
- **Specification Status**: Complete.
- **What it specifies**: Resolves human-readable names (like `user@domain.com`) to DNSSEC-validated `bitcoin:` payment instructions. Clients must validate DNSSEC themselves and must not simply trust the remote resolver's validation result.
- **What it does not specify**: Does not define how the wallet selects between the methods contained in the resolved URI.
- **Request-integrity properties**: Rely on DNSSEC for cryptographic origin validation.
- **Relevance to selection**: Acts as a lookup helper to retrieve multi-method BIP 321 URIs.
- **Remaining uncertainty**: The privacy implications of DNS resolver queries and traffic correlation are separate research questions.

### BOLT 11 (Lightning Invoice)
- **Primary Source**: [BOLT 11 Specification](https://github.com/lightning/bolts/blob/master/11-payment-encoding.md)
- **Specification Status**: Complete.
- **What it specifies**: Ephemeral, single-use Lightning invoices containing payment hash, amount, expiry, description, and route hints.
- **What it does not specify**: Multi-rail selection or fallback paths.
- **Request-integrity properties**: Contains a cryptographic signature from the recipient node.
- **Relevance to selection**: Represents a primary off-chain payment instruction.
- **Remaining uncertainty**: None.

### BOLT 12 (Lightning Offers)
- **Primary Source**: [BOLT 12 Specification](https://github.com/lightning/bolts/blob/master/12-offer-encoding.md)
- **Specification Status**: Draft (called a “Negotiation Protocol for Lightning Payments”).
- **What it specifies**: Reusable offers that allow payers to request an ephemeral invoice from a recipient. Its negotiation concerns offers, invoice requests, and invoices *within* Lightning.
- **What it does not specify**: Selection across different payment instructions (e.g. between Lightning and on-chain).
- **Request-integrity properties**: Uses signatures on offers and invoices.
- **Sender Privacy**: BOLT 12 uses onion messages, blinded paths where provided, and payer keys that need not be the payer’s ordinary node identity. Timing and transport metadata remain implementation and network-analysis questions.
- **Relevance to selection**: An interactive Lightning option that must be compared against non-interactive or interactive on-chain alternatives.
- **Remaining uncertainty**: Metadata correlation during onion routing.

### BIP 78 (Payjoin v1)
- **Primary Source**: [BIP 78 Specification](https://github.com/bitcoin/bips/blob/master/bip-0078.mediawiki)
- **Specification Status**: Deployed.
- **What it specifies**: Synchronous on-chain transaction collaboration. It requires the original PSBT (Partially Signed Bitcoin Transaction) to be broadcastable.
- **What it does not specify**: Does not define selection between Payjoin and off-chain protocols.
- **Fallback Behavior**: The specification does not claim that an HTTP failure requires automatic fallback to standard on-chain payment. Whether and when the original transaction is broadcast after negotiation failure is wallet behavior requiring implementation evidence.
- **Request-integrity properties**: Relies on HTTPS for communication channel protection.
- **Relevance to selection**: A collaborative on-chain workflow.
- **Remaining uncertainty**: How wallets handle transaction delays during interactive negotiation.

### BIP 77 (Async Payjoin)
- **Primary Source**: [BIP 77 Specification](https://github.com/bitcoin/bips/blob/master/bip-0077.md)
- **Specification Status**: Draft.
- **What it specifies**: Asynchronous collaborative on-chain transaction construction. It specifies the use of an untrusted store-and-forward directory to relay negotiation messages. Encrypted payloads are routed using Oblivious HTTP (OHTTP) to protect IP privacy.
- **What it does not specify**: Selection between BIP 77 and other rails.
- **Request-integrity properties**: Relies on OHTTP and client-directory payload encryption.
- **Relevance to selection**: A receiver-offline interactive on-chain workflow.
- **Remaining uncertainty**: Directory availability and routing delays.

### BIP 352 (Silent Payments)
- **Primary Source**: [BIP 352 Specification](https://github.com/bitcoin/bips/blob/master/bip-0352.mediawiki)
- **Specification Status**: Complete.
- **What it specifies**: Non-interactive private on-chain transactions. BIP 352 aims for resulting transactions to blend in with ordinary Bitcoin transactions.
  - Static, reusable payment address.
  - Unique on-chain destination per payment.
  - No sender-receiver interaction.
  - Receiver scanning requirement.
- **What it does not specify**: Selection between Silent Payments and other methods.
- **Request-integrity properties**: None natively.
- **Relevance to selection**: A non-interactive private on-chain method.
- **Remaining uncertainty**: Light-client support remains a research and implementation challenge.

### BIP 351 (Private Payments)
- **Primary Source**: [BIP 351 Specification](https://github.com/bitcoin/bips/blob/master/bip-0351.mediawiki)
- **Specification Status**: Complete.
- **What it specifies**: An on-chain payment scheme using reusable payment codes to generate unique, unlinkable addresses for each transaction without sender-receiver interaction.
- **What it does not specify**: Selection policy or negotiation between BIP 351 and other methods.
- **Request-integrity properties**: None natively.
- **Relevance to selection**: A non-interactive private on-chain method that wallets need to compare against standard on-chain or off-chain alternatives.
- **Remaining uncertainty**: Ecosystem adoption remains limited.

### BIP 70 (Payment Protocol)
- **Primary Source**: [BIP 70 Specification](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki)
- **Specification Status**: Deprecated.
- **What it specifies**: An interactive, HTTP-based protocol for communication between a merchant and a customer's wallet. Standardizes payment request headers, payment details, and refund addresses.
- **What it does not specify**: Dynamic selection between different underlying payment rails (like Lightning or Silent Payments), as it was designed around on-chain transactions.
- **Request-integrity properties**: Protects integrity using X.509 certificates and digital signatures on `PaymentRequest` messages.
- **Relevance to selection**: An early example of interactive, signed payment request delivery and payment response workflows.
- **Remaining uncertainty**: None (deprecated due to complexity and security issues).

### bitcoin-payment-instructions (Reference Implementation)
- **Primary Source**: [bitcoin-payment-instructions Codebase](https://github.com/rust-bitcoin/bitcoin-payment-instructions)
- **Specification Status**: Active reference implementation.
- **What it specifies**: A Rust library implementing parsing and representation of BIP 321 payment requests.
- **What it does not specify**: Selection policy or default method selection algorithms.
- **Request-integrity properties**: None natively.
- **Relevance to selection**: Serves as a reference implementation for handling BIP 321 URIs across multiple rails.
- **Remaining uncertainty**: Client application layer integration patterns.

### LNURL Pay (LUD-06)
- **Primary Source**: [LNURL Specification (LUDs)](https://github.com/lnurl/luds)
- **Specification Status**: De facto standard.
- **What it specifies**: An interactive HTTP/HTTPS workflow where a client fetches payment parameters and a BOLT 11 invoice from a server.
- **What it does not specify**: Cross-method selection policy (e.g. falling back to on-chain).
- **Request-integrity properties**: HTTPS transport security.
- **Relevance to selection**: Widely deployed interactive off-chain request protocol.
- **Remaining uncertainty**: Privacy implications of direct server requests.

### Lightning Address
- **Primary Source**: [Lightning Address Specification](https://github.com/andrerfneves/lightning-address)
- **Specification Status**: De facto standard.
- **What it specifies**: A protocol for translating an email-like identifier to an LNURL Pay endpoint.
- **What it does not specify**: Cross-rail selection policy.
- **Request-integrity properties**: HTTPS transport security.
- **Relevance to selection**: DNS-based lookup helper yielding LNURL Pay.
- **Remaining uncertainty**: Metadata leakage and server custody assumptions.

---

## Summary of Claims & Evidence

| Claim / Behaviour | Evidence Type | Source Reference | Confidence | Notes |
| :--- | :--- | :--- | :--- | :--- |
| BIP 321 does not define a cross-method priority or selection algorithm | Directly specified | [BIP 321 Section: General Format](https://github.com/bitcoin/bips/blob/master/bip-0321.mediawiki) | Confirmed by specification | Deciding which rail to use is left to the client application. |
| BIP 78 requires the original PSBT to be broadcastable | Directly specified | [BIP 78 Specification](https://github.com/bitcoin/bips/blob/master/bip-0078.mediawiki) | Confirmed by specification | Payjoin v1 relies on a valid fallback transaction being ready. |
| BOLT 12 uses onion messages and blinded paths to protect node identity | Directly specified | [BOLT 12 Specification](https://github.com/lightning/bolts/blob/master/12-offer-encoding.md) | Confirmed by specification | Blinded paths prevent revealing the recipient's node ID. |
| BIP 353 requires client-side DNSSEC validation | Directly specified | [BIP 353 Specification](https://github.com/bitcoin/bips/blob/master/bip-0353.mediawiki) | Confirmed by specification | Trusting the resolver's validation status is forbidden. |
| BIP 352 Silent Payments outputs are indistinguishable on-chain | Directly specified | [BIP 352 Specification](https://github.com/bitcoin/bips/blob/master/bip-0352.mediawiki) | Confirmed by specification | Outputs look like standard P2TR scriptpubkeys. |
| BIP 77 specifies store-and-forward directory relaying with OHTTP encryption | Directly specified | [BIP 77 Specification](https://github.com/bitcoin/bips/blob/master/bip-0077.md) | Confirmed by specification | Protects sender IP from directory host. |
| Intermediaries can strip query parameters if they control the delivery channel | Research inference | Threat Model analysis | Reasonable inference | Standard security property of unauthenticated or unencrypted channels. |
| BIP 352 light-client scanning introduces wallet latency | Research inference | Developer discussions | Reasonable inference | Light clients must fetch filters/keys to scan blocks. |
