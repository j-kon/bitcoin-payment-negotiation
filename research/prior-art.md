# Prior-Art Review Framework

This document catalogs existing Bitcoin payment standards, protocols, and implementation practices. We examine each standard's capabilities, payment details, and how (or if) it addresses payment-method selection and negotiation.

> **Research Finding:** We have not yet identified a common cross-protocol specification addressing this exact combination of concerns.

---

## Existing Protocols & Standards

### BIP 21 (URI Scheme)
- **Direct Source Link**: [BIP 21 Specification](https://github.com/bitcoin/bips/blob/master/bip-0021.mediawiki)
- **What problem it solves**: Standardizes a URI scheme for designating a Bitcoin destination address, an amount, and optional label/message fields.
- **What payment information it communicates**: A single on-chain address, requested amount, label, and message. It also allows arbitrary query parameters (e.g., `lightning=lnbc...`).
- **Supports multiple methods**: No natively, but in practice, custom query parameters (e.g., adding `lightning=` or `sp=`) are appended to represent multiple payment options within a single string.
- **Defines method selection**: No. It does not provide rules for how a wallet chooses between the base address and the parameter-based methods.
- **Communicates sender preferences**: No.
- **Communicates receiver preferences**: No, though the base address is sometimes assumed to be the primary option.
- **Privacy implications**: Minimal local privacy; reuse of the single on-chain address leaks receiver identity. Adding static lightning parameters can also create tracking vectors.
- **Downgrade considerations**: If a sender doesn't support the optional query parameter, they fall back to the base on-chain address. An intermediary can strip query parameters to force on-chain payment.
- **Relevance to this project**: It is the historical basis for multi-method payment strings.

### BIP 321 (Payment Request URI)
- **Direct Source Link**: [BIP 321 Specification](https://github.com/bitcoin/bips/blob/master/bip-0321.mediawiki)
- **What problem it solves**: Standardizes the serialization of one or more payment instructions (including multiple outputs) into a single URI, extending BIP 21.
- **What payment information it communicates**: Multiple distinct outputs (addresses, amounts), labels, messages, and arbitrary parameter fields across indexed groups.
- **Supports multiple methods**: Yes, via multiple payment instructions serialized in index parameters (e.g., `address`, `address.1`, `address.2`).
- **Defines method selection**: No. The specification focuses on parsing and formatting, leaving selection logic to the client wallet.
- **Communicates sender preferences**: No.
- **Communicates receiver preferences**: No explicit ordering or preference weight is defined by the standard.
- **Privacy implications**: Similar to BIP 21, but carrying multiple addresses allows a receiver to potentially correlate different addresses if they are scanned.
- **Downgrade considerations**: Intermediaries can intercept and strip indexed parameters, potentially removing newer/more private payment methods and forcing the wallet to use the base on-chain address.
- **Relevance to this project**: BIP 321 is the primary vehicle for representing multi-instruction payments in our scope.

### BIP 353 (DNS Payment Instructions)
- **Direct Source Link**: [BIP 353 Specification](https://github.com/bitcoin/bips/blob/master/bip-0353.mediawiki)
- **What problem it solves**: Resolves human-readable identifiers (e.g., `user@domain.com`) to standard Bitcoin payment URIs via DNS TXT records.
- **What payment information it communicates**: A BIP 21 or BIP 321 URI containing the actual payment instructions (on-chain addresses, BOLT 12 offers, Silent Payments).
- **Supports multiple methods**: Indirectly, by returning a URI that may contain multiple methods.
- **Defines method selection**: No. It defers all selection logic to the wallet parsing the returned URI.
- **Communicates sender preferences**: No.
- **Communicates receiver preferences**: No.
- **Privacy implications**: Resolving via DNS can leak the payer's IP address and query target to the DNS resolver or name servers, though DNSSEC, DoH/DoT mitigate some third-party eavesdropping.
- **Downgrade considerations**: A malicious DNS server or resolver can manipulate the TXT record to remove specific payment options.
- **Relevance to this project**: BIP 353 represents a key discovery mechanism for multi-instruction payment payloads.

### BOLT 11 (Lightning Invoice)
- **Direct Source Link**: [BOLT 11 Specification](https://github.com/lightning/bolts/blob/master/11-payment-encoding.md)
- **What problem it solves**: Standardizes payment requests over the Lightning Network, securing payments using hashes of preimages.
- **What payment information it communicates**: Node ID, payment hash, amount, expiry, description, and routing hints.
- **Supports multiple methods**: No. It represents a single, ephemeral Lightning invoice.
- **Defines method selection**: N/A (single method).
- **Communicates sender preferences**: N/A.
- **Communicates receiver preferences**: N/A.
- **Privacy implications**: Reveals the receiver's node ID (for public nodes) or path hints (revealing private channel peers).
- **Downgrade considerations**: N/A.
- **Relevance to this project**: A primary off-chain payment instruction.

### BOLT 12 (Lightning Offers)
- **Direct Source Link**: [BOLT 12 Specification](https://github.com/lightning/bolts/blob/master/12-offer-encoding.md)
- **What problem it solves**: Introduces reusable, static payment requests ("offers") that allow payers to request a custom, ephemeral invoice directly from the receiver over the Lightning Network.
- **What payment information it communicates**: Node ID or public key, supported chains, payment descriptions, and recurrence info.
- **Supports multiple methods**: No. It is specific to the BOLT 12 interactive invoice acquisition flow.
- **Defines method selection**: N/A.
- **Communicates sender preferences**: No.
- **Communicates receiver preferences**: No.
- **Privacy implications**: More private than BOLT 11 as invoices are fetched dynamically via onion routing, hiding the sender's node ID.
- **Downgrade considerations**: If the interactive path fails, the wallet has no direct fallback unless other instructions are bundled.
- **Relevance to this project**: A major interactive off-chain payment instruction.

### Payjoin (BIP 78) & Payjoin v2
- **Direct Source Links**: [BIP 78 Specification](https://github.com/bitcoin/bips/blob/master/bip-0078.mediawiki) | [Payjoin Dev Repo](https://github.com/payjoin/rust-payjoin)
- **What problem it solves**: Mitigates blockchain analysis heuristics by facilitating collaborative, privacy-enhancing on-chain transactions where both sender and receiver contribute inputs. Payjoin v2 extends this to asynchronous and serverless environments.
- **What payment information it communicates**: Endpoint URLs for negotiation, payment amounts, and accepted transaction parameters.
- **Supports multiple methods**: No. It is an on-chain collaboration protocol.
- **Defines method selection**: N/A.
- **Communicates sender preferences**: No.
- **Communicates receiver preferences**: No.
- **Privacy implications**: Greatly improves on-chain privacy by breaking the common-input ownership heuristic. However, the sender must communicate directly with the receiver's server, leaking the sender's IP address (unless routed through Tor/VPN).
- **Downgrade considerations**: BIP 78 specifies a fallback to standard on-chain payments if the HTTP endpoint fails to respond or errors. A network attacker can block the Payjoin endpoint to force a downgrade to standard on-chain.
- **Relevance to this project**: Represents a high-privacy, interactive on-chain payment method.

### Silent Payments (BIP 352)
- **Direct Source Link**: [BIP 352 Specification](https://github.com/bitcoin/bips/blob/master/bip-0352.mediawiki)
- **What problem it solves**: Standardizes static, reusable on-chain donation addresses that prevent blockchain observers from linking transactions to the recipient's identity.
- **What payment information it communicates**: Scan public key and spend public key.
- **Supports multiple methods**: No.
- **Defines method selection**: N/A.
- **Communicates sender preferences**: N/A.
- **Communicates receiver preferences**: N/A.
- **Privacy implications**: Excellent on-chain privacy for the receiver. The sender must perform Elliptic Curve Diffie-Hellman (ECDH) calculations using their transaction inputs to generate the unique destination script.
- **Downgrade considerations**: If a wallet does not support Silent Payments, it falls back to other methods present in a URI, exposing the receiver's standard addresses to linkage if those are used instead.
- **Relevance to this project**: Represents a high-privacy, non-interactive on-chain payment method.

### LNURL Pay & Lightning Address
- **Direct Source Links**: [LNURL Specification](https://github.com/lnurl/luds) | [Lightning Address](https://lightningaddress.com/)
- **What problem it solves**: Uses HTTP requests to dynamically fetch BOLT 11 invoices, allowing static QR codes and email-like identifiers to represent Lightning destinations.
- **What payment information it communicates**: HTTP endpoints, min/max sendable amounts, metadata, and success actions.
- **Supports multiple methods**: No. It is designed to yield a BOLT 11 invoice.
- **Defines method selection**: N/A.
- **Communicates sender preferences**: No.
- **Communicates receiver preferences**: No.
- **Privacy implications**: The sender makes an HTTP request to the receiver's web server, leaking IP addresses, timing info, and payment metadata to the server operator.
- **Downgrade considerations**: If the server is offline, the payment fails immediately.
- **Relevance to this project**: A widely deployed interactive off-chain resolution mechanism.

### Nostr Wallet Connect (NWC)
- **Direct Source Link**: [NIP-47 Specification](https://github.com/nostr-protocol/nips/blob/master/47.md)
- **What problem it solves**: Standardizes how external applications (clients) communicate commands (like sending or receiving payments) to a user's wallet via the Nostr relay network.
- **What payment information it communicates**: Connection URIs, commands (pay_invoice, make_invoice), and execution statuses.
- **Supports multiple methods**: No. It is a control protocol, not a direct payment request.
- **Defines method selection**: N/A.
- **Communicates sender preferences**: N/A.
- **Communicates receiver preferences**: N/A.
- **Privacy implications**: Commands travel through Nostr relays; encryption protects payload contents, but relay metadata (IPs, timing, public keys) may be visible.
- **Downgrade considerations**: N/A.
- **Relevance to this project**: Illustrates a message-based transport model that could theoretically carry negotiation messages.

---

## Unified QR & Multi-Payment Libraries

### Multi-Payment QR Approaches
Many wallets generate unified QR codes containing a BIP 21 URI with appended parameters (e.g., `bitcoin:address?lightning=lnbc...`). 
- **Selection Behaviour**: Most wallets prioritize the Lightning parameter if supported, falling back to on-chain only if the Lightning invoice fails to parse, is expired, or if the wallet has insufficient channel liquidity.
- **Coordination**: There is no standard coordination; some wallets automatically pay the Lightning invoice without presenting choices, while others ask the user.

### Payment Parsing Libraries
Libraries like `bip21` (JavaScript), `rust-bitcoin`'s BIP 21 parser, and similar Dart/Flutter libraries decode URIs but do not enforce selection policies. They expose parsed key-value maps to the wallet application layer, shifting the entire selection burden to custom application-level logic.

---

## Evidence That Would Invalidate the Research Hypothesis

Our working hypothesis is that *defining an interoperable framework for payment-method selection and negotiation is necessary to prevent fingerprinting and downgrade attacks.*

The following evidence would invalidate this hypothesis:
1. **Existing Unified Specifications**: The discovery of an active, peer-reviewed standard that already defines a secure, private, cross-protocol capability negotiation mechanism for Bitcoin.
2. **Universal Client-Side Sufficiency**: Empirical proof that local-only wallet heuristics (e.g., choosing purely based on fee/privacy thresholds without receiver interaction or capability disclosure) can prevent downgrade attacks and fingerprinting without any coordination.
3. **Impossibility of Private Selection**: Mathematical or cryptographic proof showing that any form of capability negotiation or selection inevitably leaks more identity metrics (fingerprinting) than a basic static fallback scheme.
