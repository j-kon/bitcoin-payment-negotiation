# Initial Research Findings

This document summarizes our initial findings, audits the validity of our core hypothesis, and identifies areas requiring further empirical evidence.

---

## 1. What Existing Standards Clearly Solve

Existing standards already provide robust, extensible mechanisms for representing and resolving payment instructions:
- **Representation & Packaging**: [BIP 321](prior-art.md) defines an extensible syntax for packaging multiple payment instructions (e.g., `lightning`, `lno`, `sp`, and `pay`) within a single URI or QR code. 
- **Required Prefixes**: BIP 321's `req-` prefix standardizes backward compatibility. If a client does not understand a query parameter prefixed with `req-`, it must reject the entire URI.
- **Identifier Resolution**: [BIP 353](prior-art.md) standardizes how human-readable identifiers resolve to BIP 21/321 payment URIs via DNS TXT records.
- **On-chain Fallbacks**: BIP 78 (Payjoin) directly specifies standard fallback to standard on-chain transactions if interactive negotiation fails.

---

## 2. What Remains Implementation-Specific

The following behaviors are not standardized by any protocol and are left entirely to individual wallet applications:
- **Selection Heuristics**: The logic a wallet uses to choose a payment instruction when multiple compatible rails are offered (e.g., whether to default to Lightning, Silent Payments, or standard on-chain).
- **Interactive Retry & Timeout Policies**: How long a wallet waits for an interactive handshake (such as BOLT 12 invoice requests or Payjoin sessions) before aborting or reverting to a fallback.
- **User Interface Choices**: Whether to present the user with a list of available payment options and fees, or silently execute the default choice.
- **Failure Handlers**: How the wallet informs the user when a preferred, high-privacy method (like Payjoin) fails and a downgrade to standard on-chain occurs.

---

## 3. What Evidence is Still Missing

To draw firm conclusions, we require empirical data in the following areas:
- **Comprehensive Wallet Behavior Mapping**: Authoritative documentation on the default selection heuristics of major non-custodial and custodial wallets (e.g., Sparrow, Phoenix, Nunchuk, Green).
- **Mempool-Responsive Selection**: Observations on whether wallets dynamically alter their selection defaults (e.g., preferring Lightning over on-chain) based on current network congestion and transaction fees.
- **Error-Handling Edge Cases**: How wallets handle malformed, duplicate, or expired query parameters in BIP 321 payloads.

---

## 4. How the Original Hypothesis has Weakened

Our initial hypothesis was that *defining a new interoperable payment-method selection and representation standard was necessary.*

This hypothesis has weakened in the following ways:
- **No Representation Gap**: BIP 321 already provides a fully functional, standard method for packaging multiple rails (e.g. `lno`, `sp`, `lightning`) in a single payload. No new standard is needed to represent multi-method instructions.
- **Unverified Need for Policy Standardization**: It is not yet clear whether differing local wallet selection policies cause actual user failures or economic harm. If wallets implement robust local policies (e.g., prioritizing low fees and high privacy), coordination may be entirely unnecessary.

---

## 5. Strongest Remaining Research Questions

The core areas of inquiry that remain highly relevant are:
- **Privacy Preservation during Negotiation**: How can wallets execute interactive protocols (like BOLT 12 or Payjoin) without leaking node node IDs, IP addresses, or capability fingerprints to the receiver or network observers?
- **Downgrade Attack Detection**: Can a client reliably detect if an intermediary proxy stripped preferred parameters (such as `lno` or `sp`) from a payment request to force a linkable on-chain payment, without relying on central authorities?

---

## Conclusion

We have not yet found an interoperable specification for payment-method selection policy, but additional implementation research is required before determining whether standardization would be beneficial.
