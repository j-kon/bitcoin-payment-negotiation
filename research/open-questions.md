# Open Research Questions

This document compiles outstanding questions identified during our exploratory phase. We categorize these queries to guide ongoing investigations and help contributors identify areas where empirical data or analysis is needed.

---

## 1. Standards & Specifications
- **Can existing BIP 321 fields support the required behaviour?** Can we express receiver preferences or requirements using existing URI key-value structures without breaking backward compatibility?
- **Is a new transport mechanism unnecessary?** Can all negotiation and selection occur using existing layers (e.g., DNS, HTTP, Nostr, Lightning onion messages) rather than defining a custom network transport?
- **Should the receiver express preferences, requirements, or both?** How should a payment request distinguish between "I prefer method X but support Y" and "I require method Z"?

## 2. Privacy
- **Can payment selection occur without exposing wallet identity?** How can a wallet select or negotiate a payment method without leaking its software brand, version, or config settings (fingerprinting)?
- **Should wallets communicate explicit capabilities?** Does disclosing a list of supported features to a merchant server during negotiation constitute an unacceptable privacy risk?
- **Can capability disclosure be strictly request-scoped?** If a wallet must disclose capabilities, can that disclosure be limited to the immediate transaction context without leaking broader wallet properties?

## 3. Security & Resilience
- **How should downgrade resistance work?** If an intermediary strips preferred payment instructions (e.g., removing a BOLT 12 offer to force on-chain fallback), how can the client detect this manipulation?
- **Are fallback mechanisms secure against social engineering?** If an interactive payment fails, how can we prevent the wallet from falling back to a fraudulent address provided in the same payload?

## 4. Interoperability
- **Which properties can be compared safely across different protocols?** Can we build a universal translation layer to compare on-chain mining fees directly against Lightning routing fees, or to compare on-chain settlement finality against Lightning instant-locking?
- **How do we reconcile conflicting defaults?** If the sender's wallet defaults to Lightning-first, but the receiver's request defaults to On-chain-first, how is this conflict resolved without round-trips?

## 5. User Experience (UX)
- **Which parts belong in wallet UX rather than a protocol standard?** Should the wallet software automatically select the optimal method based on heuristics, or should the choice be presented to the user?
- **Can protocols with different trust models be represented without misleading users?** For example, if a request bundles a self-custodial on-chain address and a custodial lightning address, how can the wallet represent these differences clearly to a non-technical user?

## 6. Implementation
- **What implementation evidence is needed before proposing a specification?** What benchmarks, simulated environments, or wallet integrations are required to prove that a proposed selection method is viable?
- **What is the CPU and latency overhead of local selection?** For example, does computing Silent Payment outputs for multiple options introduce noticeable delays on low-powered mobile devices?

## 7. Research Validity
- **Is a shared selection standard actually necessary?** Or should method selection remain entirely local to each wallet's proprietary heuristics?
- **Will the market naturally converge on a single dominant payment method?** If Lightning or another protocol becomes the near-universal standard for retail, does the multi-instruction selection problem dissolve?
