# Axion Foundations

These foundations define the architectural constraints against which Axion designs should be evaluated. They are intended to remain stable even as implementations, models, vendors, protocols, and deployment patterns change.

They are not a claim that a short list can make autonomous systems safe. Each foundation creates design obligations that must be tested in the context of a specific use case, threat model, and operating environment.

## 1. Models Reason. Systems Authorize.

Models may interpret intent, correlate evidence, identify uncertainty, and propose actions. They must not make the final authorization decision for an action carrying real-world authority.

Probabilistic reasoning must operate within deterministic policy and enforcement boundaries. Model output is an input to a decision, not proof of permission.

## 2. Identity Before Capability

No actor should receive or exercise a capability before its identity, role, delegation, and execution context are established.

Human, agent, workload, service, and tool identities should remain distinguishable throughout the action chain. Capability discovery must not be treated as authorization to use that capability.

## 3. Least Privilege by Default

Authority should be limited by actor, capability, resource, purpose, tenant, audience, time, and context. Credentials should be short-lived and narrowly scoped where the target system permits it.

Broader access requires an explicit justification and acceptance of the resulting residual risk. It must not emerge from convenience, fallback behavior, or limitations in model reasoning.

## 4. Failure Reduces Capability

Failure, ambiguity, stale state, or loss of a security-relevant dependency must not increase privilege or bypass controls.

Degraded modes should be predefined, bounded, time-limited, and observable. A system should not fall back to shared credentials, unmediated access, default allow, or unaudited execution.

## 5. Audit the Decision Chain Continuously

Audit must preserve the chain from initiating intent to external effect: initiating actor, agent and workload identity, delegation, requested action, policy inputs and decision, approval, tool invocation, and result.

Audit is an ongoing control, not only a forensic record. Missing, delayed, inconsistent, or unexplained events should be detectable. Evidence collection must still respect data minimization, confidentiality, and retention requirements.

## 6. Prefer Orchestration Over Replacement

Axion should coordinate and extend existing enterprise controls before introducing a new system to duplicate them. IAM, PAM, API gateways, SIEM, EDR, DLP, cloud security, CI/CD, and target applications should remain authoritative in the domains where they are well positioned to protect.

A new component is justified when existing controls cannot meet a defined security or operational requirement, not merely because consolidation appears simpler. Reuse must not force an existing tool into a role for which its trust model or enforcement position is unsuitable.

## 7. Keep Complexity Inside the Platform

Necessary integration, policy, identity, telemetry, and reasoning complexity should be absorbed by the platform rather than transferred to routine customer operations.

This does not mean hiding consequential behavior. Operators must retain clear controls, understandable failure modes, actionable explanations, and supported recovery procedures. Internal sophistication is acceptable only when it reduces—not obscures—operational burden.

## 8. Make Security Decisions Explainable

An authorized reviewer should be able to determine what decision was made, which trusted facts and policy caused it, what obligations applied, and which component enforced it.

Model-generated explanations may improve presentation but are not the authoritative decision record. At the end of the accountability chain is a human: someone must be able to understand, challenge, and take responsibility for the policy and its consequences.

## 9. Preserve Human Authority for High-Impact Actions

Humans must retain meaningful authority to approve, deny, interrupt, constrain, or revoke high-impact autonomous activity.

Human involvement should be specific and informed, not a ceremonial confirmation or a stream of low-value prompts. Approval must bind to the material action, target, scope, consequences, and expiry. Irreversible actions require particular caution because interruption may no longer be possible after execution begins.

## 10. Design for Vendor and Model Change

The architecture should preserve its security properties when models, model providers, cloud platforms, tools, protocols, and enterprise products change.

Interfaces and control responsibilities should be explicit and portable. Vendor-specific implementations are expected, but they must not become implicit trust assumptions or make core policy semantics inseparable from one provider.

## 11. Escalate Containment Predictably

Every governed activity should have a defined initial authority, acceptable behavior envelope, and response path for policy violations.

Repeated or sustained violations should trigger proportionate containment: increased observation, reduced rate or scope, step-up verification, suspension, quarantine, and, when warranted, full access denial. Escalation must be deterministic, attributable, reversible where appropriate, and resistant to attackers using false signals to cause denial of service.

Containment is applied to identities, sessions, workloads, capabilities, or resources—not as punishment of a model, but as risk reduction for the system.

## 12. Optimize for Meaningful Outcomes

Axion should reduce material risk and improve the control of autonomous activity. It should not be evaluated by the number of alerts, policies, integrations, model calls, or automated responses it produces.

Controls and telemetry should support actionable outcomes such as prevented unauthorized effects, reduced exposure, faster containment, clearer accountability, and safer recovery. Additional security noise is a cost and may conceal the events that matter.

## Applying the Foundations

A proposed Axion capability should be challenged with four questions:

1. Which foundation does the design support, and how is that support demonstrated?
2. Does it weaken another foundation or move risk into a less visible part of the system?
3. What assumptions must hold for the control to work?
4. What measurable outcome would show that the design improves security or operations?

Where a design cannot satisfy a foundation, the exception should be explicit, bounded, reviewed by an accountable human, and recorded with its residual risk.
