# Architecture

## Purpose and Scope

This document describes a vendor-neutral control architecture for agent-initiated actions. It focuses on the point at which probabilistic reasoning becomes a request to exercise real-world authority.

Axion does not prescribe a particular model, identity provider, policy language, broker, or telemetry system. Product mappings and deployment topology remain implementation concerns.

## Assumptions

- Models, prompts, retrieved content, memory, tools, and surrounding services may fail or be compromised.
- Enterprise systems remain authoritative for their own data and domain invariants.
- A deterministic policy system can evaluate structured requests without depending on free-form model output.
- Strong workload identity, bounded delegation, and short-lived credentials are available or can be introduced. Where they are unavailable, residual risk increases and compensating controls are required.
- Some actions cannot be made safe through authorization alone and require validation, approval, workflow constraints, or prohibition.

## Actors and Components

| Actor or component | Responsibility | Trust consideration |
|---|---|---|
| Initiating human or service | Provides intent and originating authority | Its identity may be stolen; intent may be ambiguous or manipulated |
| Agent runtime | Coordinates reasoning, context, and proposed tool calls | Not an authorization authority; treat its output as untrusted input |
| Model | Interprets input and proposes plans or actions | Probabilistic and susceptible to instruction and context attacks |
| Identity and delegation service | Authenticates actors and represents bounded delegation | Must preserve the distinction between initiator, agent, and workload |
| Axion control boundary | Mediates proposed actions and coordinates enforcement | Security-critical; bypass resistance and fail-closed behavior are required |
| Policy decision point (PDP) | Evaluates structured authorization requests | Decisions must be deterministic, versioned, and explainable |
| Policy enforcement point (PEP) | Prevents execution without an allow decision and required obligations | Must be on every authoritative execution path |
| Tool/API broker | Resolves capabilities, constrains requests, and invokes targets | Tools and their descriptions are untrusted; broker must enforce contracts |
| Approval service | Obtains attributable approval for defined actions | Approval must bind to the exact action or bounded change set |
| Target API or SaaS | Performs the domain operation | Retains domain validation and should verify presented identity and scope |
| Audit and telemetry services | Preserve evidence and support detection and response | Logs may contain sensitive data and require integrity and access controls |

## Data Plane and Control Plane

The **data plane** carries action requests, validated parameters, delegated credentials, tool invocations, and results. It should expose only the capabilities and data required for the approved action.

The **control plane** defines identities, trust relationships, capability registrations, policies, approval rules, revocation state, and telemetry configuration. Changes to the control plane can alter future authority and therefore require stronger administrative controls, separation of duties, and audit than ordinary data-plane requests.

The separation is logical. An implementation may combine services, but it must preserve distinct authorization, administration, and audit responsibilities.

## Trust Boundaries

1. **Intent boundary:** unstructured human, service, or retrieved input enters the agent context.
2. **Reasoning-to-action boundary:** model output becomes a structured action proposal. This is Axion's central boundary; model output is data, not authority.
3. **Authorization boundary:** the proposal, identities, delegation, and context are evaluated by deterministic policy.
4. **Execution boundary:** an allowed request is converted into a target-specific invocation with bounded credentials.
5. **External-system boundary:** the broker interacts with an independently administered API, SaaS service, or local tool.
6. **Telemetry boundary:** action and result evidence moves into monitoring and audit systems, potentially crossing data-classification boundaries.

See [diagrams/trust-boundaries.mmd](diagrams/trust-boundaries.mmd).

## Request Flow

1. The entry point authenticates the initiating human or service and records the presented intent.
2. The agent runtime invokes a model using only the context permitted for the session.
3. The model proposes a named capability and structured arguments. This proposal carries no authority.
4. The control boundary validates the proposal against the capability's schema and rejects unknown, malformed, or disallowed fields.
5. The boundary constructs an authorization request containing the initiating identity, agent identity, workload identity, delegation chain, requested capability, resource, parameters relevant to policy, and current environmental context.
6. The PDP returns a deterministic decision: deny, allow, or allow with obligations. Obligations may include parameter limits, redaction, approval, step-up authentication, rate limits, or additional logging.
7. If approval is required, the approver sees the material action, target, scope, and consequences. Approval is bound to the request digest and expires.
8. The PEP verifies that the decision and obligations remain valid immediately before execution.
9. The broker obtains or presents a short-lived, audience-restricted credential and invokes the target through an allow-listed adapter.
10. The target applies its own authentication, authorization, validation, and business invariants.
11. The broker validates and classifies the result before returning only permitted data to the agent context.
12. Audit records link the initiator, delegation, proposal, policy version and decision, approval, credential reference, invocation, and outcome.

Authorization is re-evaluated when the action, target, material parameters, identity, delegation, risk context, or policy version changes. A plan-level approval must not silently authorize materially different execution-time actions.

## Policy Decision Points

Policy evaluation is expected at more than one stage:

- **Session admission:** may this actor use this agent and data context?
- **Context access:** may this workload retrieve this source or memory segment?
- **Capability discovery:** which tools may be exposed for this session?
- **Action authorization:** may this exact actor-agent-workload chain perform this action on this resource?
- **Credential issuance:** may a credential with the requested audience, scope, and lifetime be created?
- **Result release:** may the returned data enter agent context or be disclosed to the initiator?

Not every deployment needs a single centralized PDP. Distributed decisions are acceptable if semantics are consistent, policy versions are attributable, and no execution path bypasses enforcement.

## Credential Delegation

Delegation must preserve identity rather than collapsing the human, agent, and workload into one principal. The authorization request should distinguish:

- the initiating actor;
- the agent or logical automation acting on the request;
- the executing workload instance;
- the target audience and requested capability;
- the delegation constraints and expiry.

The effective authority should be no greater than the intersection of these constraints. Credential exchange or capability tokens are possible mechanisms, not architectural requirements. Long-lived user tokens stored in agent memory are inconsistent with this design.

Open implementation questions include how delegation is represented across vendors, whether downstream systems can consume actor-chain claims, and how legacy APIs without fine-grained scopes are contained.

## Revocation and Interruption

Revocation operates at several levels:

- disable an initiating, agent, workload, service, or tool identity;
- invalidate a delegation grant or session;
- withdraw a capability registration or policy permission;
- revoke or allow a short-lived credential to expire;
- block a target, tenant, resource, or action class;
- interrupt queued or in-progress work where the target operation permits cancellation.

Short credential lifetimes bound exposure but do not replace revocation. The design must define how quickly changes propagate to PDPs, brokers, caches, and active sessions. Irreversible external actions cannot always be stopped once accepted, so interruption guarantees must be described honestly for each capability.

## Failure Behavior

Failures should reduce available capability:

- unavailable policy, identity, approval, or revocation services deny new high-impact actions;
- stale policy or identity data is rejected beyond a defined maximum age;
- malformed model output is rejected rather than repaired into a more privileged request;
- credential issuance failure does not fall back to shared or broader credentials;
- unknown tools and unverified tool versions are unavailable;
- telemetry failure triggers a risk-based response defined by policy, with security-critical actions blocked when required evidence cannot be recorded;
- partial multi-step execution stops, records completed effects, and follows an explicit compensation or operator-recovery path.

Availability exceptions, if any, must be narrow, pre-authorized, observable, time-bound, and incapable of increasing privilege.

## Example Flows

### Read-only support lookup

A support analyst asks an agent to summarize a customer case. The agent proposes a case-read capability. Policy allows access only to cases assigned to the analyst's team, and the broker retrieves a scoped record. Sensitive fields are removed before the result enters model context. The residual concern is that permitted content may itself contain prompt injection; retrieved text therefore cannot authorize subsequent actions.

### High-impact account change

An operator asks an agent to change a production account setting. Policy requires step-up authentication and approval by a separate authorized person. The approval binds the account, setting, proposed value, initiator, and expiry. The broker rechecks policy and uses a single-purpose credential. If the value or target changes, approval is invalid and must be obtained again.

### Service-to-service remediation

A monitoring service asks an automation agent to isolate a suspicious workload. The service, agent, and executing workload retain separate identities. Policy permits isolation only for specified environments and confidence inputs, imposes a short expiry, and records the evidence reference. Whether automated isolation is appropriate for production remains a risk decision; Axion supplies control points but does not make that business decision.

### Multi-agent delegation

One agent delegates a research subtask to another. The second agent receives a narrower delegation that permits retrieval but not external mutation. Any proposed downstream action is evaluated against the full chain; delegation cannot add authority absent from the initiator or delegating agent. Protocol-level agent-to-agent trust and interoperable delegation formats remain open questions.

## Open Questions

- What common representation can carry actor chains and delegation constraints across heterogeneous systems?
- Which policy inputs are stable and trustworthy enough for deterministic evaluation?
- How should capability descriptions and tool versions be signed, distributed, and revoked?
- What assurance is required for local tools versus hosted tools and models?
- How can secure memory preserve provenance, retention, and deletion semantics without becoming an ambient authority source?
- Which actions require approval, dual control, simulation, or prohibition?
- What evidence is sufficient to reconstruct decisions without retaining excessive sensitive content?
- How should organizations measure policy correctness, enforcement coverage, and containment effectiveness?
