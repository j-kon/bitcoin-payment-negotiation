# Bitcoin Payment Instruction Selection Research

> **Exploratory Research Project**
> 
> **Disclaimer:** This repository does not currently propose a BIP or a production protocol. It is intended to document the problem, study prior work, collect implementation experience, and determine whether additional standardization would be useful.
> 
> This repository is investigating whether a new standard is necessary at all. The project may conclude that wallet-specific policy or existing standards are already sufficient.

---

## Overview

Bitcoin Payment Negotiation is an exploratory research project investigating how Bitcoin wallets may privately and safely select between multiple compatible payment instructions. It begins with the multi-instruction model supported by BIP 321 and examines selection policy, interoperability, wallet fingerprinting, downgrade resistance, fees, interactivity, settlement properties, and receiver availability.

## Motivation

As the Bitcoin ecosystem expands, wallets are incorporating a diverse array of payment methods (e.g., standard on-chain, Silent Payments, Payjoin, BOLT 11, BOLT 12, LNURL, etc.). Modern payment requests increasingly package multiple compatible options to maximize the probability of a successful transaction.

However, when a payment request contains more than one instruction that both the sender and receiver support:
- How does the sender's wallet decide which option is the most appropriate?
- How does the wallet balance trade-offs between fee costs, privacy properties, settlement speeds, and trust assumptions?
- Does the process of selecting a method leak identifying information about the wallet's capabilities to the receiver or network observers?

Without coordination or clear guidelines, independent wallet implementations may establish fragmented policies, leading to inconsistent user experiences or vulnerability to protocol downgrade attacks.

## Key Research Question

> **When several valid payment instructions are available, what should remain local wallet policy, and what, if anything, benefits from interoperable guidance?**

## Current Hypothesis

This project investigates whether shared guidance, test vectors, or implementation conventions could reduce measurable problems.

Conversely, we must also test the alternative hypothesis: that local wallet policy, combined with existing specifications (such as BIP 321 and BIP 353), is already sufficient without introducing new protocols or standards.

## Scope

- **Analysis of selection trade-offs**: Investigating privacy, fee estimation, interactivity, and settlement properties.
- **Threat modeling**: Identifying attacker vectors such as wallet capability probing and intermediary downgrade attacks.
- **Prior-art evaluation**: Cataloging existing payment protocols and how they communicate preferences.
- **Scenario mapping**: Documenting common transaction workflows to compare selection algorithms.
- **Implementation feedback**: Surveying how existing Bitcoin wallets handle multi-payment instructions.

## Non-Goals

- Proposing a new Bitcoin layer (Layer 2 or Layer 3).
- Replacing or deprecating [BIP 321](https://github.com/bitcoin/bips/blob/master/bip-0321.mediawiki).
- Creating a utility token or cryptographic asset.
- Mandating a single universal selection algorithm for all wallets.
- Establishing branding, marketing materials, or promotional campaigns.

## Key Distinction: Representation vs. Decision

It is important to separate the transport/representation of payment options from the decision-making process:
1. **Payment Instructions Representation**: Protocols like [BIP 321](https://github.com/bitcoin/bips/blob/master/bip-0321.mediawiki) (or BIP 21) carry or represent the payment instructions.
2. **Payment Instruction Selection**: This research focuses on the logic—how a wallet decides *which* compatible instruction to execute from a multi-instruction payload.

## Example Problem

A merchant presents a payment request containing three payment instructions:
1. Standard On-chain Address
2. Payjoin Endpoint
3. BOLT 12 Offer

The wallet must choose how to proceed, but whether that choice benefits from shared guidance or should remain entirely local is an open research question. If it automatically defaults to standard on-chain, it misses out on Lightning's speed or Payjoin's privacy. If it automatically queries the Payjoin endpoint, it might leak IP/UTXO data to a receiver before deciding to pay via Lightning. If it attempts to fetch a BOLT 12 invoice, it initiates an interactive onion-message flow whose timing and transport metadata may require analysis, while BOLT 12 is designed not to require disclosure of the payer's ordinary node identity.

The wallet requires a clear, safe, and private policy to resolve this selection without exposing its capabilities or compromising user privacy.

## Research Investigations

Our current work focuses on five core investigations:
1. **Measuring wallet implementation differences**: Comparing how various wallet clients handle combined payment instructions.
2. **Evaluating wallet-fingerprinting hypotheses**: Analyzing the risk of unique user tracking based on optional parameter scanning and responses.
3. **Separating request-integrity risks from selection risks**: Disentangling channel modification vectors (like parameter stripping) from local client policy decisions.
4. **Studying fees, availability, liquidity, and interaction trade-offs**: Quantifying the economic and technical variables across payment methods.
5. **Determining whether shared test vectors or guidance are useful**: Exploring if collaborative tooling can improve interoperability without standardizing wallet policy.

## Repository Structure

The project is structured as follows:

```
bitcoin-payment-negotiation/
├── README.md                           # Project overview and research scope
├── CONTRIBUTING.md                     # Contribution guidelines and criteria
├── LICENSE                             # MIT License
├── SECURITY.md                         # Theoretical security vulnerability policy
├── .gitignore                          # Ignored file patterns
├── research/
│   ├── problem-statement.md           # Structured statement of the core problems
│   ├── prior-art.md                   # Review of existing standards and systems
│   ├── terminology.md                 # Precise definition of core concepts
│   ├── threat-model.md                # Security and privacy threat analysis
│   ├── open-questions.md              # Categorized outstanding research queries
│   ├── research-methodology.md        # Disciplined approach and validation rules
│   ├── initial-findings.md            # Summary of findings and hypotheses validation
│   └── wallet-implementation-matrix.md # Verifiable wallet support matrix
├── specification/
│   └── README.md                      # Reserved for future specifications (currently empty)
├── examples/
│   └── payment-selection-scenarios.md # Concrete payment selection scenarios
└── .github/
    ├── ISSUE_TEMPLATE/                # GitHub issue templates for structured feedback
    ├── pull_request_template.md       # Pull request submission template
    └── DISCUSSION_TEMPLATE/           # Discussion templates for research ideas
```

## How to Contribute

We welcome technical reviews, prior-art references, threat model critiques, and implementation feedback. Please read [CONTRIBUTING.md](CONTRIBUTING.md) to understand how to format and submit your contributions.

## Prior-Art Notice

We have not yet identified a common cross-protocol specification addressing this exact combination of payment-method selection and negotiation concerns. Please refer to [prior-art.md](research/prior-art.md) for details on existing standards and how they relate to this project. Refer to [initial-findings.md](research/initial-findings.md) and the [wallet-implementation-matrix.md](research/wallet-implementation-matrix.md) for our active research findings.

## Project Status

This project is currently in the **exploratory research phase**. No protocol schema, code implementation, or draft standard has been proposed.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
