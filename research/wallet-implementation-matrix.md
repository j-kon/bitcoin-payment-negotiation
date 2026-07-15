# Wallet Implementation Matrix

This matrix maps out supported payment methods and selection behaviors across various Bitcoin wallets. 

To maintain scientific integrity:
- We only populate cells where **authoritative evidence** (wallet source code, official release notes, or empirical tests) is available.
- All other cells are marked as **Unknown** or **Not yet investigated**.
- We invite contributors to help fill this matrix by opening issues with supporting references.

---

## Matrix

| Wallet | On-chain | BOLT 11 | BOLT 12 | BIP 321 | Payjoin | Silent Payments | Multi-method selection behaviour | Evidence |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Phoenix** | Confirmed | Confirmed | Confirmed | Confirmed | Unsupported | Unsupported | Automatically prioritizes Lightning / BOLT 12 over on-chain when both are offered in a unified URI. | Phoenix v2.x documentation & client tests. |
| **Sparrow** | Confirmed | Unsupported | Unsupported | Confirmed | Confirmed | Confirmed | Defaults to Payjoin if the URI contains a Payjoin endpoint; otherwise defaults to Silent Payments or standard on-chain based on address. User can manually choose. | Sparrow release notes and user manual. |
| **BlueWallet** | Confirmed | Confirmed | Unsupported | Confirmed | Confirmed | Unsupported | Automatically selects Lightning over on-chain if a `lightning` query parameter is present in the URI. | BlueWallet user guide and parsing code. |
| **Nunchuk** | Confirmed | Unsupported | Unsupported | Confirmed | Unsupported | Confirmed | Defaults to standard on-chain, but parses and allows sending to BIP 352 Silent Payment destinations. | Nunchuk release notes. |
| **Zeus** | Confirmed | Confirmed | Unknown | Unknown | Unknown | Unknown | Not yet investigated | Not yet investigated. |
| **Blockstream Green** | Confirmed | Unknown | Unknown | Unknown | Unknown | Unknown | Not yet investigated | Not yet investigated. |
| **Muun** | Confirmed | Confirmed | Unknown | Unknown | Unknown | Unknown | Not yet investigated | Not yet investigated. |
