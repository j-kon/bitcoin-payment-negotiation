# Problem Statement

As the Bitcoin ecosystem scales, wallets are transitioning from supporting a single, uniform transaction style to coordinating a diverse set of payment workflows. This document structures the core research questions and hypotheses regarding this transition.

---

## Hypotheses Under Investigation

We are currently investigating several working hypotheses regarding multi-method selection:

1. **User Experience Impact**: We hypothesize that inconsistent wallet selection policies may cause poor user outcomes, such as transaction failures, high fees, or unexpected payment delays. This remains to be verified by empirical testing.
2. **Capability Disclosure & Fingerprinting**: We hypothesize that capability disclosure during payment negotiation may be common and could create unique fingerprinting vectors. We must survey existing wallets to determine what details are actively shared.
3. **Downgrade Attacks**: We hypothesize that an active attacker controlling the request source or delivery channel can strip preferred, privacy-enhancing parameters (like Silent Payments or BOLT 12) from payment requests. However, we note that parameter stripping requires an attacker capable of modifying the request at its source or along its delivery channel. A static QR code scanned directly from a physical source presents a different threat profile from an unauthenticated web response, a compromised clipboard, a replaced physical QR sticker, or a malicious host.
4. **Need for Shared Policy**: We are testing the hypothesis that a shared, standardized selection policy is required to prevent these issues, vs. the alternative hypothesis that local client-side heuristics are sufficient.

---

## The Selection Dilemma

Modern Bitcoin payment requests increasingly package multiple compatible options to ensure the payment can be completed under varying conditions. Simple compatibility does not dictate the *most appropriate* option. For instance, if both wallets support standard on-chain payments and BOLT 12, the transaction could proceed via either.

Deciding which instruction to execute is a multi-dimensional problem. Senders must evaluate the following trade-offs in hypothetical scenarios:
- **Privacy**: Does the method reuse addresses, leak input ownership, or link network identity (IP addresses)?
- **Fees**: What are the estimated mining fees vs. Lightning routing fees?
- **Interactivity**: Does the method require real-time communication (e.g., Payjoin, BOLT 12 invoice requests) or is it non-interactive?
- **Expiry**: How long are the individual instructions valid?
- **Liquidity**: Does the sender have sufficient outbound channel capacity? Does the receiver have inbound capacity?
- **Receiver Availability**: Must the receiver be online at the time of payment?
- **Settlement Properties**: Does the method settle instantly, or does it require blockchain confirmations?
- **Custody Assumptions**: What trust profile is associated with each option?

---

## Practical Example

Consider the following scenario where both sender and receiver support multiple overlapping options:

### Sender Wallet Capabilities:
- Standard On-chain
- BOLT 12 (Lightning)
- Payjoin (Collaborative On-chain)
- Silent Payments (Private On-chain)

### Receiver Payment Request:
- Standard On-chain Address
- BOLT 12 Offer

### Overlap & Selection Decision:
Both wallets are compatible with **Standard On-chain** and **BOLT 12**. 

Despite this compatibility, the sender's wallet still must decide:
- Should it automatically prefer the BOLT 12 path to minimize fees and speed up settlement? If it attempts to fetch a BOLT 12 invoice, it initiates an interactive onion-message flow whose timing and transport metadata may require analysis, while BOLT 12 is designed not to require disclosure of the payer’s ordinary node identity.
- Should it fall back to standard on-chain because the payment amount exceeds available aggregate outbound liquidity or configured wallet limits?
- If the BOLT 12 connection fails, should it silently execute the on-chain payment, or alert the user that a more private option failed?

---

## Falsifiable Research Questions

Rather than proposing a specific specification or protocol, this project aims to answer the following research questions:

1. **Is a standardized method selection policy necessary?** Can we demonstrate concrete cases where uncoordinated, local wallet policies lead to user failure rates, high fees, or privacy leaks that cannot be resolved individually?
2. **Does capability disclosure during negotiation introduce unique, measurable fingerprinting vectors?** Can an observer or receiver identify a wallet's brand or version based purely on how it negotiates or selects a payment method?
3. **Can a payment-request signature fully mitigate payment-method downgrade attacks in non-interactive (offline) contexts?**
4. **Is it possible for a sender's wallet to execute private, local selection without revealing any subset of its unsupported capabilities to the receiver?**
