# ADR-003: Authorization Policy Must Produce Explainable Decisions

## Status

Accepted

## Context

Agentic systems can generate high volumes of actions through complex identity and delegation paths. Operators, reviewers, and affected users need to understand why access was allowed or denied. A bare decision is insufficient for incident response, policy validation, approval, and governance.

## Decision

Every authorization decision must record a stable policy version, the trusted inputs material to the result, the applicable rule or reason, the outcome, obligations, and validity window. Explanations must be derived from deterministic policy evaluation rather than generated solely by a model.

Explanations may be redacted or audience-specific when policy details or attributes are sensitive, but a protected full-fidelity record must remain available to authorized reviewers.

## Rationale

Explainable decisions make policy behavior testable and auditable. They reduce the risk that an organization operates controls it cannot diagnose and distinguish policy defects from model, identity, and enforcement failures.

## Consequences

### Positive

- Reviewers can reconstruct and challenge decisions.
- Policy changes can be compared and regression-tested.
- Denials can be resolved without relying on model speculation.
- Approval interfaces can show which conditions require human judgment.

### Negative / Trade-offs

- Decision records may reveal sensitive attributes or policy structure.
- Explanation schemas and reason codes require governance and versioning.
- Distributed PDPs must correlate their decisions consistently.
- Complete records increase storage, privacy, and retention concerns.

## Alternatives Considered

- **Record only allow or deny.** Rejected because it is inadequate for diagnosis and accountability.
- **Ask an LLM to explain the decision afterward.** Rejected as the authoritative explanation because it may invent or omit causal factors; model-generated summaries may supplement the policy record.
- **Expose full policy traces to every caller.** Rejected because they may leak sensitive logic or attributes.

## Open Questions

- What common decision-record schema is sufficiently portable?
- Which facts should be retained versus referenced to minimize sensitive duplication?
- How should explanations work across multiple PDPs and target-system decisions?
- What information should requesters, approvers, operators, auditors, and developers each see?
