# Enterprise Capability Graph

## Status and Intent

This note introduces the **Enterprise Capability Graph (ECG)**, the conceptual data model underpinning Axion's reasoning engine. It is a design exploration rather than a finalized schema, storage architecture, graph algorithm, or implementation specification.

The ECG is intended to describe enterprise autonomy: which actors and systems exist, what capabilities they can request or exercise, how authority and data move between them, and what outcomes their activity produces.

## Why a Capability Graph

Autonomous enterprise activity rarely remains within one product or control domain. A human may delegate to an agent, the agent may run as a workload, the workload may invoke an MCP tool, and that tool may call a SaaS API that reads one data store and writes another. Identity, policy, security, application, and business systems each observe only part of this path.

Axion therefore needs a model that can correlate relationships across those domains without treating any single telemetry source as complete. The ECG provides that conceptual model.

Axion continuously constructs and updates the graph by combining:

- **inline observations** from gateways, policy enforcement points, capability brokers, approvals, and governed tool invocations; and
- **out-of-path telemetry** from identity systems, SIEM, EDR, cloud platforms, CI/CD systems, MCP registries, SaaS audit sources, data platforms, and other authorized observation sources.

“Continuously” does not imply immediate global consistency. Sources may be delayed, incomplete, contradictory, duplicated, or unavailable. The graph must preserve provenance, observation time, confidence, and applicable scope so that consumers can distinguish authoritative facts from inferred or stale relationships.

## Design Principle

> Observability provides context. Deterministic controls provide authority. Better context produces better decisions.

The ECG supplies policy-relevant context and supports reasoning, investigation, simulation, and explanation. It does not authorize actions by itself. A graph edge inferred from behavior cannot create a permission, delegation, or trust relationship. Final authorization remains a deterministic decision based on trusted policy inputs and is enforced outside the model reasoning path.

## Conceptual Graph Model

The ECG represents enterprise entities as nodes and their material relationships as typed, directed edges. Nodes and edges may carry attributes such as tenant, owner, source, classification, region, version, observation time, validity interval, confidence, and evidence references.

The model should distinguish at least three categories of graph information:

1. **Authoritative facts** supplied by a system responsible for the relevant domain, such as an identity provider asserting that a workload identity exists.
2. **Observed activity** recorded by an enforcement or telemetry source, such as a workload invoking a tool at a particular time.
3. **Derived relationships or assessments** produced through correlation or reasoning, such as a suspected dependency or changing trust score.

These categories must not be interchangeable. A derived edge may prompt investigation, constrain access, or trigger deterministic policy, but it must not silently become an authoritative grant.

## Graph Nodes

### Humans

Human users, operators, administrators, developers, approvers, and other accountable natural persons. Relevant attributes may include organizational role, tenant, authentication state, employment status, and risk signals. Sensitive identity attributes require minimization and access control.

### Agents

Logical agent identities, including their purpose, owner, configuration version, allowed operating domain, and lifecycle state. An agent remains distinct from the human or service that initiated activity and from the workload executing it.

### Workloads

Processes, services, containers, functions, virtual machines, devices, or other runtime instances that execute actions. Workload nodes may connect logical agents and services to observable infrastructure identities and execution environments.

### Capabilities

Versioned business actions that callers or agents may request, such as `hr.employee.onboard` or `security.workload.isolate`. Capability nodes describe governed intent; knowledge of a capability identifier does not convey authority to invoke it.

### MCP Servers

Model Context Protocol servers that expose tools, resources, or prompts. MCP server nodes should capture operator, deployment, provenance, version, available interfaces, and applicable trust or isolation state without assuming that registration establishes trust.

### Tools

Discrete operations exposed to agents or workloads through MCP, local execution, plugins, brokers, or other interfaces. Tools are capabilities presented through an interface, not trust boundaries. Their descriptions and results remain untrusted inputs unless independently verified.

### APIs

Downstream service interfaces and operations, including internal, partner, cloud, and public APIs. API nodes may represent a service, operation, version, audience, and resource domain at the level required for policy and dependency analysis.

### SaaS Platforms

Externally or internally administered software platforms that provide business functions and authoritative domain controls. A SaaS node may contain tenant-specific subgraphs because identity, configuration, and accessible resources differ across tenants.

### Data Stores

Databases, object stores, document repositories, queues, knowledge stores, and vector databases. Data-store nodes should represent ownership, tenant, location, classification, retention, and interface boundaries. Vector database content and retrieval relationships require particular attention because authorized data may contain malicious or misleading instructions.

### Policies

Versioned deterministic rules, approval requirements, trust configurations, and capability constraints. Policy nodes allow decisions and observed actions to be linked to the exact control state under which they occurred. A natural-language policy proposal is not equivalent to an approved policy node.

### Business Outcomes

The intended or observed organizational result of activity, such as onboarding an employee, containing a compromised workload, approving a payment, or restoring a service. Outcome nodes connect technical activity to its purpose and consequence.

Business outcomes may be difficult to verify and should not be inferred from completion of API calls alone. The graph should distinguish requested, predicted, partially achieved, verified, failed, and reversed outcomes where evidence permits.

## Graph Relationships

Relationships are typed and directional. Their meaning depends on provenance, time, scope, and whether the edge is authoritative, observed, or derived.

### `delegates`

An actor grants bounded authority to another actor or workload. A `delegates` edge requires authoritative evidence of the grant, scope, constraints, and validity. Observed activity must not create delegation implicitly.

### `invokes`

An actor, workload, agent, capability, or tool calls another capability, tool, API, or workflow entry point. The edge may describe an observed invocation, an allowed design path, or both; those meanings must remain distinct.

### `authorizes`

A policy decision permits a bounded action for a specific identity and context. This edge should reference the policy version, decision evidence, obligations, and validity. It must not be inferred solely from the fact that an action occurred.

### `consumes`

An entity uses a service, artifact, event stream, model output, or data product as an input. This relationship can reveal supply-chain and contextual dependencies that are not visible from direct API calls alone.

### `reads`

An actor, workload, tool, API, or capability retrieves data from a store or resource. The graph should preserve resource scope, classification, purpose, and observation window when available.

### `writes`

An actor, workload, tool, API, or capability changes a store, system, or resource. Write relationships are material to impact, rollback, data lineage, and cumulative-risk analysis.

### `depends_on`

An entity requires another entity, control, data source, identity provider, model, capability, or platform for operation. Dependencies may be declared, observed, or inferred and should carry the corresponding evidence type.

### `produces`

An action, workflow, capability, or system produces data, telemetry, an artifact, a decision, or a business outcome. A produced technical artifact does not by itself prove that the intended business outcome was achieved.

## Relationship Context

A single edge label is insufficient for security decisions. Where relevant, relationships should include or reference:

- source and evidence type;
- originating tenant and security domain;
- first-observed and last-observed time;
- effective and expiry time for authoritative relationships;
- frequency, volume, and recent change;
- resource, capability, or argument scope;
- policy and configuration version;
- confidence and derivation method;
- data classification and regional constraints; and
- links to corroborating or conflicting observations.

The graph may need temporal or event-sourced representations rather than overwriting each relationship with only its latest state. The appropriate persistence model remains undecided.

## Independent Baselines

Axion should maintain multiple independent baselines. A single aggregate “normal” or “trust” score would obscure why activity changed and could allow strength in one dimension to cancel material risk in another.

### Identity Baseline

Describes expected initiating actors, agent identities, workload identities, delegation paths, authentication strength, tenant relationships, and administrative roles. Deviations may include a new workload for a known agent, an unusual delegation chain, or a changed identity provider.

### Capability Baseline

Describes which actors and workloads normally discover, request, receive, and invoke specific business capabilities, tools, APIs, and argument scopes. It helps distinguish a known identity from its expected authority and operating purpose.

### Behavioral Baseline

Describes typical action sequences, resource selection, tool combinations, retries, errors, and response handling. Behavioral deviation is an observation, not proof of malicious intent or automatic grounds for privilege expansion.

### Temporal Baseline

Describes expected timing, duration, frequency, ordering, seasonality, and lifecycle phase. Examples include activity outside an employee's working pattern, access before a start date, or a capability invoked after its normal process window.

### Volumetric Baseline

Describes expected counts, rates, fan-out, data volume, spend, concurrency, and cumulative effects. It supports detection of automation loops, bulk extraction, credential abuse, and individually permitted actions that become harmful in aggregate.

### Security Baseline

Describes expected control posture, vulnerabilities, endpoint state, policy decisions, denials, detections, data classifications, and containment conditions. Security telemetry should remain linked to its source and freshness because control products may disagree or report late.

### Trust Baseline

Describes the evidence supporting reliance on identities, tools, platforms, models, data sources, and relationships. Trust is scoped and conditional rather than a permanent property of a node. A tool may be trusted for one operation and tenant while remaining unsuitable for another.

The baselines may interact during reasoning, but their evidence and conclusions should remain independently inspectable. Deterministic policy decides how a change in any baseline constrains, escalates, or denies activity.

## Observation Sources

Axion learns from every trustworthy observation source rather than requiring all traffic to pass through inline gateways.

In this context, “learns” means that Axion updates evidence, relationships, baselines, and derived context. It does not require machine-learning training, and it does not imply that every source is equally trustworthy.

Potential sources include:

- Axion edge gateways and capability brokers;
- identity and privileged-access systems;
- API gateways and service meshes;
- SIEM, EDR, DLP, and cloud security platforms;
- CI/CD and software supply-chain systems;
- SaaS audit and administrative APIs;
- MCP registries and server telemetry;
- workflow engines and event buses;
- databases, vector databases, and data-governance platforms; and
- target applications capable of reporting verified business outcomes.

An inline gateway provides high-quality evidence for the traffic it mediates and can enforce before execution. It cannot observe every relevant identity change, endpoint event, workflow transition, or direct integration. Out-of-path telemetry broadens context, detects bypass and cross-domain patterns, and supports longer-horizon reasoning.

Out-of-path observation does not retroactively prevent an action. When reasoning proposes a response, that response must return through deterministic policy and an authoritative enforcement point.

## Reasoning and Decision Use

The ECG may support:

- reconstructing an identity-to-outcome decision chain;
- identifying unexpected capability and data paths;
- assessing the likely impact of an action or revocation;
- correlating behavior across agents, regions, and control domains;
- comparing activity with independent baselines;
- proposing containment, approval, or investigation steps;
- explaining which evidence informed a policy decision; and
- simulating how policy or capability changes could alter reachable authority.

Model-assisted reasoning may traverse or summarize the graph, but it should receive only authorized and minimized graph views. Its conclusions must preserve evidence references and uncertainty. Deterministic systems decide whether those conclusions may affect authority.

## Security and Governance Considerations

### Provenance and integrity

Every security-relevant fact or derivation needs attributable provenance. Sources require authentication, integrity protection, schema governance, replay and duplication handling, and defined freshness. A compromised source should not silently rewrite the enterprise's trust model.

### Sensitive relationship disclosure

The ECG may reveal privileged identities, high-value systems, undocumented dependencies, data flows, and defensive coverage. Graph queries and exports therefore require least privilege, tenant and regional controls, purpose limitation, audit, and protection against inference across otherwise separated records.

### Poisoning and false correlation

Attackers may generate activity intended to create misleading edges, corrupt baselines, or trigger containment of legitimate systems. Observed frequency must not be confused with legitimacy. Derived relationships should retain confidence, corroboration, and the ability to be challenged or removed.

### Feedback loops

An Axion response can change the graph and influence later reasoning. Automated containment, policy changes, or new observations may create reinforcing loops. Decisions that change authority require deterministic limits, idempotency, rate controls, and human interruption appropriate to their impact.

### Outcome validity

Technical completion events are imperfect proxies for business success. Where outcomes influence policy or learning, Axion should prefer evidence from authoritative business systems or accountable humans and represent uncertainty when verification is unavailable.

## Working Assumptions

- A shared conceptual graph can relate heterogeneous sources without forcing them into one physical database.
- Source systems can provide stable identifiers or resolvable mappings for at least some entities.
- Provenance, tenancy, time, and confidence can be preserved through correlation and derivation.
- Useful security context can be produced even when graph coverage is incomplete.
- Deterministic policy can consume bounded graph facts without depending on unconstrained graph or model inference.
- Organizations can identify authoritative sources for critical identity, policy, capability, and business-outcome facts.

These assumptions require validation in representative enterprise environments.

## Open Research Questions

### Graph aging

- How should observed, declared, and inferred nodes and edges decay or expire?
- Which relationships require event history, effective dates, tombstones, or explicit revocation rather than time-based aging?
- How should Axion distinguish an inactive relationship from one that has become unobservable?

### Confidence scoring

- How should confidence reflect source authority, corroboration, freshness, collection coverage, and derivation depth?
- Should confidence be represented per fact, edge, path, or conclusion?
- How can confidence inform deterministic policy without turning an opaque score into an authorization authority?

### Graph explainability

- How can Axion present the minimal evidence path that caused a conclusion or decision while retaining access controls and uncertainty?
- How should contradictory sources and alternative paths be shown to operators and auditors?
- What explanation remains possible when model-assisted reasoning spans a large subgraph?

### Cross-tenant isolation

- Can shared infrastructure support useful cross-tenant threat intelligence without exposing tenant identities, relationships, behavior, or policy?
- Which nodes and derived indicators may be shared, aggregated, anonymized, or never combined?
- How are deletion, residency, and legal-boundary requirements preserved in derived graph state?

### Efficient updates at very large event volumes

- Which observations warrant graph mutation, aggregation, sampling, or retention only in an event store?
- How are high-rate updates applied without losing ordering, provenance, revocation freshness, or tenant isolation?
- Which baselines require streaming computation, approximate algorithms, or offline recomputation?
- How should regional graph partitions exchange context without requiring global synchronization for every event?
- What consistency guarantees are necessary for reasoning, investigation, and deterministic policy consumption?
