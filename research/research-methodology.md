# Research Methodology

This document outlines the disciplined research methodology governing the `bitcoin-payment-negotiation` project. Because payment protocols are highly sensitive to security, privacy, and user experience, we adhere to a structured and cautious evaluation process.

---

## The Ten-Step Research Process

We follow these ten sequential steps to evaluate our hypotheses:

1. **Review Existing Standards**:
   Thoroughly examine all established and draft Bitcoin payment standards (e.g., BIP 21, BIP 321, BIP 352, BIP 353, BOLTs, LNURLs). We must map their capabilities, parameters, and design limitations before proposing any additions.
   
2. **Review Wallet Behaviour**:
   Audit how modern Bitcoin wallets currently handle incoming multi-instruction payment requests. This includes tracing their default selection logic, UI prompts, and fallback behaviors.
   
3. **Collect Real Implementation Examples**:
   Document specific instances, configurations, and logs of wallets interacting with multi-method payment strings.

4. **Identify Concrete Interoperability Failures**:
   Focus on real-world edge cases where a payment fails or degrades to a suboptimal method due to conflicting local policies or parsing inconsistencies.

5. **Construct Threat Models**:
   Continuously update and maintain a threat model mapping capabilities, attacker powers, and privacy leakage vectors.
   
6. **Compare Local Selection with Interactive Negotiation**:
   Analyze the trade-offs of local selection (sender decides autonomously from a static payload) against interactive negotiation (sender and receiver exchange messages). We evaluate these options specifically on privacy preservation, network latency, and complexity.

7. **Design Only Minimal Experiments**:
   If research suggests a gap, we construct the smallest possible experiment or proof-of-concept to validate the claim. We avoid writing complex protocol engines or production libraries prematurely.

8. **Publish Assumptions**:
   All working assumptions, security models, and design trade-offs must be documented openly in our research folders.

9. **Invite Criticism**:
   Actively seek out peer review from protocol designers, wallet developers, and the broader open-source community to find flaws in our reasoning or assumptions.

10. **Revise or Abandon the Hypothesis**:
    If empirical evidence, security audits, or peer feedback shows that wallet-local policy is sufficient, or that negotiation introduces unacceptable privacy leaks, we will revise or fully abandon the hypothesis.

---

## Core Rules

### Rule of Prior-Art Search
> [!IMPORTANT]
> **No novelty claim shall be made in this project without a documented prior-art search.**
> 
> Before claiming that a problem is "unsolved," a concept is "new," or a design is "original," contributors must search existing standards, mailing lists (e.g., Bitcoin-dev, Lightning-dev), and open-source project issue trackers. The findings of this search must be added to [prior-art.md](file:///Users/jaykon/Developer/open-source/bitcoin-payment-negotiation/research/prior-art.md) with direct source links.

### Falsifiability
All research papers and problem statements must conclude with falsifiable research questions. We do not design solutions for speculative problems; we must first demonstrate that the problem exists and causes tangible issues for users or developers.
