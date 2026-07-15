# Terminology

This document defines core concepts and terms used throughout this research project. To avoid ambiguity, contributors must apply these terms strictly as defined.

---

## Core Definitions

### Payment Instruction
A specific, self-contained set of parameters directing how to construct and send a transaction (e.g., an on-chain address with an amount, a BOLT 11 invoice, a BOLT 12 offer, or a BIP 352 Silent Payment destination).

### Payment Request
A wrapper, URI, or payload that encapsulates one or more payment instructions. A [BIP 321](prior-art.md) URI is an example of a payment request that can carry multiple payment instructions.

### Payment Method
The protocol or rail through which the payment will be routed (e.g., Bitcoin On-Chain, Lightning Network, Liquid Network).

### Payment Workflow
The sequence of operational steps required by a payer and receiver to complete a transaction (e.g., scanning a static QR code, requesting an ephemeral invoice over Lightning, or collaborating on a joint transaction).

### Capability
A feature, protocol, or standard supported by a wallet's software (e.g., the ability to parse BIP 352 Silent Payments, resolve BIP 353 DNS names, or execute BIP 78 Payjoin).

### Preference
A soft priority order expressed by a payer or receiver regarding which payment method they *prefer* to use (e.g., preferring Lightning over on-chain due to lower fees).

### Requirement
A hard constraint expressed by a payer or receiver. If a payment request contains a requirement the other party does not support, the transaction cannot proceed.

### Policy
A set of local rules embedded in a wallet's software or configured by a user to evaluate trade-offs and select a payment instruction (e.g., "always use Lightning if fee is < 1% and amount is < $50").

### Compatibility
The state where a sender's capabilities overlap with the receiver's provided payment instructions, allowing at least one payment method to be executed.

---

## Operations and Processes

### Selection
The autonomous, local decision-making process performed by the sender's wallet to choose a single payment instruction from a set of options already provided in the payment request. Selection does **not** involve active back-and-forth communication with the receiver at the time the choice is made.

### Negotiation
An interactive protocol where the sender and receiver actively communicate (exchange messages or metadata) to agree on a mutually supported payment method.

> [!IMPORTANT]
> **Do not use "Selection" and "Negotiation" interchangeably.**
> - **Selection** is local and unidirectional (performed by the payer).
> - **Negotiation** is interactive and bidirectional (requiring round-trip communication).

### Fallback
An alternative payment instruction designated to be used if the preferred payment instruction cannot be completed or resolved (e.g., falling back to a standard on-chain address if the offer cannot be resolved or the invoice-request flow times out).

### Downgrade Attack
An exploit where a malicious intermediary or observer tampers with a payment request to remove preferred or higher-privacy payment options (e.g., stripping a Silent Payments or Lightning instruction to force a standard, linkable on-chain transaction).

### Wallet Fingerprinting
The tracking or identification of a user's wallet software by analyzing the specific subset of optional capabilities, parameters, or heuristics it discloses during a payment request or negotiation flow.

### Interactive Payment
A payment workflow requiring additional communication between the payer and receiver to construct the transaction, which may be synchronous (e.g. BIP 78 Payjoin) or asynchronous (e.g. BIP 77 Async Payjoin or BOLT 12 invoice requests).

### Non-Interactive Payment
A payment workflow that can be executed entirely by the payer using static information provided by the receiver, without real-time communication (e.g., standard on-chain address or BIP 352 Silent Payments).

### Settlement
The finality state of a transaction where the funds are considered settled and secure under the specific economic and protocol assumptions of that payment method (rather than calling Bitcoin payments absolutely irreversible).

### Custody Assumption
The trust profile associated with a transaction. Custody assumptions may depend on the specific wallet or service implementation, not only the encoded payment method itself (e.g. a self-custodial Lightning wallet vs. a custodial Lightning account).

### Receiver Availability
The requirement for the receiver to be online or active to process or sign a transaction (e.g., interactive Lightning nodes vs. offline static on-chain addresses).
