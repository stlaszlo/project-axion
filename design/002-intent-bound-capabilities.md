# Intent-Bound Capabilities

## Status and Intent

This note documents the evolution of Axion's original tokenized API-call concept into a possible model for intent-bound capabilities. It is a design exploration, not a finalized protocol or implementation specification.

The term **Axion Capability Grant** is used for discussion. Its representation, issuance protocol, validation mechanism, and interoperability requirements remain undecided.

## Historical Design: Tokenized API Calls

The original design placed an Axion gateway between clients and downstream SaaS APIs. A client invoked a stable opaque token at the gateway. The token mapped to a predefined business action or to a known sequence of downstream API calls.

For example, a token could represent an employee-onboarding action without exposing the client to the individual identity, HR, finance, email, and IT APIs needed to perform it. Axion would resolve the token, obtain protected downstream credentials, and execute the configured calls.

### Intended Benefits

- **Deterministic execution context:** the token selected a predefined action rather than accepting arbitrary downstream API instructions.
- **Credential isolation:** downstream credentials remained behind the gateway and were not disclosed to callers.
- **Repeatability:** equivalent invocations followed a known path with consistent controls.
- **Containment:** disabling a token or mapping could remove a caller's route to the associated action.
- **Interface stability:** callers could depend on a business-level identifier while downstream APIs changed.

### Problems Identified

- **Static-token leakage:** if possession of the token conveyed authority, disclosure could enable replay or unauthorized use.
- **Reverse engineering:** observed identifiers, request patterns, errors, or documentation could reveal valuable business capabilities even when tokens were opaque.
- **Gateway complexity:** mapping, validation, credentials, sequencing, retries, compensation, and downstream changes could accumulate in one security-critical component.
- **Tight workflow coupling:** business process changes could require gateway changes and create dependencies between policy enforcement and workflow implementation.
- **Unclear ownership:** executing multi-step business processes risked turning Axion into a workflow engine rather than a security and trust boundary.

The original concept therefore contained a useful constraint—the caller requested a predefined business intent rather than arbitrary API access—but coupled the identifier, authority, and execution model too closely.

## Current Direction

The current direction preserves the useful part of the concept as **Intent-Bound Capabilities**, represented during authorized use by **Axion Capability Grants**.

An intent-bound capability names a bounded business action and defines the downstream capabilities through which that action may be performed. The identifier makes the requested intent explicit. It does not, by itself, authorize execution.

The design separates four concepts:

1. **Identity — who is requesting an action.** The initiating human, service, agent, and executing workload are authenticated independently and remain attributable through the delegation chain.
2. **Capability — what business action is requested.** A stable capability identifier names a governed action and its contract, such as `hr.employee.onboard`.
3. **Policy/Decision — whether the action is allowed under current context.** Deterministic policy evaluates identity, delegation, tenant, capability, requested scope, material arguments, environment, risk, and other trusted facts.
4. **Execution — how approved downstream systems perform the action.** A broker, application, or external workflow engine invokes the permitted APIs and tools under the approved constraints.

These concepts may be implemented by cooperating services, but their security responsibilities should not be collapsed. In particular, knowledge or possession of a capability identifier must not be treated as identity or authorization.

## Capability Identifiers

Capability identifiers are stable, namespaced references to business actions. Examples might include:

- `hr.employee.onboard`
- `finance.invoice.approve`
- `security.workload.isolate`
- `customer.case.summarize`

Stable capability IDs are identifiers, not secrets. They may appear in documentation, schemas, policy, telemetry, or agent tool descriptions. Authorization must never depend solely on possession, obscurity, or unpredictability of the ID.

Callers authenticate independently using workload or service identity and, where applicable, a delegated initiating-user identity. Capability discovery may be filtered to reduce misuse and unnecessary exposure, but hiding an identifier is not a security boundary.

## Capability Definitions

A capability definition describes the bounded action that policy and mediation recognize. Depending on the use case, it may specify:

- a canonical capability ID and version;
- permitted caller, agent, workload, or tenant classes;
- required and optional structured arguments;
- policy-relevant argument fields;
- allowed resource scopes and argument ranges;
- downstream APIs and operations that may be reached;
- permitted MCP servers and tool operations;
- credential audience and scope requirements;
- approval, step-up authentication, logging, or rate-limit obligations;
- expected result classification and release constraints; and
- execution ownership, including any external workflow definition.

The definition is not a natural-language prompt and should not rely on model interpretation for enforcement. Human-readable descriptions may help agents select a capability, but the enforceable contract must be structured, versioned, validated, and governed.

Capability definitions can allow an agent to request a business intent without receiving arbitrary authority over the underlying APIs or MCP tools. An agent permitted to invoke `hr.employee.onboard` need not receive general access to create arbitrary identities, modify payroll records, send unrestricted email, or execute unrelated IT workflows.

## Axion Capability Grants

After deterministic policy permits an invocation, Axion may issue or establish a short-lived, narrowly scoped capability grant. The grant would bind the authorization decision to a particular execution context.

A grant may contain or securely reference:

- caller and initiating-actor identity;
- agent and executing-workload identity where relevant;
- tenant or organizational boundary;
- capability ID and version;
- approved resource scope;
- expiry and, where required, not-before time;
- nonce, transaction identifier, or replay-prevention state;
- policy version and decision reference;
- approved argument constraints;
- required obligations, such as approval or enhanced audit; and
- intended broker, workflow, or downstream audience.

The effective authority remains the intersection of identity, delegation, policy, capability definition, and grant constraints. A grant cannot add authority absent from the authorization decision.

### Representation Remains Undecided

This design does not select a grant representation. Candidate approaches include:

- JSON Web Tokens (JWTs);
- PASETO tokens;
- macaroons or another attenuable authorization format;
- opaque references resolved through an Axion decision service; and
- a hybrid approach using signed assertions and server-side state.

The choice depends on requirements for revocation, attenuation, privacy, delegation, offline validation, audience binding, token size, key management, interoperability, replay resistance, latency, and audit. No candidate should be treated as preferred until those requirements are defined and tested.

## Authorization and Invocation Flow

1. The caller authenticates using its workload or service identity and presents any permitted delegation context.
2. The caller or agent requests a named capability with structured arguments.
3. Axion resolves a versioned capability definition and validates the request schema.
4. Deterministic policy evaluates the identity chain, tenant, capability, scope, arguments, current context, and applicable risk conditions.
5. Required obligations—such as human approval or step-up authentication—are completed and bound to the material request.
6. Axion issues or records a short-lived capability grant with no more authority than the approved invocation requires.
7. The capability mediator validates the grant, checks replay and revocation state as required, and releases only the permitted downstream operations and credentials.
8. A target application or external workflow engine performs the action using its own domain controls.
9. Results, partial effects, errors, and compensation events are correlated with the original identity, decision, grant, and invocation.

A material change to the caller, tenant, capability, target resource, argument constraints, policy state, or execution plan requires re-evaluation. Execution components must not reinterpret a grant into broader authority.

## Boundary with Workflow Engines

Axion owns authorization and capability mediation. It determines whether a caller may request a bounded business action and constrains the authority made available to perform that action.

External workflow engines should own long-running workflow state, scheduling, retries, waiting, branching, compensation, and business-process progression. Axion may authorize individual transitions or issue grants for bounded workflow segments, but it should not become the general system of record for workflow execution.

This boundary is intentionally strict but not yet fully specified. Some short, atomic mediation sequences may reasonably execute within an Axion broker. The distinction should be based on security responsibility and lifecycle complexity rather than a fixed number of downstream calls.

## Example: `hr.employee.onboard`

The `hr.employee.onboard` capability represents the intent to establish approved enterprise access and services for a new employee. It does not provide the caller with general administrative access to the participating systems.

### Example Request Context

An authenticated HR service requests the capability for a named employee record, employing entity, manager, role, location, start date, and approved access profile. Policy evaluates the service and workload identities, tenant, source HR record, initiating human if applicable, requested access profile, regional rules, start date, and required approvals.

### Possible Participating Systems

An external workflow engine may coordinate bounded operations across:

- **Workday:** confirm or reference the authoritative worker record and employment attributes;
- **Microsoft Entra:** create or enable an identity and assign only approved groups or access packages;
- **SAP:** establish the permitted finance or enterprise-resource record and role mapping;
- **email tooling:** provision a mailbox and approved distribution memberships; and
- **IT workflow tooling:** create fulfillment tasks for equipment, endpoint enrollment, software, or physical-access processes.

These products are illustrative integration points, not required Axion components.

### Capability Constraints

The capability definition could restrict:

- which Workday worker record may be used;
- which Entra tenant, identity type, groups, and access packages are available;
- which SAP company code and role templates are permitted;
- which email domain and distribution lists may be assigned;
- which IT workflow template and equipment profile may be requested;
- the valid relationship among legal entity, location, role, manager, and access profile;
- whether separate approvals are required for privileged or sensitive access; and
- when the grant becomes valid relative to the employee's start date.

The agent sees and invokes the business capability. The mediator and workflow receive only the downstream authority needed for the approved instance. A request to add an unapproved privileged group, change the employee, use another tenant, or invoke an unrelated MCP tool falls outside the grant and requires a new policy decision.

### Execution and Audit

The workflow engine records long-running state and handles asynchronous fulfillment, retries, and compensation. Axion records the initiating identity, capability request, policy decision, approvals, grant, permitted workflow entry point, and subsequent governed transitions.

If onboarding partially succeeds, Axion does not infer that rollback is safe. The workflow definition must identify compensating actions and cases requiring human recovery. Each compensating action remains subject to appropriate authorization.

## Benefits

- Callers and agents request business intent without receiving arbitrary downstream API authority.
- Stable IDs provide an understandable policy and audit vocabulary without functioning as bearer secrets.
- Downstream credentials remain isolated behind brokers or workflow execution identities.
- Grants can bind a one-time decision to identity, tenant, scope, time, policy, and argument constraints.
- Capability definitions create a consistent control point across APIs, MCP tools, and SaaS integrations.
- Identity, policy, mediation, and workflow responsibilities can evolve independently.
- A capability or grant can be suspended without granting broader access or changing the caller's primary identity.

## Drawbacks and Trade-offs

- Capability definition, versioning, lifecycle, and ownership create a new governance burden.
- Business-level capabilities can become too coarse to enforce least privilege or too fine-grained to operate effectively.
- Translating policy constraints into heterogeneous downstream APIs may produce semantic gaps.
- Grant validation, replay protection, revocation, and audit add latency and availability dependencies.
- Workflow and Axion responsibilities may remain ambiguous for short sequences, compensating actions, and asynchronous callbacks.
- Stable capability contracts may slow downstream changes or require compatibility layers.
- An expanding capability catalog can become difficult for humans, agents, and policy authors to navigate safely.

## Security Considerations

### Capability discovery and enumeration

Identifiers are not secrets, but unrestricted discovery can help an attacker map high-value actions. Discovery should be authenticated, policy-filtered, rate-limited, and audited where appropriate. Enforcement must remain correct even when every capability ID is known.

### Replay and duplication

Short expiry alone may not prevent duplicate high-impact actions. Grants and execution endpoints may require nonces, single-use state, idempotency keys, transaction binding, or target-specific deduplication. Not every downstream system provides reliable idempotency.

### Revocation and stale decisions

The design must define whether and how an issued grant can be revoked before expiry, how validators receive revocation state, and what happens during control-plane unavailability. Higher-impact grants may require online validation rather than self-contained offline acceptance.

### Grant theft and audience confusion

A stolen grant could be exercised within its remaining authority. Audience restriction, workload or channel binding, minimal lifetime, proof of possession where justified, secure storage, and narrow scope can reduce exposure. Grants must not be placed in model context, general logs, or durable memory.

### Argument substitution

Authorization must bind to material arguments or enforce explicit ranges. Canonicalization is required before decisions, approvals, signatures, and request digests are compared. Unknown fields, ambiguous identifiers, and target-specific defaults should be rejected or resolved before authorization.

### Confused deputy and delegation

The mediator must preserve the initiating actor, agent, workload, tenant, and delegation context. A highly privileged workflow engine must not treat possession of a valid-looking capability ID as permission to use its own broader authority.

### Capability-definition compromise

Changing a capability definition can change the authority available behind a stable ID. Definitions therefore require versioning, integrity protection, controlled publication, review, rollback, and audit. A grant should bind to the evaluated definition version.

### Downstream enforcement

Targets retain their own authorization and business invariants. Axion mediation narrows reachable authority but does not prove that the workflow is correct, that target behavior is unchanged, or that an approved business action is safe.

## Open Questions

- What grant representation best fits the required revocation, privacy, delegation, portability, and latency properties?
- Should grants be self-contained, reference server-side state, or combine both approaches?
- Which capability attributes must be standardized, and which remain implementation-specific?
- How are capabilities named, versioned, deprecated, discovered, and assigned accountable owners?
- At what granularity should a business intent be divided into separately authorized operations?
- Which downstream argument constraints can be represented and enforced without reproducing workflow logic in Axion?
- When does an atomic mediation sequence become a workflow that must move to an external engine?
- How should long-running workflows refresh authority without turning a short-lived grant into ambient access?
- How are callbacks, retries, compensation, and partial completion bound to the original decision?
- Can grants be attenuated during agent-to-agent or service-to-service delegation without creating inconsistent semantics?
- Which high-impact capabilities require online validation, human approval, dual control, or prohibition?
- What evidence would demonstrate that intent-bound capabilities reduce authority and operational risk compared with direct API permissions?
