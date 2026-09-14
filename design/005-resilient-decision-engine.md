# Resilient Decision Engine

## Status and Intent

This note explores resilience requirements for Axion's decision engine. It is not a finalized runtime topology, consensus protocol, availability design, or service-level specification.

## Core Requirement

AI reasoning must never become a single point of failure in the enforcement path.

Axion may use models to interpret intent, correlate evidence, assess uncertainty, and propose responses. The ability to enforce established security policy must not depend on one model, one model family, one reasoning service, or the availability of probabilistic inference at all.

This requirement has two implications:

1. probabilistic reasoning and deterministic enforcement remain separate; and
2. loss, delay, compromise, or disagreement in reasoning reduces the available decision envelope rather than disabling enforcement or expanding authority.

Resilience does not require every action to continue during every failure. It requires predictable, policy-governed behavior that preserves security properties and bounds operational impact.

## Architectural Separation

### Probabilistic Reasoning

The reasoning layer may:

- interpret unstructured intent;
- correlate Enterprise Capability Graph evidence;
- compare activity with independent baselines;
- identify possible threats, dependencies, or business outcomes;
- estimate uncertainty;
- propose classifications, constraints, or response options; and
- produce evidence-linked explanations for review.

Reasoning outputs are untrusted, advisory claims. A model response is not an authorization token, policy decision, or enforcement command.

### Deterministic Decision and Enforcement

The deterministic layer:

- validates identity, delegation, capability, request schema, and trusted context;
- evaluates versioned policy;
- applies arbitration rules to reasoning outputs;
- selects the permitted decision class and obligations;
- enforces time, rate, scope, resource, and approval constraints;
- validates cached knowledge and its freshness;
- issues, restricts, or revokes capability grants; and
- records an attributable decision and enforcement result.

If reasoning is unavailable, deterministic enforcement remains active. The set of actions it may safely authorize can become narrower.

## Parallel Specialized Reasoning Sub-Engines

Axion may distribute reasoning across specialized sub-engines rather than depend on one general model. Possible responsibilities include:

- intent and capability interpretation;
- identity and delegation anomaly analysis;
- behavioral and volumetric analysis;
- tool, MCP server, and API risk assessment;
- data-movement and disclosure analysis;
- workflow and business-outcome consistency;
- threat correlation across regions or control domains; and
- explanation or evidence summarization.

A sub-engine may be a model, a statistical detector, a graph algorithm, a rules-based analyzer, or a composition of these. Specialization is a logical separation and does not require one deployed model per responsibility.

Parallel execution may reduce latency and common dependence on a single reasoning result. It can also introduce correlated failure, cost, operational complexity, contradictory outputs, and false confidence when several sub-engines share the same model family, data source, prompt pattern, or provider.

Independence must therefore be evaluated through failure domains and evidence diversity, not inferred from the number of calls.

## Deterministic Decision Arbitration

Reasoning outputs should enter a deterministic arbitration process. Arbitration evaluates structured claims, confidence metadata, evidence references, source health, timeliness, applicable policy, and disagreement.

Arbitration may produce outcomes such as:

- allow within existing deterministic policy;
- allow with narrower scope or additional obligations;
- require further evidence or another reasoning path;
- require step-up authentication or human approval;
- defer or queue the request within a bounded interval;
- quarantine or suspend a capability, session, tool, or workload; or
- deny.

Arbitration must not ask a final model to choose among model outputs and then treat that choice as authoritative. A model may summarize the disagreement, but policy determines how the system responds.

The arbitration record should identify which reasoning outputs were available, which were unavailable or late, where they disagreed, which evidence was considered, which policy rule applied, and why the enforced result followed.

## Disagreement as a Risk Signal

Disagreement between reasoning engines is evidence of uncertainty. It may indicate:

- ambiguous intent or incomplete context;
- different but plausible interpretations;
- stale or conflicting observation sources;
- a model or detector outside its reliable operating domain;
- manipulation of prompts, context, telemetry, or tools;
- a recent behavioral or system change; or
- a defect or compromise in a reasoning component.

Disagreement does not prove that an action is malicious, and agreement does not prove that it is safe. Correlated engines can agree on the same error.

Policy should define how the type and materiality of disagreement affect an action. A low-impact read may proceed with narrower results and additional audit, while an irreversible high-impact action may require resolution or human approval. Repeated disagreement can also reduce the trust assigned to a sub-engine or trigger its isolation.

## Bounded Decision Latency

Every governed capability should define a maximum decision interval appropriate to its impact and operational path. The interval includes the time allowed for required context retrieval, reasoning, arbitration, policy evaluation, approval where applicable, and delivery of an enforceable decision.

When the interval expires, Axion follows the policy-specific failure behavior for that action. It must not wait indefinitely, silently skip a required control, or reinterpret a timeout as approval.

### Fast-Path SLO Is Not an AI Deadline

A hard 150 ms end-to-end budget is not a credible general requirement for useful reasoning from a general-purpose model. A model may occasionally begin returning tokens within that interval, but first-token latency is not a complete, structured, parsed, and arbitrated risk contribution. Network transit, queueing, prompt and context assembly, inference, parsing, policy evaluation, and enforcement all consume the same end-to-end budget.

Axion may nevertheless use 150 ms—or another measured value—as a fast-path decision SLO for selected capabilities. AI can participate inside that interval when an appropriate specialized component completes in time. The enforcement architecture must not depend on heavyweight reasoning doing so.

An illustrative decomposition is:

| Illustrative elapsed time | Candidate decision contribution |
|---|---|
| 0–10 ms | Local cache, signature checks, hard policy, and locally available revocation state |
| 10–50 ms | Graph lookup, behavioral features, lightweight scoring, and bounded anomaly checks |
| 50–250 ms | Compact or specialized local or regional model contributions |
| 250 ms–2 s | Richer regional reasoning, broader context assembly, and multi-engine analysis |
| Seconds or longer | Deep correlation, global reasoning, investigation, simulation, and human approval |

The ranges describe architectural placement hypotheses, not service guarantees. Exact targets require measurement under the intended model, context, network, workload, and failure conditions.

### Latency Budget by Capability

Latency is a property of the governed capability and its risk policy. It should not be one platform-wide timeout.

For example:

- `sharepoint.read.document` might permit a 30 ms fast-path decision based on identity, hard policy, fresh revocation state, and a bounded cached verdict. The decision could include `monitor=true` and a 30-second decision validity, subject to the document's classification and release controls.
- `entra.assign.global-admin` may allocate up to two seconds for required regional reasoning and still require step-up authentication, separation of duties, and human approval. Because the effect is high impact, it must not proceed merely because the reasoning budget expired.

Latency budgets may differ for:

- synchronous API and tool invocations;
- interactive agent actions;
- CI/CD controls;
- incident containment;
- long-running workflow transitions; and
- out-of-path investigations or recommendations.

Exact latency targets remain an open design question. They should be established from representative use cases, target-system behavior, safety requirements, and measured infrastructure rather than chosen as one platform-wide number.

## Deterministic Fallback and Cached Knowledge

When required reasoning is unavailable or does not complete within its budget, Axion may fall back to deterministic policy and trusted cached knowledge. Candidate cached material includes:

- signed policy and capability definitions;
- identity and delegation state within a defined freshness window;
- hashes and signed reputation data;
- recent deterministic verdicts;
- bounded allow or deny decisions; and
- revocation and containment state.

Cached state is not automatically safe because it was previously valid. Each entry must be bound to relevant identities, tenant, capability, resource, policy version, constraints, provenance, and expiry. Revocation and invalidation requirements remain applicable.

Fallback may permit a narrow class of well-understood, low-impact actions. It must not use broader credentials, omit mandatory approval, accept stale high-risk allows, or route around the enforcement point. If deterministic evidence is insufficient, the decision envelope contracts.

## Progressive Reasoning and Enforcement

The AI contribution does not necessarily need to finish before every permitted request completes. For appropriate low-risk or reversible capabilities, Axion may make an initial constrained deterministic decision while regional or global reasoning continues asynchronously. This is **progressive reasoning and enforcement**.

For example, a gateway might return an initial decision in approximately 20 ms:

```text
ALLOW
monitor=true
decision_ttl=30s
```

Two hundred milliseconds later, a regional reasoner may report increased risk. Deterministic arbitration can then produce:

```text
future_invocations=RESTRICT
revoke_unused_grant=true
```

The values and syntax are illustrative, not a proposed wire format or latency commitment.

An initial decision may limit:

- accessible resources and returned fields;
- capability arguments;
- rate, concurrency, duration, or cumulative volume;
- network destinations and downstream tools;
- credential lifetime and audience;
- whether changes are staged rather than committed; and
- whether results may leave a controlled environment.

Later evidence may preserve the initial decision, tighten its constraints, require approval, suspend further progress, or revoke remaining authority. Enforcement points must receive and apply these changes within a defined propagation interval.

Progressive enforcement is appropriate only when the initial action is safely bounded and later restriction remains meaningful. It cannot retroactively reverse data disclosure, message delivery, financial transfer, destructive modification, or another irreversible effect. Such actions must wait until their required evidence, reasoning, policy decision, and approval are complete.

An already-issued capability grant may be revoked only while meaningful authority remains to be exercised and the enforcement path can observe the revocation in time. Axion must not imply that revoking a grant reverses an action that has already crossed an irreversible boundary.

Deleting data, changing privileged IAM, moving money, and deploying production code are examples of actions for which a larger reasoning budget or human approval may be operationally acceptable. Their business execution paths are not ordinarily justified by a sub-100 ms security decision if that deadline would omit required assurance.

The system should not present speculative execution as guaranteed rollback. Capability definitions must state which effects are reversible, compensable, stageable, or irreversible.

## Policy-Specific Failure Behavior

Axion should not expose one universal fail-open or fail-closed setting. Those labels are too coarse for systems that govern actions with different impact, reversibility, evidence needs, and availability requirements.

Failure behavior should be defined per policy and capability, considering:

- identity and authentication assurance;
- action impact and reversibility;
- affected tenant, resource, data classification, and region;
- availability and freshness of policy, revocation, and cached state;
- which reasoning functions are required versus advisory;
- acceptable delay and queueing behavior;
- permitted degraded scope;
- approval and audit requirements; and
- recovery and compensation options.

Examples include:

- permit a cached, field-limited read for a known workload while regional behavioral reasoning is unavailable;
- allow an already-started reversible workflow to reach a safe checkpoint but prevent new downstream effects;
- deny a privileged identity change without fresh policy and revocation state;
- queue a non-urgent high-impact action for later reasoning and approval; or
- permit an emergency containment action only through a pre-authorized, narrow capability with enhanced audit.

An exception for availability remains a policy decision. It must be explicit, bounded, attributable, and incapable of creating authority that did not exist before the failure.

## Circuit-Breaker Modes

The decision engine should expose explicit operating modes so that degraded behavior is observable and testable. Mode transitions are deterministic control-plane events, not model judgments.

### Normal

Required reasoning, policy, identity, revocation, audit, and enforcement services operate within their defined health and latency bounds. Actions follow their normal policy paths, including escalation and approval obligations.

### Degraded Reasoning

One or more reasoning sub-engines are unavailable, late, unhealthy, or producing material disagreement. Deterministic enforcement remains available. Policies may use remaining independent reasoning, narrower capability limits, increased audit, human review, or deterministic fallback.

### Deterministic Safe Mode

Probabilistic reasoning is unavailable or outside its acceptable assurance envelope. Only actions fully supported by deterministic policy, sufficiently fresh trusted state, and explicitly approved safe-mode rules may proceed. Dynamic interpretation and actions requiring model-derived context pause or deny.

### Emergency Restricted Mode

A severe control-plane, integrity, regional, or enterprise incident restricts operation to a minimal pre-authorized set of capabilities. Candidate actions may include containment, revocation, health verification, evidence preservation, and carefully defined recovery operations.

Emergency restricted mode is not administrative bypass. It should require protected activation, separation of duties where feasible, short validity, explicit scope, immutable audit, and tested exit criteria.

### Mode Transition Requirements

Transitions should consider component health, telemetry gaps, policy and revocation freshness, reasoning disagreement, integrity verification, regional isolation, and operator declaration. The design must prevent rapid oscillation, unauthorized downgrades, and a compromised component falsely declaring itself healthy.

Returning to a less restrictive mode requires evidence that relevant dependencies and state have recovered, not only the passage of time.

## Control and Management Plane Protection

The resilience of data-plane enforcement depends on protecting the systems that define and distribute authority.

### Signed Policy and Configuration

Policy, capability definitions, trust configuration, cache rules, model routing, and circuit-breaker configuration should be versioned and integrity-protected. Enforcement points must reject invalid or unauthorized state and know which versions remain acceptable during disconnection.

### Workload Identity

Reasoning engines, policy services, gateways, brokers, configuration distributors, and administrative tools require distinct workload identities. Network location or possession of a shared secret is insufficient for control-plane trust.

### Immutable Audit

Policy changes, deployments, mode transitions, overrides, decisions, grant issuance, revocations, and administrative access require tamper-resistant audit. “Immutable” refers to protection against unauthorized alteration and deletion under the defined retention model; it does not remove privacy, legal deletion, or governed correction requirements.

### Staged Rollout

Changes to policy, arbitration, reasoning components, models, prompts, capability definitions, and cache behavior should support evaluation, shadow operation, limited exposure, regional canaries, and progressive rollout. A reasoning change must not silently alter authorization semantics.

### Rollback

Known-good policy, configuration, and component versions should be recoverable through authenticated, tested rollback procedures. Rollback must account for schema compatibility, already-issued grants, cached state, revocations, and actions already executed.

### Separation of Duties

No single routine operator or reasoning component should be able to author policy, approve its release, deploy it globally, suppress its audit, and override its enforcement. The degree of separation should match the potential impact and operational constraints.

### Regional Survivability

Regions should retain enough authenticated policy, revocation, identity, capability, and circuit-breaker state to enforce their defined local decision envelope during loss of global or cross-region services. Regional autonomy must be bounded so that isolation does not preserve stale high-risk authority indefinitely.

Recovery requires reconciliation of policy versions, decisions, grants, revocations, audit, queued actions, and partial workflows. Cross-region failover must not violate tenant isolation, data residency, or the authority assigned to a region.

## Example Failure Scenarios

### One reasoning engine times out

An identity-risk sub-engine completes, while the behavioral sub-engine exceeds the capability's latency budget. Arbitration records the missing result. A low-impact read proceeds with reduced fields and enhanced audit because policy marks behavioral reasoning as advisory for that action. A privileged write under the same condition waits or denies because the reasoning input is mandatory at that risk tier.

### Reasoning engines disagree

One engine interprets an action as routine remediation while another identifies unusual target scope and tool fan-out. Deterministic arbitration treats the disagreement as material uncertainty. The request is narrowed to a reversible staging action and escalated for human review; no engine receives authority to resolve the conflict by itself.

### Regional reasoning becomes unavailable

Edge gateways enter degraded reasoning mode. They continue a bounded set of deterministic actions using signed policy and sufficiently fresh cached state. Actions requiring regional correlation pause or deny. If policy or revocation freshness exceeds its limit, the affected gateways enter deterministic safe mode.

### Global control services become unreachable

A region continues within its pre-established authority envelope. It cannot publish new global policy, expand capabilities, or accept stale high-risk grants. Local decisions and mode transitions are recorded for later reconciliation. Emergency restricted mode remains available through protected regional controls if the incident affects the integrity of local state.

## Security and Operational Considerations

### Common-mode failure

Multiple reasoning calls do not provide resilience when they share the same provider, model lineage, prompt, retrieval source, network, identity, or orchestration defect. Failure-domain analysis should accompany any claim of independence.

### Adversarial disagreement

An attacker may manipulate one source or sub-engine to create disagreement and force costly escalation or denial. Arbitration needs source weighting, rate controls, corroboration, and policy limits without suppressing legitimate uncertainty.

### Stale deterministic state

Deterministic fallback can still be unsafe when policy, identity, delegation, or revocation state is stale. Freshness is part of correctness, and each policy must define its maximum acceptable age.

### Recovery storms

When reasoning or control services recover, queued requests, cache refreshes, graph updates, and policy reconciliation can overload the system. Recovery needs prioritization, backpressure, idempotency, and preservation of original decision context.

### Human overload

Routing every uncertain request to a person creates approval fatigue and an operational bottleneck. Policies should first narrow, defer, or deny activity and reserve human judgment for meaningful, sufficiently contextualized decisions.

### Resilience versus consistency

Regional survival and local caching trade immediate consistency for availability. Axion must make this trade explicit by capability and risk tier rather than hiding it behind infrastructure behavior.

## Current Design Commitments

- Probabilistic reasoning does not authorize or directly enforce actions.
- Deterministic enforcement remains operational when reasoning is degraded or unavailable.
- Reasoning outputs are structured, attributable inputs to deterministic arbitration.
- Disagreement is preserved as uncertainty rather than averaged away.
- Decision latency is bounded by use-case policy.
- Fallback cannot broaden authority.
- Progressive enforcement is limited to actions whose initial effects are safely constrained.
- Failure behavior is policy-specific.
- Circuit-breaker state is explicit, auditable, and deterministically controlled.
- Regional survivability must preserve policy, identity, revocation, audit, and tenant boundaries.

## Open Design Questions

- How many reasoning sub-engines are required for each decision class?
- Which model families, statistical methods, graph algorithms, or rules should be used for each reasoning responsibility?
- What degree of model, provider, data, prompt, infrastructure, and regional diversity provides meaningful independence?
- Should any decision use quorum rules, and if so, how should quorum interact with engine specialization, confidence, absence, and disagreement?
- What exact latency targets apply to each capability and operating mode?
- Which capabilities require a fast-path SLO, and how should that SLO be distinguished operationally from individual reasoning-engine deadlines?
- Which decision contributions can reliably complete in the 0–10 ms, 10–50 ms, 50–250 ms, 250 ms–2 s, and longer-duration tiers?
- Which reasoning inputs are mandatory, advisory, or prohibited for each risk tier?
- How should arbitration schemas represent uncertainty, contradictory evidence, and unavailable engines without collapsing them into one opaque score?
- Which deterministic decisions may be cached, for how long, and with what revocation guarantees?
- What initial effects are sufficiently reversible or constrained for progressive enforcement?
- How quickly must tightened decisions and revocations propagate to gateways, brokers, workflows, and targets?
- What objective health criteria trigger and clear each circuit-breaker mode?
- How much authority may a region retain during global isolation, and for how long?
- How should staged reasoning changes be evaluated without allowing advisory outputs to alter production authorization unexpectedly?
- What evidence demonstrates that the resilient design reduces material risk rather than adding correlated complexity?
