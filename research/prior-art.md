# Prior-Art Review Framework

This document catalogs and reviews existing Bitcoin payment standards, specifications, and observed wallet implementations. It examines how these protocols handle multiple payment instructions and method selection.

> **Research Finding:** We have not yet identified a common cross-protocol specification addressing this exact combination of payment-method selection and negotiation concerns. This is classified as a **research inference**.

---

## Existing Protocols & Standards

### BIP 21 (URI Scheme)
- **Primary Source**: [BIP 21 Specification](https://github.com/bitcoin/bips/blob/master/bip-0021.mediawiki)
- **Specification Status**: Standard / Final.
- **What is directly specified**: A standard URI scheme for representing a single Bitcoin destination address, an amount, and optional label/message parameters.
- **What is not specified**: Does not specify representation of multiple alternative payment methods or selection logic.
- **Known implementation evidence**: Supported by virtually all Bitcoin wallets (e.g., BlueWallet, Phoenix, Nunchuk).
- **Relevance to selection**: Historical baseline. In practice, developers append query parameters (such as `lightning=lnbc...`) to package options, but this is a **research inference** as it is not standardized in the BIP 21 document itself.
- **Remaining uncertainty**: None on the core spec.

### BIP 321 (Payment Request URI)
- **Primary Source**: [BIP 321 Specification](https://github.com/bitcoin/bips/blob/master/bip-0321.mediawiki)
- **Specification Status**: Complete.
- **What is directly specified**: Standardizes the serialization of one or more payment instructions into a single URI. It defines the path component as a Bitcoin address, and query parameters for payment instructions (`lightning` for BOLT 11, `lno` for BOLT 12, `pay` for BIP 351, `sp` for BIP 352). It allows multiple query parameters with the same key for payment instructions. It specifies the use of `req-` prefix for mandatory parameters (if a client doesn't support a `req-` parameter, it MUST consider the URI invalid).
- **What is not specified**: Selection behavior is **directly specified by a protocol** to be left entirely unspecified. BIP 321 defines formatting and parsing but explicitly leaves selection logic to the client. No ordering, priority, or preference semantics are defined.
- **Correction regarding indexing**: In previous documentation, it was claimed that BIP 321 indexes (e.g., `address.1`, `address.2`, `amount.1`) represent alternative rails. This is **incorrect** and is corrected here as a **directly specified** protocol rule: BIP 321 indexing represents *multiple outputs intended to be paid simultaneously in a single transaction*, not mutually exclusive options. Alternative rails are represented as distinct query parameter keys (e.g., `lno` and `sp`).
- **Known implementation evidence**: Standard libraries like `rust-bitcoin`'s `bip21` crate or Dart payment parsers decode these parameters, but leave selection to the app.
- **Relevance to selection**: It is the primary transport/representation format for multi-instruction payloads.
- **Remaining uncertainty**: How wallets behave when encountering unrecognized query keys prefixed with `req-` vs. ignoring standard unknown keys.

### BIP 353 (DNS Payment Instructions)
- **Primary Source**: [BIP 353 Specification](https://github.com/bitcoin/bips/blob/master/bip-0353.mediawiki)
- **Specification Status**: Active / Draft.
- **What is directly specified**: Resolution of human-readable identifiers (e.g., `user@domain.com`) to standard BIP 21/321 payment URIs via DNS TXT records.
- **What is not specified**: Does not define how the wallet selects between the methods contained in the returned URI.
- **Known implementation evidence**: Resolving DNSSEC-validated TXT records in supporting wallets (e.g., Phoenix).
- **Relevance to selection**: Translates human-readable names to multi-rail URIs.
- **Remaining uncertainty**: The timing correlation vectors between DNS queries and subsequent transaction broadcasts is an **open question**.

### BOLT 11 (Lightning Invoice)
- **Primary Source**: [BOLT 11 Specification](https://github.com/lightning/bolts/blob/master/11-payment-encoding.md)
- **Specification Status**: Complete / Active.
- **What is directly specified**: Serialization of single-use, receiver-generated ephemeral payment requests over the Lightning Network.
- **What is not specified**: Multi-method packaging or fallback logic.
- **Known implementation evidence**: Ubiquitous across Lightning wallets.
- **Relevance to selection**: Represents a primary off-chain payment instruction.
- **Remaining uncertainty**: None.

### BOLT 12 (Lightning Offers)
- **Primary Source**: [BOLT 12 Specification](https://github.com/lightning/bolts/blob/master/12-offer-encoding.md)
- **Specification Status**: Draft / Active.
- **What is directly specified**: Static, reusable payment requests ("offers") that allow a payer to request an ephemeral invoice from the receiver. It specifies the use of onion messages with transient keys for the `invoice_request` flow.
- **What is not specified**: Selection or negotiation between BOLT 12 and on-chain rails.
- **Known implementation evidence**: Core Lightning (CLN) natively supports BOLT 12. Phoenix wallet supports sending to BOLT 12 offers.
- **Sender Privacy**: The protocol specifies that invoice requests use onion routing and transient keys, meaning the payer's node public key is not disclosed to the receiver. This is **directly specified**. However, whether network observers can correlate node identities via timing analysis remains an **open question**.
- **Remaining uncertainty**: Robustness of onion routing paths for asynchronous invoice delivery.

### Payjoin (BIP 78) & Payjoin v2
- **Primary Source**: [BIP 78 Specification](https://github.com/bitcoin/bips/blob/master/bip-0078.mediawiki) | [Payjoin Dev Repo](https://github.com/payjoin/rust-payjoin)
- **Specification Status**: Active / Standard.
- **What is directly specified**: BIP 78 specifies collaborative on-chain transaction construction. It **directly specifies** that if the Payjoin HTTP session fails, the wallet MUST fall back to a standard on-chain transaction. Payjoin v2 specifies asynchronous/receiver-serverless setups using relays (like Nostr or TURN).
- **What is not specified**: Selection between Payjoin and off-chain protocols (like Lightning).
- **Known implementation evidence**: Supported in Sparrow, Wasabi, BTCPay Server, and BlueWallet.
- **Downgrade resistance**: Under BIP 78, falling back to a standard transaction on HTTP failure is **directly specified**. However, the fact that an attacker can block the HTTP endpoint to force a downgrade is a **research inference**.
- **Payjoin v2 framing**: The protocol specifies a receiver-serverless model where the receiver doesn't host an HTTPS endpoint. However, it relies on public relays/TURN proxies to route traffic, which is a **research inference**.
- **Remaining uncertainty**: Relay uptime and vulnerability to message blocking.

### Silent Payments (BIP 352)
- **Primary Source**: [BIP 352 Specification](https://github.com/bitcoin/bips/blob/master/bip-0352.mediawiki)
- **Specification Status**: Complete.
- **What is directly specified**: Non-interactive on-chain addresses that prevent public linkage. Senders generate unique destinations using recipient keys and their own input UTXOs.
- **What is not specified**: Selection between Silent Payments and other methods.
- **Known implementation evidence**: Cake Wallet, Silentium.
- **Relevance to selection**: Highly private on-chain option.
- **Remaining uncertainty**: Computational scan latency on mobile wallets is an **open question**.

### LNURL Pay & Lightning Address
- **Primary Source**: [LNURL Specs (LUDs)](https://github.com/lnurl/luds)
- **Specification Status**: De facto standard.
- **What is directly specified**: Protocol for fetching BOLT 11 invoices via HTTP requests to a web server.
- **What is not specified**: Direct selection or negotiation.
- **Known implementation evidence**: Widely deployed in custodial and non-custodial Lightning wallets (e.g., Wallet of Satoshi, BlueWallet).
- **Privacy implications**: Resolving LNURL leaks IP address and payment metadata to the server. This is a **research inference**.
- **Remaining uncertainty**: Long-term interoperability as BOLT 12 and BIP 353 gain traction.

### Nostr Wallet Connect (NWC)
- **Primary Source**: [NIP-47 Specification](https://github.com/nostr-protocol/nips/blob/master/47.md)
- **Specification Status**: Draft.
- **What is directly specified**: A relay-based protocol utilizing Nostr to send wallet commands (like paying invoices).
- **What is not specified**: Payment instruction selection.
- **Known implementation evidence**: Alby, Mutiny Wallet (historical).
- **Relevance to selection**: Example of a relay-based interactive transport.
- **Remaining uncertainty**: Metadata leakage across public Nostr relays.

---

## Wallet Selection Behaviour (Generalizations Audited)

### Lightning vs On-chain Selection Heuristic
- **Statement**: "Wallets prioritize Lightning over on-chain when both are present."
- **Audit**: This is an **observed behavior** in named implementations (Phoenix and BlueWallet), which automatically load and select the Lightning/BOLT 12 instruction over the on-chain address. However, generalizing this to "most wallets" is an **unverified assumption**.

### Intermediary Parameter Stripping
- **Statement**: "An intermediary can strip parameters to force a downgrade."
- **Audit**: This is a **research inference** based on the fact that BIP 21/321 URIs are often transmitted via QR codes, text, or cleartext protocols, which lack integrity validation unless encapsulated in secure protocols (like TLS or BIP 322 signatures).

### Fallback Mechanics
- **Statement**: "Wallets automatically fall back to on-chain."
- **Audit**: Under BIP 78, fallback to standard on-chain payments upon HTTP failure is **directly specified**. However, fallback from a failed Lightning routing attempt to on-chain is not specified in any standard and is purely a **client-side wallet policy**.

---

## Research Evidence & Confidence Table

| Claim / Behaviour | Evidence Type | Source Reference | Confidence | Notes |
| :--- | :--- | :--- | :--- | :--- |
| BIP 321 indexes (`address.1`) represent multi-output payments, not alternative choices | Directly specified | [BIP 321 Spec](https://github.com/bitcoin/bips/blob/master/bip-0321.mediawiki) | Confirmed by specification | A crucial correction of the previous documentation's framing. |
| BIP 78 requires standard on-chain fallback if the HTTP session fails | Directly specified | [BIP 78 Section: Fallback](https://github.com/bitcoin/bips/blob/master/bip-0078.mediawiki) | Confirmed by specification | Standard protocol behavior. |
| BOLT 12 invoice requests hide the sender's node ID using onion routing transient keys | Directly specified | [BOLT 12 Specification](https://github.com/lightning/bolts/blob/master/12-offer-encoding.md) | Confirmed by specification | Built-in protocol privacy property. |
| Phoenix and BlueWallet default to Lightning when parsing a unified BIP 21 URI | Observed in implementation | Phoenix/BlueWallet client behavior | Confirmed by implementation | Empirically verified in these specific clients. |
| Intermediaries can strip query parameters to execute downgrade attacks | Research inference | Threat model analysis | Reasonable inference | Standard security property of unsigned cleartext payloads. |
| DNS resolution of BIP 353 leaks timing data that correlates to settlement | Research inference | Privacy analysis | Reasonable inference | A known traffic analysis vector. |
| Silent Payments scanning introduces high CPU latency on low-power devices | Research inference | Performance discussions | Reasonable inference | Discussed in BIP 352 implementation reviews. |
| Most wallets in the ecosystem default to prioritizing Lightning over on-chain | Unverified assumption | Ecosystem observation | Unverified | Requires broader implementation matrix data to confirm. |
| Payjoin v2 is completely serverless | Unverified assumption | Payjoin v2 concept | Unverified | Technically incorrect; it is receiver-serverless but requires Nostr/TURN relays. |
