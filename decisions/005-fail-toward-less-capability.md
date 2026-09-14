# ADR-005: Fail Toward Less Capability

## Status

Accepted

## Context

Agentic action paths depend on identity, policy, delegation, approval, credentials, brokers, targets, and telemetry. Failures and stale state are inevitable. A fallback that skips controls, broadens credentials, or defaults to allow can turn an availability event into a security incident.

## Decision

When a security-relevant dependency is unavailable, invalid, stale beyond its defined limit, or produces an ambiguous result, the system reduces capability or denies the action. It must not fall back to broader credentials, unmediated tool access, default allow, or unaudited execution.

Any degraded mode must be predefined, narrowly scoped, time-bound, observable, and incapable of granting authority not already explicitly approved for that mode.

## Rationale

Security failures should have bounded consequences. A predictable reduction in capability is easier to test, monitor, and recover from than an implicit privilege expansion. Explicit degraded modes allow availability requirements to be considered without disguising bypasses as resilience.

## Consequences

### Positive

- Dependency failure cannot silently increase privilege.
- Failure semantics can be tested and included in risk analysis.
- Operators can distinguish intentional degraded operation from control bypass.
- Credential and policy fallbacks remain explicit and auditable.

### Negative / Trade-offs

- Policy, identity, approval, revocation, or audit outages may block business operations.
- High availability and propagation engineering become important for control services.
- Emergency procedures require careful design and exercises.
- Partial multi-step actions may still require compensation or manual recovery.

## Alternatives Considered

- **Fail open for availability.** Rejected for security-relevant decisions because it converts dependency failure into unauthorized authority.
- **Reuse cached allow decisions indefinitely.** Rejected because revocation and policy changes would not take effect; bounded caches may be used where risk permits.
- **Fall back to a shared administrator credential.** Rejected because it expands privilege and destroys attribution.
- **Block every action whenever any telemetry is impaired.** Not adopted universally; evidence requirements depend on action risk, but exceptions must be explicit policy rather than ad hoc bypass.

## Open Questions

- Which low-impact operations may continue with cached decisions, and for how long?
- What telemetry loss should block each action class?
- How should emergency access preserve separation of duties and later review?
- What compensation guarantees are realistic for partially completed workflows?
