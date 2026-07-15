# Payment Selection Scenarios

This document maps out concrete user and network scenarios where a wallet must select between multiple payment instructions. These scenarios are intended to analyze trade-offs and clarify edge cases; they do not prescribe a final algorithm.

---

## Scenario 1: On-Chain vs. BOLT 12

- **Sender Capabilities**: On-Chain, Lightning (BOLT 12).
- **Receiver Instructions**: 
  - Standard On-chain Address
  - BOLT 12 Offer
- **Environmental Conditions**: 
  - On-chain mempool is heavily congested (medium-priority fee is 150 sat/vB).
  - Payer has active, well-funded Lightning channels.
- **Possible Choices**: 
  - Pay instantly via Lightning (BOLT 12 path).
  - Construct and broadcast an on-chain transaction (on-chain path).
- **Security & Privacy Concerns**: Paying via BOLT 12 requires fetching an invoice from the receiver, which reveals that the sender is active on the network. On-chain payment leaks inputs and UTXO history but can be broadcast asynchronously.
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
- **Security & Privacy Concerns**: Payjoin significantly improves privacy by breaking common-input heuristics. However, querying the Payjoin endpoint requires an HTTP request to the receiver's server, disclosing the sender's IP address.
- **Open Design Question**: If the Payjoin endpoint times out or returns an error, should the wallet automatically fallback to standard on-chain, or warn the user that they are about to lose Payjoin privacy?

---

## Scenario 3: Silent Payments vs. Standard Address

- **Sender Capabilities**: On-Chain (standard), Silent Payments (BIP 352).
- **Receiver Instructions**: 
  - Standard On-chain Address
  - Silent Payments Address (Scan Key & Spend Key)
- **Environmental Conditions**: 
  - Mempool fees are normal.
  - Both addresses are present in a single BIP 321 payload.
- **Possible Choices**: 
  - Compute the Silent Payment destination script and send.
  - Send directly to the standard address.
- **Security & Privacy Concerns**: Silent Payments prevent address reuse and blockchain linkage for the receiver. However, generating a Silent Payment destination script requires the sender to perform elliptic curve operations using their selected input UTXOs.
- **Open Design Question**: If a wallet supports Silent Payments, is there ever a valid reason to prefer the standard address fallback, other than minor computational savings?

---

## Scenario 4: Receiver Temporarily Offline

- **Sender Capabilities**: On-Chain, Lightning (BOLT 12).
- **Receiver Instructions**: 
  - Standard On-chain Address
  - BOLT 12 Offer
- **Environmental Conditions**: 
  - The receiver's Lightning node is currently offline or unreachable.
  - Payer wants to complete the checkout immediately.
- **Possible Choices**: 
  - Wait and retry the BOLT 12 path.
  - Fall back to the non-interactive standard on-chain address.
- **Security & Privacy Concerns**: Falling back to on-chain allows the payment to succeed asynchronously, but exposes the recipient to address reuse and both parties to higher fees.
- **Open Design Question**: How long should a wallet wait/retry an interactive method before proposing a non-interactive fallback to the user?

---

## Scenario 5: Lightning Liquidity Unavailable

- **Sender Capabilities**: On-Chain, Lightning (BOLT 12).
- **Receiver Instructions**: 
  - Standard On-chain Address
  - BOLT 12 Offer
- **Environmental Conditions**: 
  - The payment amount is $500.
  - Payer's local outbound channels have a maximum single-payment capacity of $200.
- **Possible Choices**: 
  - Split the payment (Multi-Path Payments - MPP) if supported by the receiver.
  - Fall back to the on-chain address.
- **Security & Privacy Concerns**: Attempting MPP may fail mid-way, locking up partial liquidity temporarily. On-chain fallback guarantees settlement but requires waiting for blocks.
- **Open Design Question**: Should the wallet check local channel liquidity *before* requesting a BOLT 12 invoice, or only after the invoice request fails?

---

## Scenario 6: Payment Instruction Removed by an Intermediary

- **Sender Capabilities**: On-Chain, Lightning (BOLT 12).
- **Receiver Instructions**: 
  - Standard On-chain Address (originally had a BOLT 12 offer, but it was stripped by a proxy server).
- **Environmental Conditions**: 
  - A malicious proxy in the middle of a public Wi-Fi network stripped the `lightning` query parameter from the BIP 21 string.
- **Possible Choices**: 
  - Pay the standard on-chain address.
  - Refuse to pay due to mismatch or suspicious lack of options.
- **Security & Privacy Concerns**: An active downgrade attack. The payer is forced to pay on-chain, incurring higher fees and leaking UTXO data.
- **Open Design Question**: Can we implement a light check (e.g., verifying hash commitments of the original payload) to detect when options have been stripped?

---

## Scenario 7: Privacy-Focused Sender

- **Sender Capabilities**: On-Chain, Silent Payments, Payjoin.
- **Receiver Instructions**: 
  - Standard On-chain Address
  - Payjoin Endpoint
  - Silent Payments Address
- **Environmental Conditions**: 
  - Sender has configured their wallet settings to "Maximum Privacy".
- **Possible Choices**: 
  - Use the Silent Payments address (non-interactive, private).
  - Use the Payjoin endpoint (interactive, private).
- **Security & Privacy Concerns**: Silent Payments leak no receiver address data on-chain and require no network handshake. Payjoin breaks input heuristics but requires HTTP network calls.
- **Open Design Question**: For a privacy-focused user, should the wallet default to Silent Payments (avoiding network IP leaks) or Payjoin (breaking input heuristics)?

---

## Scenario 8: Fee-Sensitive Sender

- **Sender Capabilities**: On-Chain, Lightning (BOLT 12).
- **Receiver Instructions**: 
  - Standard On-chain Address
  - BOLT 12 Offer
- **Environmental Conditions**: 
  - On-chain mempool fees are extremely high.
  - Routing fees on Lightning for this path are estimated at 0.5%.
- **Possible Choices**: 
  - Choose BOLT 12 to minimize fee expense.
  - Choose On-chain.
- **Security & Privacy Concerns**: Minimizing fees leads to using Lightning, which is highly private but reveals active network participation.
- **Open Design Question**: What is the threshold ratio of (On-chain Fee / Lightning Routing Fee) at which a wallet should recommend Lightning over on-chain?

---

## Scenario 9: Merchant Requiring Rapid Settlement

- **Sender Capabilities**: On-Chain, Lightning (BOLT 12).
- **Receiver Instructions**: 
  - Standard On-chain Address
  - BOLT 12 Offer
- **Environmental Conditions**: 
  - A physical point-of-sale terminal at a coffee shop.
  - The customer is waiting to leave with their item.
- **Possible Choices**: 
  - Select Lightning (near-instant settlement).
  - Select On-chain (merchant must wait for confirmations or accept zero-conf risk).
- **Security & Privacy Concerns**: Zero-conf on-chain payments are vulnerable to double-spending, while waiting for 1 confirmation is impractical for retail.
- **Open Design Question**: Should the payment request payload include a field indicating "immediate settlement required," disabling the on-chain option unless zero-conf is supported?

---

## Scenario 10: Unsupported Experimental Payment Method

- **Sender Capabilities**: On-Chain, BOLT 12.
- **Receiver Instructions**: 
  - Standard On-chain Address
  - Experimental Cashu Mint Proof token request parameter
- **Environmental Conditions**: 
  - The receiver has added a new, experimental protocol parameter to their URI.
  - The sender's wallet does not recognize this parameter.
- **Possible Choices**: 
  - Ignore the experimental parameter and parse/execute the known methods (on-chain or BOLT 12).
  - Fail the payment entirely.
- **Security & Privacy Concerns**: Parsing must be robust. Unknown parameters must not crash the wallet or lead to invalid state execution.
- **Open Design Question**: How do we ensure forward compatibility in payment parsing libraries when handling new, unrecognized parameters?
