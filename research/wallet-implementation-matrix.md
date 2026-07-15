# Wallet Implementation Matrix

This matrix maps out supported payment methods and selection behaviors across various Bitcoin wallets.

> [!IMPORTANT]
> **Evidence Policy**: Absence of evidence is not evidence of non-support. A feature is marked Unsupported only when official documentation or source code explicitly establishes non-support.

---

## Active Investigations

| Wallet | On-chain | BOLT 11 | BOLT 12 | BIP 321 | Payjoin | Silent Payments | Multi-method selection behaviour | Evidence strength | Evidence / Source Links |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Phoenix** | Confirmed | Confirmed | Confirmed | Confirmed | Unknown | Unknown | Parser preserves the on-chain address and optional BOLT 11/BOLT 12 instruction. The inspected SendManager returns a URI result when the on-chain address is present. Final UI selection behavior has not yet been fully traced. | Source-code trace | [Phoenix Parser.kt](https://github.com/ACINQ/phoenix/blob/3749b0817d60b0a17a97d21338e2d9038efb2d9e/phoenix-shared/src/commonMain/kotlin/fr.acinq.phoenix/utils/Parser.kt) and [SendManager.kt](https://github.com/ACINQ/phoenix/blob/3749b0817d60b0a17a97d21338e2d9038efb2d9e/phoenix-shared/src/commonMain/kotlin/fr.acinq.phoenix/managers/SendManager.kt) |
| **BlueWallet** | Confirmed | Confirmed | Unknown | Confirmed | Confirmed | Unknown | When a combined Bitcoin and Lightning request is detected, BlueWallet opens wallet selection. The selected wallet’s chain determines whether the on-chain or Lightning route is used. | Source-code trace | [BlueWallet deeplink-schema-match.ts](https://github.com/BlueWallet/BlueWallet/blob/acf9cd068572c3712f8f409ce8a19aaca689a7e0/class/deeplink-schema-match.ts) |

---

## Pending Investigation

The following wallets require further research and code traces to verify their exact support matrices and selection policies. Cells are marked **Unknown** per the evidence policy until primary code links or specifications are mapped.

| Wallet | On-chain | BOLT 11 | BOLT 12 | BIP 321 | Payjoin | Silent Payments | Multi-method selection behaviour | Evidence strength |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Sparrow** | Unknown | Unknown | Unknown | Unknown | Unknown | Unknown | Unknown | Unverified |
| **Nunchuk** | Unknown | Unknown | Unknown | Unknown | Unknown | Unknown | Unknown | Unverified |
| **Zeus** | Unknown | Unknown | Unknown | Unknown | Unknown | Unknown | Unknown | Unverified |
| **Blockstream Green** | Unknown | Unknown | Unknown | Unknown | Unknown | Unknown | Unknown | Unverified |
| **Muun** | Unknown | Unknown | Unknown | Unknown | Unknown | Unknown | Unknown | Unverified |
