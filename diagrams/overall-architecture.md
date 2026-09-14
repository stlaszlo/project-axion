# Axion Overall Architecture

This overview shows how Axion combines distributed observation and reasoning with deterministic authorization and enforcement. It is intended as a concise discussion aid; the repository's design notes define the qualifications and unresolved questions behind each component.

```mermaid
flowchart TB
    subgraph Actors["Actors and autonomous systems"]
        Human["Human"]
        Service["Service"]
        Agent["Agent / LLM\nProbabilistic reasoning"]
        Human --> Agent
        Service --> Agent
    end

    subgraph Axion["Axion — policy and trust orchestration"]
        direction TB

        subgraph Enforcement["Enforcement Plane — inline"]
            Edge["Edge Gateway\nIdentity + request validation"]
            Policy["Deterministic Policy\nAllow · restrict · approve · deny"]
            Grant["Intent-Bound Capability\nShort-lived scoped grant"]
            Broker["Capability / Tool Broker"]

            Edge --> Policy
            Policy --> Grant
            Grant --> Broker
        end

        subgraph TelemetryPlane["Telemetry Plane — inline + out-of-path"]
            Observe["Trusted observations\nIdentity · behavior · performance · security"]
            ECG["Enterprise Capability Graph\nEntities · relationships · baselines · outcomes"]
            Observe --> ECG
        end

        subgraph Reasoning["Reasoning Plane — advisory"]
            Regional["Regional Reasoning\nFast context + correlation"]
            GlobalReasoning["Global Reasoning\nCross-region analysis + investigation"]
            Regional <--> GlobalReasoning
        end

        subgraph Control["Control Plane"]
            Core["Global Axion Core\nPolicy · identity trust · capabilities · revocation"]
            HumanControl["Human Authority\nApproval · interruption · recovery"]
        end

        ECG --> Regional
        ECG --> GlobalReasoning
        Regional -->|"Evidence + uncertainty"| Policy
        GlobalReasoning -->|"Evidence + uncertainty"| Policy
        Core -->|"Signed control state"| Policy
        Core -->|"Governed configuration"| Edge
        HumanControl -->|"Bound approval / override"| Policy
    end

    subgraph Enterprise["Existing enterprise controls and systems"]
        Controls["IAM · PAM · SIEM · EDR · DLP\nAPI Gateways · CI/CD · Cloud Security"]
        Capabilities["APIs · SaaS · MCP Servers · Tools\nData Stores · Vector Databases · Workflows"]
        Outcomes["Business Outcomes"]
        Controls --> Capabilities
        Capabilities --> Outcomes
    end

    Audit["Immutable decision-chain audit\nActor → intent → policy → grant → invocation → outcome"]

    Agent -->|"Untrusted intent / action proposal"| Edge
    Broker -->|"Constrained invocation"| Capabilities
    Capabilities -->|"Result"| Broker
    Broker -->|"Policy-filtered result"| Agent

    Controls -.->|"Telemetry"| Observe
    Capabilities -.->|"Telemetry"| Observe
    Outcomes -.->|"Outcome evidence"| Observe
    Edge -.->|"Requests + decisions"| Observe

    Edge -.-> Audit
    Policy -.-> Audit
    Grant -.-> Audit
    Broker -.-> Audit
    HumanControl -.-> Audit
    Outcomes -.-> Audit
```

## Reading the Diagram

- Models and reasoning engines interpret intent and evidence; their outputs remain advisory.
- Deterministic policy authorizes, constrains, or denies actions at the inline enforcement boundary.
- Axion Capability Grants bind approved business intent to a short-lived execution context.
- The Enterprise Capability Graph combines inline observations with telemetry from systems that Axion does not mediate directly.
- Edge, regional, and global components distribute latency and context without creating different authorization semantics.
- Existing enterprise products retain their domain responsibilities; Axion coordinates them rather than replacing them.
- The audit chain connects the initiating actor to the resulting business outcome.

The most important boundary is the transition from an untrusted action proposal to deterministic authorization. Failure or delay in AI reasoning may reduce capability, but it must not disable enforcement or create authority.
