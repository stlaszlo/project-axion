# Security Model

## Scope

The Axion security model governs agent-initiated access to tools, APIs, data, and external effects. Its primary objective is to prevent model reasoning, untrusted context, or a compromised agent runtime from independently acquiring or exercising authority.

It does not eliminate risk from an authorized but mistaken human decision, defects in a target system, or actions whose consequences cannot be determined before execution.

## Security Objectives

- Authenticate and attribute every actor in an action chain.
- Ensure exercised authority is explicitly delegated, least-privilege, current, and independently enforced.
- Contain compromise to the smallest practical identity, capability, resource, time window, and network path.
- Preserve evidence sufficient to explain and investigate each consequential action.
- Require meaningful human control for defined high-impact operations.
- Detect policy bypass, anomalous behavior, identity misuse, and control degradation.

## Identity

Humans, agents, workload instances, services, tools, and administrative components require distinct identities appropriate to their lifecycle. A display name or model-supplied identifier is not identity evidence.

Agent identity describes the logical agent configuration and its authorized role. Workload identity describes the executing instance. User identity describes the initiating actor. These identities may be linked in a delegation chain but must not be conflated. Authentication strength and freshness should match the action's impact.

Identity claims used by policy must come from trusted issuers and be audience-bound. Attributes with uncertain provenance, including model-derived assertions, may inform risk handling but cannot grant authority.

## Authorization

The model produces structured proposals, never authorization decisions. A deterministic PDP evaluates trusted identity, delegation, action, resource, and environmental attributes. A PEP located on the authoritative execution path enforces the result.

Default behavior is deny. Policies should use allow-listed capabilities and bounded resource selectors. Decisions should include the policy version, relevant inputs, reason, validity window, and obligations. Explainability means a qualified reviewer can determine which policy and facts caused a decision; it does not require exposing sensitive policy internals to every caller.

Application and target-system authorization remains in force. Axion authorization narrows access but must not be treated as a substitute for domain controls.

## Credential Lifecycle

Credentials used for execution should be:

- issued only after authorization;
- short-lived and, where practical, single-use;
- restricted by audience, capability, resource, and tenant;
- bound to the executing workload or protected channel where supported;
- absent from prompts, model context, memory, tool descriptions, and ordinary logs;
- revocable or sufficiently short-lived to bound revocation delay.

Credential references may be audited, but secrets and reusable token values must not be logged. Refresh and exchange operations are themselves privileged actions. Implementations must define issuance, storage, rotation, revocation, cache behavior, and destruction.

Legacy systems that require broad or long-lived credentials should be placed behind a broker with narrower logical capabilities, network isolation, strict policy, enhanced monitoring, and explicit residual-risk acceptance.

## Isolation

Isolation is applied across several dimensions:

- separate tenants and security domains;
- isolate agent sessions and memory;
- restrict workload network egress to approved brokers and targets;
- sandbox local tools and constrain filesystem, process, and device access;
- separate control-plane administration from data-plane execution;
- limit concurrency, request volume, spend, and cumulative effects;
- quarantine compromised identities, tools, sessions, or capability versions.

Isolation boundaries must be tested rather than inferred from prompt instructions. A model instruction to avoid a resource is not a security control.

## Audit

Each consequential action should produce a correlated evidence chain containing:

- initiating actor and authentication context;
- agent, model configuration reference, and workload identity;
- delegation chain and session identifier;
- requested capability, target, and policy-relevant parameters;
- policy decision, version, rationale, and obligations;
- approval identity and the exact request digest, if applicable;
- credential issuance reference, excluding secret material;
- broker and target invocation outcome;
- result classification, release decision, and error or compensation events.

Logs require integrity protection, synchronized time, retention rules, access controls, and data minimization. Exact prompts or payloads may be necessary for investigation but may also contain personal, confidential, or malicious content. Retention of full content is therefore an explicit, classified policy choice rather than a universal requirement.

## Human Approval and Interruption

Human approval is a policy obligation for actions whose potential impact exceeds defined thresholds. Approval must be informed and specific: the interface should show the action, target, important parameters, expected effects, uncertainty, and whether rollback is available.

Approval must bind to an immutable request representation, authorized approver, purpose, and expiry. Material changes invalidate it. The requesting actor must not approve its own action where separation of duties is required.

Humans must be able to suspend identities, sessions, capabilities, or queued work. Cancellation of in-flight or irreversible target operations is capability-dependent and must not be promised when the downstream system cannot provide it. Repeated approval prompts can produce fatigue; policy should reserve approval for meaningful decisions and prefer bounded automation for routine low-impact actions.

## Secrets

Secrets are resolved at the execution boundary by the broker or workload, not passed through the model. Secret stores authenticate workloads, enforce access policy, rotate material, and produce access telemetry. Tool output, errors, and retrieved documents are scanned or constrained to reduce inadvertent secret disclosure before entering model context.

Secret detection is imperfect. Preventive controls—non-exportable keys, audience binding, scoped identities, and network restrictions—are preferred over reliance on content scanning alone.

## Observability and Response

Telemetry should support:

- end-to-end correlation across intent, decision, invocation, and result;
- policy deny and obligation trends;
- abnormal tool selection, target access, volume, rate, and cumulative effect;
- delegation and credential anomalies;
- capability, prompt, model, policy, and tool-version changes;
- control-health signals, including missing or delayed audit events;
- containment actions and recovery evidence.

Detection systems may use statistical or model-assisted techniques, but automated containment that changes authority must pass deterministic policy. Telemetry access is itself privileged because it can expose prompts, identities, resources, and organizational behavior.

## Secure Context, Memory, and RAG

Retrieved data and memory retain provenance, classification, tenant, owner, and freshness metadata. Retrieval authorization is evaluated before content enters context, and result-release policy is evaluated before content leaves it. Content is treated as untrusted instructions even when it originates from an authorized repository.

Memory must not serve as a credential store or implicit authorization record. Durable memory requires explicit retention, correction, and deletion semantics. The effectiveness of prompt-injection detection remains limited; containment depends primarily on separating content from authority and re-authorizing every consequential action.

## Local and Hosted Models

The control model applies to both. Local models may reduce some data-transfer risks while adding endpoint, supply-chain, and operational risks. Hosted models may provide stronger managed controls while introducing third-party processing, residency, availability, and contractual considerations. Model location does not change the requirement for external authorization and brokered capabilities.

## Open Questions

- Which audit fields are mandatory across all risk tiers, and which require content retention?
- What assurance levels should agent and tool identities support?
- How should cumulative-risk policies govern a series of individually low-impact actions?
- What is the maximum acceptable revocation and telemetry delay for each action class?
- Which isolation mechanisms are required for local code-execution tools?
- How should policy conflicts across organizations and SaaS providers be resolved?
