# Contributing to Bitcoin Payment Negotiation

We welcome contributions from researchers, wallet developers, protocol designers, and the broader Bitcoin community. As an exploratory research project, our primary goal is to gather high-quality documentation, analyze prior work, and identify concrete interoperability and privacy issues.

## Types of Contributions We Encourage

1. **Prior-Art References**:
   - Highlighting existing standards, specifications, draft proposals, or academic papers that address multi-payment selection or negotiation.
   - Identifying how these works overlap with or diverge from our research questions.
2. **Wallet Implementation Examples**:
   - Sharing details on how existing production or development wallets handle payment requests containing multiple instructions.
   - Documenting current user experiences, default selection policies, and fallback mechanics.
3. **Privacy & Security Analysis**:
   - Submitting potential attack vectors, information leaks, or fingerprinting techniques related to capability negotiation or selection.
4. **Threat-Model Corrections**:
   - Critiquing, expanding, or refining the entries in our [threat-model.md](research/threat-model.md).
5. **Arguments Against Standardization**:
   - Providing reasoned technical arguments explaining why coordination or new standards might be unnecessary, harmful, or better handled purely at the local wallet UX level.
6. **Reproducible Interoperability Failures**:
   - Documenting cases where two wallets failed to negotiate or settle a payment despite having overlapping capabilities due to conflicting selection logic.
7. **Small, Focused Pull Requests**:
   - Making incremental improvements, adding missing definitions, or fixing typographical errors.

## Content and Rigor Standards

To maintain scientific and technical integrity, all contributors must adhere to the following rules:

### 1. Back Claims with Primary Sources
Any factual claim regarding a protocol, BIP, wallet behavior, or security property must include a link or citation to a primary source (e.g., official BIP document, source code repository, specification markdown file, or mailing list post).

### 2. Explicitly Categorize Claims
When contributing text to the research documents, you must explicitly distinguish between different types of statements. Avoid presenting opinions as facts. Use clear framing or labels to separate:
- **Existing Behaviour**: Documented, verifiable behaviors of current protocols or software.
- **Observation**: Empirical data gathered from tests or wallet interactions.
- **Assumption**: Hypotheses or working premises that form the basis of an argument but require verification.
- **Proposal**: Suggested changes, specifications, or algorithms.
- **Open Question**: Unresolved areas needing further inquiry or experimentation.

### 3. Maintain Cautious and Precise Jargon
- Avoid marketing language (e.g., "revolutionary", "game-changing", "universal").
- Apply terminology strictly as defined in [terminology.md](research/terminology.md). Specifically, do not use "selection" and "negotiation" interchangeably.
- Do not imply endorsement from Bitcoin developers or organizations.

## Pull Request Guidelines

- **Branch Naming**: Use descriptive prefixes for branches (e.g., `research/add-prior-art-xyz`, `docs/fix-terminology-typo`).
- **Focus**: Keep PRs focused on a single topic, document, or issue. Large, sprawling PRs are difficult to review and will be deferred.
- **Commit Messages**: Write clear, descriptive commit messages starting with a standard prefix (e.g., `docs:`, `chore:`).
