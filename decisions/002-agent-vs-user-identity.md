# ADR-002: Agent Identity Must Remain Distinct from User Identity

## Status

Accepted

## Context

An agent may act for a human or service while running as a separate workload, using its own configuration and capabilities. If downstream systems see only the user's identity, agent-specific restrictions and attribution are lost. If they see only a shared agent identity, the initiating actor and delegated purpose are lost.

## Decision

Represent the initiating actor, logical agent, and executing workload as distinct identities linked by an explicit, bounded delegation chain. Authorization considers all relevant identities and grants no more than their intersecting authority and delegation constraints.

## Rationale

Distinct identities support least privilege, attribution, revocation, separation of duties, and containment. They allow an agent to be disabled without disabling the user, and prevent possession of a user's token from becoming an unrestricted substitute for delegated authority.

## Consequences

### Positive

- Audit can identify who initiated, which agent interpreted, and which workload executed an action.
- Agent and workload permissions can be independently limited and revoked.
- Delegation cannot silently inherit all user authority.
- Compromise can be contained at a narrower principal.

### Negative / Trade-offs

- Identity issuance, token exchange, claims design, and lifecycle management become more complex.
- Downstream systems may not understand multi-actor identity.
- Legacy integrations may require a broker to preserve attribution indirectly.
- Policy must define how conflicting constraints across the chain are resolved.

## Alternatives Considered

- **Impersonate the user end to end.** Rejected because it obscures agent involvement and often over-delegates user authority.
- **Use one shared service identity for all agents.** Rejected because it weakens attribution, isolation, revocation, and least privilege.
- **Represent the agent only as an audit attribute.** Rejected where enforcement needs agent-specific restrictions; it may be a compatibility fallback for legacy targets if the broker remains authoritative.

## Open Questions

- Which interoperable token or credential formats can preserve actor chains?
- How should non-human initiating services be represented alongside agents?
- What fallback is acceptable when a target accepts only one principal?
- How are agent versions and configurations bound to a stable logical identity?
