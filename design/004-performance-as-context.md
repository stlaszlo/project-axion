# Performance as Context

## Status and Intent

This note explores how Axion may use performance and operational behavior as context for autonomous trust decisions. It is not a performance-monitoring product specification, an infrastructure-optimization design, or a finalized detection model.

## Central Thesis

> **Axion does not monitor performance to optimize infrastructure. It monitors performance because changes in system behaviour provide evidence that influences autonomous trust decisions.**

Performance is relevant to Axion when it helps answer questions about identity, intent, capability use, system integrity, execution confidence, and the safety of continued autonomous operation.

The same signal may have different meanings in different systems. A latency increase may indicate ordinary load, a degraded dependency, policy-engine delay, unexpected tool chaining, data exfiltration, deliberate evasion, or measurement error. Axion should therefore treat performance as contextual evidence rather than a verdict.

## Architectural Boundary

Traditional performance management asks whether a system is fast, available, efficient, and adequately provisioned. Axion asks a narrower security and trust question:

> Does observed operational behaviour change the confidence under which an autonomous actor, capability, or action should be allowed to continue?

Axion may consume performance telemetry from application performance monitoring, cloud platforms, API gateways, service meshes, endpoint controls, workflow engines, MCP servers, model platforms, data systems, and its own enforcement components. Those systems remain responsible for infrastructure optimization, capacity management, service-level objectives, and routine operational alerting.

Axion should not duplicate their dashboards, recommend general capacity changes, or become the system of record for performance operations. It should extract only the signals and derived context needed for governed autonomous activity.

## Why Performance Can Be Security Context

Autonomous systems can change their behavior without changing their declared identity or requested capability. A valid workload may begin invoking tools more frequently, traversing longer dependency chains, processing unusual data volumes, waiting on previously unused services, or repeating partially successful actions.

These changes can reveal conditions that static identity and permission checks do not capture:

- compromised or misdirected agents operating within valid credentials;
- prompt injection causing unexpected tool sequences;
- automation loops, retry storms, or uncontrolled fan-out;
- degraded dependencies that make an otherwise safe workflow unreliable;
- tools or MCP servers behaving differently from their established profile;
- data movement inconsistent with the requested business outcome;
- policy or approval services approaching a state in which safe enforcement cannot be assured; and
- cumulative effects from individually permitted actions.

Performance context does not replace identity, authorization, payload controls, or domain validation. It adds evidence about whether the current behavior remains within the assumptions under which authority was granted.

## Performance Context in the Enterprise Capability Graph

The Enterprise Capability Graph (ECG) relates humans, agents, workloads, capabilities, tools, APIs, data stores, policies, and business outcomes. Performance observations add time-varying evidence to those nodes, relationships, and action paths.

Examples include:

- invocation rate on an `invokes` relationship;
- response latency between a tool and an API;
- read or write volume associated with a capability;
- fan-out from an agent to tools or SaaS platforms;
- queue delay within a long-running workflow;
- retry and error patterns on a dependency path;
- model-call volume associated with one business outcome;
- policy-decision latency and cache behavior at an edge gateway; and
- time between authorization, execution, and verified outcome.

Performance observations should preserve source, tenant, region, measurement interval, collection coverage, units, confidence, and timestamps. Aggregation must not erase short-lived events that are material to security, while raw high-volume telemetry need not become permanent graph state.

## Relevant Signal Categories

### Latency

Changes in request, decision, tool, dependency, or end-to-end outcome latency may indicate altered execution paths, unavailable controls, unexpected external calls, degraded targets, or deliberate delay. Latency must be evaluated against the relevant capability and context rather than a universal threshold.

### Throughput and Volume

Request count, data volume, token usage, records processed, messages produced, and cumulative effects may reveal bulk access, extraction, runaway automation, or a change in business scope. A high rate can be legitimate; the security question is whether it remains consistent with identity, capability, policy, and expected outcome.

### Fan-Out and Concurrency

The number of parallel actions, downstream systems, identities, resources, or regions involved can indicate broader authority use than the business intent appears to require. Sudden fan-out may also amplify mistakes even when each individual action is allowed.

### Error and Retry Behaviour

New error classes, repeated denials, fallback attempts, retries with changed arguments, and partial completion can indicate probing, policy evasion, broken automation, target drift, or ordinary dependency failure. Retrying must not become a path to different or broader authority.

### Resource Consumption

CPU, memory, storage, network, model-token consumption, and cost may provide indirect evidence of unexpected workloads, loops, cryptomining, data transformation, or model behavior changes. These signals are generally supporting evidence because infrastructure conditions can produce similar effects.

### Sequence and Duration

The duration of sessions, workflows, delegations, and capability use—and the ordering between steps—may reveal stalled execution, persistence beyond intended scope, out-of-order actions, or an attempt to operate after context has expired.

### Availability and Control Health

The health of identity, policy, approval, revocation, audit, and broker services affects whether Axion can safely make and enforce decisions. Performance degradation in these controls may require reduced capability even when the requested business action otherwise appears normal.

## Independent Baselines

Performance context should be compared with multiple baselines rather than one global definition of normal.

- **Identity baseline:** expected activity volume, duration, and timing for a human, agent, service, or workload identity.
- **Capability baseline:** expected latency, fan-out, data volume, and downstream dependencies for a particular business action.
- **Behavioral baseline:** expected tool and API sequence, retry behavior, error pattern, and result handling.
- **Temporal baseline:** expected variation by time of day, business cycle, release window, incident state, or lifecycle phase.
- **Volumetric baseline:** expected rate, concurrency, cumulative effect, and resource scope.
- **Security baseline:** expected policy denies, control health, containment state, and security-product signals.
- **Trust baseline:** expected operating characteristics of a model, tool, MCP server, platform, or observation source under a defined scope.

Baselines must remain independently inspectable. An aggregate score should not allow normal behavior in one dimension to cancel a material violation in another. Newly observed behavior is not automatically malicious, and frequently observed behavior is not automatically legitimate.

## From Observation to Decision

Performance context may influence trust handling through a controlled sequence:

1. An authorized source reports a performance observation.
2. Axion validates its provenance, scope, freshness, units, and collection coverage.
3. The observation is correlated with relevant ECG nodes, relationships, capability invocations, and business outcomes.
4. Deterministic logic and, where useful, model-assisted reasoning compare the observation with applicable baselines and identify uncertainty or material change.
5. The reasoning layer proposes a classification or response with supporting evidence.
6. Deterministic policy decides whether to continue, constrain, step up, pause, escalate, quarantine, or deny the activity.
7. An authoritative enforcement point applies the decision and records the result.

A model may explain a complex change or propose a likely cause. It may not convert that interpretation directly into authority. Performance evidence can justify less capability, stronger verification, or human review; it should not independently grant new privilege.

## Possible Trust Responses

Depending on impact, evidence quality, and policy, changed behavior may result in:

- continued execution with additional observation;
- reduced rate, concurrency, duration, or data volume;
- narrower resource or argument scope;
- a requirement for fresh identity or device verification;
- invalidation of a cached allow decision;
- escalation from edge to regional or global reasoning;
- human review or approval;
- suspension of a session, grant, tool, capability, or workload;
- isolation or quarantine; or
- full access denial for sustained or severe policy violations.

Responses should be proportionate, explainable, time-bound where appropriate, and reversible when the evidence no longer supports containment. Policy must also consider whether an attacker could falsify performance signals to cause denial of service.

## Example Scenarios

### Unexpected onboarding fan-out

An approved `hr.employee.onboard` invocation normally reaches a bounded set of HR, identity, finance, email, and IT workflow operations. One invocation begins enumerating many Entra groups and repeatedly calling unrelated MCP tools.

The identity and capability ID remain valid, but the fan-out and dependency path differ from the capability baseline. Axion can invalidate the current grant, deny operations outside the approved capability definition, and require investigation. Performance context helps identify the change; capability constraints and deterministic policy enforce the response.

### Gradual data-volume expansion

An agent authorized to summarize individual support cases remains within per-request permissions but increases its daily case access and result volume over several weeks. No single call violates the static policy.

The volumetric and temporal baselines provide evidence of cumulative scope expansion. Policy may reduce the session limit, require a purpose review, or suspend bulk behavior. The decision should cite the relevant observation window and must not rely on an unexplained anomaly score.

### Degraded policy service

Regional policy-decision latency exceeds the maximum defined for a high-impact action, and the edge cannot establish sufficiently fresh policy or revocation state.

Axion reduces capability rather than bypassing the control. Low-impact operations may continue under explicitly bounded cached decisions if policy allows; the high-impact action pauses or denies. The performance signal describes control health, while established failure policy determines authority.

### Tool behavior drift

A registered MCP tool begins returning substantially larger payloads, using more external calls, and taking a different execution path after a version change.

The changes do not prove compromise, but they weaken the assumptions under which the tool was admitted. Policy may quarantine that version, restrict returned data, or require revalidation. The tool's prior reputation does not override current evidence.

## Security and Governance Considerations

### Measurement integrity

Performance evidence is useful only to the extent that its source and semantics are understood. Sources should be authenticated, time-synchronized where practical, schema-controlled, and monitored for gaps, replay, duplication, and manipulation.

### False positives and operational denial

Normal releases, business peaks, incidents, regional events, and infrastructure changes can alter behavior. Automatic restriction based on weak signals can disrupt legitimate operations. High-impact containment requires corroboration, bounded policy, or human judgment appropriate to the risk.

### Baseline poisoning

An attacker may change behavior gradually to make harmful activity appear normal. Baselines need controlled learning windows, protected reference periods, change detection, and the ability to exclude known incidents or untrusted observations.

### Sensitive operational data

Performance telemetry can reveal business volume, privileged workflows, system topology, customer activity, security controls, and regional operations. Collection and graph correlation require minimization, tenant isolation, regional handling, retention limits, and audited access.

### Causal uncertainty

Correlation does not establish why behavior changed. Axion should separate observation from interpretation and preserve alternative explanations. Where causality matters to a high-impact decision, the system should escalate uncertainty rather than manufacture confidence.

### Feedback loops

Throttling, isolation, and rerouting change the performance signals that triggered them. Axion must identify its own effects, prevent oscillation, use idempotent responses, and define recovery conditions.

## Non-Goals

This design does not make Axion responsible for:

- general infrastructure optimization or capacity planning;
- replacing application performance monitoring or observability platforms;
- managing service-level objectives for participating systems;
- diagnosing every operational fault;
- automatically treating performance anomalies as attacks;
- allowing statistical or model-derived scores to authorize actions; or
- retaining all raw telemetry in the Enterprise Capability Graph.

## Working Assumptions

- Participating systems can expose enough performance context to associate observations with identities, capabilities, or execution paths.
- Baselines can be scoped by tenant, region, capability, and lifecycle rather than treated as global norms.
- Performance evidence can be represented with provenance, freshness, and confidence.
- Deterministic policy can consume bounded contextual facts without depending on opaque model judgments.
- The security value of selected signals can be measured against their collection, privacy, and operational cost.

These assumptions require validation across representative inline and out-of-path use cases.

## Open Questions

- Which performance signals materially improve autonomous trust decisions, and which primarily add security noise?
- What latency and aggregation windows are appropriate for interactive calls, long-running workflows, and cumulative behavior?
- How should Axion distinguish business growth, releases, incidents, and seasonal change from unsafe autonomy?
- Which baselines may update automatically, and which require governed approval or protected reference periods?
- How should sparse or previously unseen identities, tools, and capabilities be handled before a reliable baseline exists?
- What confidence and corroboration are required before performance context can reduce capability or trigger isolation?
- Can performance context ever support a less restrictive decision, or should it only preserve or reduce existing authority?
- How should regional performance evidence be correlated without violating data-residency or tenant-isolation requirements?
- What evidence is necessary to explain a decision derived from multiple time series and graph relationships?
- How can Axion measure whether performance-informed policy reduces material risk without creating unacceptable operational disruption?
