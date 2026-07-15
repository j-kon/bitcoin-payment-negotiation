# Initial Research Findings

This document summarizes our initial findings, audits the validity of our core hypothesis, and identifies areas requiring further empirical evidence.

---

## 1. What Existing Standards Clearly Solve

Existing standards already provide robust, extensible mechanisms for representing and resolving payment instructions:
- **Representation & Packaging**: [BIP 321](prior-art.md) defines an extensible syntax for packaging multiple payment instructions (e.g., `lightning`, `lno`, `sp`, and `pay`) within a single URI or QR code.
- **Required Prefixes**: BIP 321's `req-` prefix standardizes backward compatibility. If a client does not understand a query parameter prefixed with `req-`, it must reject the entire URI.
- **Identifier Resolution**: [BIP 353](prior-art.md) standardizes how human-readable identifiers resolve to `bitcoin:` payment instructions.

---

## 2. What Remains Implementation-Specific

The following behaviors are not standardized by any protocol and are left entirely to individual wallet applications:
- **Selection Heuristics**: The logic a wallet uses to choose a payment instruction when multiple compatible rails are offered.
- **Interactive Retry & Timeout Policies**: How long a wallet waits for an interactive handshake (such as BOLT 12 invoice requests or Payjoin sessions) before aborting or reverting to a fallback.
- **User Interface Choices**: Whether to present the user with a list of available payment options and fees, or silently execute the default choice.
- **Failure Handlers**: How the wallet informs the user when a preferred, high-privacy method (like Payjoin) fails. We have not found a general cross-method policy in the specifications surveyed so far.

---

## 3. What Evidence is Still Missing

To draw firm conclusions, we require empirical data in the following areas:
- **Comprehensive Wallet Behavior Mapping**: Authoritative documentation on the default selection heuristics of major wallets.
- **Mempool-Responsive Selection**: Observations on whether wallets dynamically alter their selection defaults based on current network congestion and transaction fees.
- **Error-Handling Edge Cases**: How wallets handle malformed, duplicate, or expired query parameters in BIP 321 payloads.

---

## 4. How the Original Hypothesis has Weakened

Our initial hypothesis was that *defining a new interoperable payment-method selection and representation standard was necessary.*

This hypothesis has weakened in the following ways:
- **No Representation Gap**: BIP 321 already provides a fully functional, standard method for packaging multiple rails in a single payload. No new standard is needed to represent multi-method instructions.
- **Unverified Need for Policy Standardization**: It is not yet clear whether differing local wallet selection policies cause actual user failures or economic harm. If wallets implement robust local policies, coordination may be entirely unnecessary.
- **Fallback Verification**: BIP 78 requires the original PSBT to be broadcastable. Automatic fallback behavior after negotiation failure has not yet been established as a cross-wallet standard and requires implementation-level evidence. BIP 77 explicitly permits either party to choose the fallback transaction during an Async Payjoin session.

---

## 5. Strongest Remaining Research Questions

Rather than focusing on establishing a new standard, our remaining research focuses on these core questions:
1. How do named wallet implementations behave when a BIP 321 URI contains multiple supported instructions?
2. Do different choices produce concrete payment failures, privacy regressions, unexpected fees, or confusing user experiences?
3. Which risks belong to request integrity and authenticated delivery rather than payment-method selection?
4. Can useful interoperability tests be created without standardizing wallet policy?
5. Is interactive negotiation necessary at all?

---

## Conclusion

We have not yet found an interoperable specification for payment-method selection policy, but additional implementation research is required before determining whether standardization would be beneficial.
