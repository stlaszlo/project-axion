# ADR-001: LLMs Must Not Make Final Authorization Decisions

## Status

Accepted

## Context

Models interpret ambiguous intent and can propose useful actions, but their outputs are probabilistic and sensitive to prompts, context, model versions, and adversarial input. A final authorization decision must be consistently enforceable, reviewable, testable, and attributable. Placing that decision inside model reasoning would make prompt injection and reasoning error direct paths to authority.

## Decision

LLMs may classify intent, extract attributes, recommend policy, and propose actions. A deterministic policy mechanism outside the model reasoning path must make the final authorization decision, and an enforcement point must prevent execution without that decision.

Model-derived attributes cannot independently increase authority. When uncertain or untrusted inputs are relevant, policy may use them to reduce capability, require approval, or deny the request.

## Rationale

Separating reasoning from authorization limits the consequences of model failure. It allows policies to be versioned, tested, explained, audited, and applied consistently across models and tools. It also makes authority independent of prompt wording and supports defense in depth with downstream authorization.

## Consequences

### Positive

- Model compromise does not by itself grant additional authority.
- Decisions can be reproduced from trusted inputs and a policy version.
- Enforcement remains stable across model and prompt changes.
- Organizations can apply existing policy-as-code and identity controls.

### Negative / Trade-offs

- Natural-language intent must be converted into structured, policy-relevant actions.
- Policy design and attribute quality become critical dependencies.
- Complex contextual judgments may require conservative rules or human review.
- Additional decision and enforcement steps add latency and operational complexity.

## Alternatives Considered

- **Allow the model to decide using policy text in its prompt.** Rejected because instructions are not an enforceable boundary and decisions are not reliably reproducible.
- **Use a separate authorization model.** Rejected as the final authority because a second probabilistic system retains the same category of risk; it may advise a deterministic PDP.
- **Rely only on target-system permissions.** Rejected because target permissions may be too coarse and may not understand the initiating actor, agent, delegation, or approval context. Target authorization still remains required.
- **Require human approval for every action.** Rejected as a universal mechanism because it does not scale and encourages approval fatigue; it remains appropriate for selected high-impact actions.

## Open Questions

- Which policy language and evaluation architecture best support portable implementations?
- Which contextual inputs are sufficiently trustworthy to grant or constrain access?
- How should policy handle actions whose risk depends on a sequence rather than one request?
- What decision latency and availability targets are acceptable for each risk tier?
