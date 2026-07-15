# Problem Statement

As the Bitcoin ecosystem scales, wallets are transitioning from supporting a single, uniform transaction style to coordinating a diverse set of payment workflows. This document structures the core problems, challenges, and risks arising from this transition.

---

## The Core Problem

Modern Bitcoin payment requests increasingly package multiple compatible options to ensure the payment can be completed under varying conditions. However, the presence of multiple compatible options introduces a complex selection problem for the payer's wallet:

1. **Expanding Workflows**: Bitcoin wallets now support on-chain addresses, Silent Payments (BIP 352), Payjoin (BIP 78), BOLT 11 invoices, BOLT 12 offers, and HTTP/DNS-based wrappers (LNURL, BIP 353).
2. **Multi-Instruction Payloads**: A single payment request (e.g., a BIP 321 URI) can contain multiple payment instructions representing different methods.
3. **Overlapping Compatibility**: Both the sender's wallet and the receiver's wallet may support multiple overlapping payment methods.
4. **Sufficiency of Compatibility**: Simple compatibility does not dictate the *most appropriate* option. For instance, if both wallets support standard on-chain payments and BOLT 12, the transaction could proceed via either.
5. **Multi-Dimensional Trade-offs**: Deciding which instruction to execute is a multi-dimensional problem. The selection policy must evaluate:
   - **Privacy**: Does the method reuse addresses, leak input ownership, or link network identity (IP addresses)?
   - **Fees**: What are the estimated mining fees vs. Lightning routing fees?
   - **Interactivity**: Does the method require real-time communication (e.g., Payjoin, BOLT 12 invoice requests) or is it non-interactive?
   - **Expiry**: How long are the individual instructions valid?
   - **Liquidity**: Does the sender have sufficient outbound channel capacity? Does the receiver have inbound capacity?
   - **Receiver Availability**: Must the receiver be online at the time of payment?
   - **Settlement Guarantees**: Does the method settle instantly, or does it require blockchain confirmations?
   - **Custody Assumptions**: What trust profile is associated with each option?
6. **Inconsistent User Experiences**: If wallets implement independent, uncoordinated selection policies, users will experience highly inconsistent payment outcomes. One wallet might silently default to a cheap but slow method, while another might choose a fast but expensive or custodial method.
7. **Fingerprinting Risks**: To determine compatibility, wallets may disclose their supported features or negotiate options. This disclosure creates a fingerprinting risk, allowing receivers or network observers to identify the wallet software.
8. **Downgrade Attacks**: If a transaction is routed through an intermediary (e.g., a payment processor or directory), that intermediary might strip privacy-enhancing options from the request to force the payer to use a less-private method.

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

### Overlap & Selection Dilemma:
Both wallets are compatible with **Standard On-chain** and **BOLT 12**. 

Despite this compatibility, the sender's wallet still must decide:
- Should it automatically prefer the BOLT 12 path to minimize fees and speed up settlement, even if doing so reveals the sender's Lightning node details?
- Should it fall back to standard on-chain because the payment amount is large and exceeds Lightning routing safety thresholds?
- If the BOLT 12 connection fails, should it silently execute the on-chain payment, or alert the user that a more private option failed?

---

## Falsifiable Research Questions

Rather than proposing a specific specification or protocol, this project aims to answer the following research questions:

1. **Is a standardized method selection policy necessary?** Can we demonstrate concrete cases where uncoordinated, local wallet policies lead to user failure rates, high fees, or privacy leaks that cannot be resolved individually?
2. **Does capability disclosure during negotiation introduce unique, irreversible fingerprinting vectors?** Can an observer or receiver identify a wallet's brand or version based purely on how it negotiates or selects a payment method?
3. **Can a payment-request signature (e.g., BIP 322 or TLS) fully mitigate payment-method downgrade attacks in non-interactive (offline) contexts?**
4. **Is it possible for a sender's wallet to execute private, local selection without revealing any subset of its unsupported capabilities to the receiver?**
