# ADR-004: Tools Are Capabilities, Not Trust Boundaries

## Status

Accepted

## Context

Agents interact with tools described through schemas, metadata, plugins, MCP servers, local executables, or service adapters. A tool description states an interface and claimed behavior; it does not establish that the implementation is safe, unchanged, correctly scoped, or non-malicious.

## Decision

Treat each tool operation as a capability that must be registered, identified, constrained, authorized, and mediated. Trust is established by surrounding identity, provenance, isolation, policy, broker, and target controls—not by the tool's presence in an agent context or its self-description.

Capability authorization applies to the specific operation, target, and policy-relevant parameters. Results are treated as untrusted content and are subject to release controls before entering further reasoning or reaching a user.

## Rationale

This model prevents tool discovery from becoming permission and limits the damage caused by deceptive descriptions, compromised servers, unsafe adapters, or excessive tool scope. It also creates a consistent control model across local tools, SaaS APIs, MCP, and future agent protocols.

## Consequences

### Positive

- Tool availability can be limited by identity, session, resource, and risk.
- Tool changes can be versioned, reviewed, revoked, and audited.
- Inputs, outputs, network access, and credentials can be independently constrained.
- The architecture does not depend on a single tool protocol.

### Negative / Trade-offs

- Capability registration and adapter governance add operational overhead.
- Fine-grained operations and schemas may be difficult to define for general tools.
- Sandboxing and output controls vary by platform and cannot eliminate all risk.
- Dynamic tool ecosystems conflict with strict allow-listing and review.

## Alternatives Considered

- **Trust tools installed or discovered from an approved source.** Rejected because provenance alone does not constrain runtime behavior or future compromise.
- **Let the model decide whether a tool is safe.** Rejected because safety and authorization cannot rest on probabilistic interpretation.
- **Treat a tool server as one coarse permission.** Rejected as the default because operations within one server can have materially different impact; it may be an explicitly accepted limitation for legacy systems.

## Open Questions

- How should capability identity, version, provenance, and revocation be standardized?
- What minimum isolation is required for local executable tools?
- How can dynamic MCP or agent-to-agent capabilities be admitted without weakening review?
- Which semantic parameter constraints can be enforced consistently across adapters?
