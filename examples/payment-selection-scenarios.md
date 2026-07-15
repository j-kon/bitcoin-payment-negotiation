# Payment Selection Scenarios

This document maps out concrete user and network scenarios where a wallet must select between multiple payment instructions. These scenarios are intended to analyze trade-offs and clarify edge cases; they do not prescribe a final algorithm.

---

## Scenario 1: On-Chain vs. BOLT 12

- **Sender Capabilities**: On-Chain, Lightning (BOLT 12).
- **Receiver Instructions**:
  - Standard On-chain Address
  - BOLT 12 Offer (using the `lno` parameter)
- **Environmental Conditions**:
  - On-chain mempool is heavily congested (hypothetically 150 sat/vB).
  - Payer has active, well-funded Lightning channels.
- **Possible Choices**:
  - Pay via Lightning (BOLT 12 path).
  - Construct and broadcast an on-chain transaction (on-chain path).
- **Security & Privacy Concerns**: Paying via BOLT 12 requires fetching an invoice from the receiver, which initiates an interactive onion-message flow whose timing and transport metadata may require analysis, while BOLT 12 is designed not to require disclosure of the payer’s ordinary node identity. On-chain payments expose selected inputs and transaction structure, not necessarily a user's complete UTXO history.
- **Open Design Question**: Should the wallet automatically default to BOLT 12 without user interaction when mempool fees exceed a certain threshold?

---

## Scenario 2: On-Chain vs. Payjoin

- **Sender Capabilities**: On-Chain, Payjoin (BIP 78).
- **Receiver Instructions**:
  - Standard On-chain Address
  - Payjoin Endpoint URL (as a BIP 21 parameter)
- **Environmental Conditions**:
  - The payment amount is large.
  - Payer has multiple UTXOs available.
- **Possible Choices**:
  - Initiate a Payjoin session to construct a collaborative transaction.
  - Fall back to standard on-chain payment.
- **Security & Privacy Concerns**: Payjoin is designed to break the common-input-ownership heuristic. The sender must communicate with the receiver's server. Payjoin HTTP communication may use authenticated HTTPS or onion services; IP disclosure depends on the underlying transport.
- **Open Design Question**: If the Payjoin endpoint times out or returns an error, should the wallet automatically fallback to standard on-chain, or warn the user that they are about to lose Payjoin privacy?

---

## Scenario 3: Silent Payments vs. Standard Address

- **Sender Capabilities**: On-Chain (standard), Silent Payments (BIP 352).
- **Receiver Instructions**:
  - Standard On-chain Address
  - Silent Payments Address (Scan Key & Spend Key using the `sp` parameter)
- **Environmental Conditions**:
  - Mempool fees are normal.
  - Both addresses are present in a single BIP 321 payload.
- **Possible Choices**:
  - Compute the Silent Payment destination script and send.
  - Send directly to the standard address.
- **Security & Privacy Concerns**: Silent Payments prevent address reuse and blockchain linkage for the receiver. The resulting outputs are not publicly linkable to the published Silent Payment address under the protocol assumptions. Senders must consider input-eligibility (only certain input types are compatible with key generation), and receivers must continuously scan the blockchain for payments.
- **Open Design Question**: If a wallet supports Silent Payments, is there ever a valid reason to prefer the standard address fallback, other than minor computational savings?

---

## Scenario 4: Receiver Temporarily Offline

- **Sender Capabilities**: On-Chain, Lightning (BOLT 12).
- **Receiver Instructions**:
  - Standard On-chain Address
  - BOLT 12 Offer (using the `lno` parameter)
- **Environmental Conditions**:
  - The receiver's Lightning node is currently offline or unreachable.
  - Payer wants to complete the checkout immediately.
- **Possible Choices**:
  - Wait and retry the BOLT 12 path.
  - Fall back to the non-interactive standard on-chain address.
- **Security & Privacy Concerns**: Falling back to on-chain allows the payment to succeed asynchronously, but exposes the recipient to address reuse and both parties to higher fees. Note that on-chain fallback does not guarantee settlement; it is subject to mempool conditions and block generation.
- **Open Design Question**: How long should a wallet wait/retry an interactive method before proposing a non-interactive fallback to the user?

---

## Scenario 5: Lightning Liquidity Unavailable

- **Sender Capabilities**: On-Chain, Lightning (BOLT 12).
- **Receiver Instructions**:
  - Standard On-chain Address
  - BOLT 12 Offer (using the `lno` parameter)
- **Environmental Conditions**:
  - The payment amount is $500.
  - Payer's local outbound channels have a maximum single-payment capacity of $200, though aggregate outbound liquidity across all channels is $600.
- **Possible Choices**:
  - Split the payment (Multi-Path Payments - MPP) if supported by the receiver.
  - Fall back to the on-chain address.
- **Security & Privacy Concerns**: Attempting MPP may fail mid-way. On-chain fallback does not guarantee settlement (subject to confirmation delays) but avoids Lightning liquidity limits. Senders must distinguish individual channel capacity from aggregate outbound liquidity.
- **Open Design Question**: Should the wallet check local channel liquidity *before* requesting a BOLT 12 invoice, or only after the invoice request fails?

---

## Scenario 6: Payment Instruction Removed by an Intermediary

- **Sender Capabilities**: On-Chain, Lightning (BOLT 12).
- **Receiver Instructions**:
  - Standard On-chain Address (originally had a BOLT 12 offer, but it was stripped).
- **Environmental Conditions**:
  - An attacker who controls an unauthenticated delivery channel, a compromised webpage, a compromised clipboard, a DNS origin, or a physically replaced QR code strips the `lno` query parameter from the BIP 321 string.
- **Possible Choices**:
  - Pay the standard on-chain address.
  - Refuse to pay due to mismatch or suspicious lack of options.
- **Security & Privacy Concerns**: An active downgrade attack. A changed request may induce fallback to a standard on-chain address but cannot force the user to pay.
- **Open Design Question**: Can we implement a light check (e.g., verifying hash commitments of the original payload) to detect when options have been stripped?

---

## Scenario 7: Privacy-Focused Sender

- **Sender Capabilities**: On-Chain, Silent Payments, Payjoin (BIP 77).
- **Receiver Instructions**:
  - Standard On-chain Address
  - Payjoin Endpoint (BIP 77 Async Payjoin)
  - Silent Payments Address (using the `sp` parameter)
- **Environmental Conditions**:
  - Sender has configured their wallet settings to "Maximum Privacy".
- **Possible Choices**:
  - Use the Silent Payments address (non-interactive).
  - Use the Payjoin endpoint (BIP 77 interactive, async).
- **Security & Privacy Concerns**: Silent Payments require no network handshake. BIP 77 Async Payjoin uses OHTTP to route payloads through a directory, which is intended to reduce client-directory linkability.
- **Open Design Question**: For a privacy-focused user, should the wallet default to Silent Payments or Async Payjoin?

---

## Scenario 8: Fee-Sensitive Sender

- **Sender Capabilities**: On-Chain, Lightning (BOLT 12).
- **Receiver Instructions**:
  - Standard On-chain Address
  - BOLT 12 Offer (using the `lno` parameter)
- **Environmental Conditions**:
  - On-chain mempool fees are extremely high.
  - Routing fees on Lightning for this path are estimated at 0.5% (hypothetically).
- **Possible Choices**:
  - Choose BOLT 12 to minimize fee expense.
  - Choose On-chain.
- **Security & Privacy Concerns**: Lightning has different privacy properties from on-chain payments; it should not be called universally private.
- **Open Design Question**: What is the threshold ratio of (On-chain Fee / Lightning Routing Fee) at which a wallet should recommend Lightning over on-chain?

---

## Scenario 9: Merchant Requiring Rapid Settlement

- **Sender Capabilities**: On-Chain, Lightning (BOLT 12).
- **Receiver Instructions**:
  - Standard On-chain Address
  - BOLT 12 Offer (using the `lno` parameter)
- **Environmental Conditions**:
  - A physical point-of-sale terminal at a coffee shop.
  - The customer is waiting to leave with their item.
- **Possible Choices**:
  - Select Lightning for fast payment completion when a viable route exists.
  - Select On-chain (merchant must wait for confirmations or accept zero-conf risk).
- **Security & Privacy Concerns**: Zero-conf on-chain payments are vulnerable to double-spending.
- **Open Design Question**: Should the payment request payload include a field indicating "immediate settlement required"?

---

## Scenario 10: Unsupported Experimental Payment Method

- **Sender Capabilities**: On-Chain, BOLT 12.
- **Receiver Instructions**:
  - Standard On-chain Address
  - Experimental parameter `req-cashu=token` (marked required)
  - Experimental parameter `something-else=value` (marked optional)
- **Environmental Conditions**:
  - The receiver has added unrecognized parameters to the URI.
- **Possible Choices**:
  - The wallet must reject the entire URI because of the unrecognized `req-` prefixed parameter (`req-cashu`).
  - The wallet can safely ignore the unrecognized optional parameter (`something-else`).
- **Security & Privacy Concerns**: Standard BIP 321 rules protect clients from executing transaction structures they do not understand if marked required.
- **Open Design Question**: How do we ensure forward compatibility in payment parsing libraries when handling new, unrecognized parameters?
