# Diagrams

This directory contains Mermaid source files for the conceptual architecture. The diagrams are explanatory views, not deployment topologies or claims about specific products.

## Files

- [architecture.mmd](architecture.mmd) — principal actors and the end-to-end control flow.
- [trust-boundaries.mmd](trust-boundaries.mmd) — the boundaries crossed as intent becomes an authorized external action and evidence is recorded.

## Rendering

Render with any Mermaid-compatible viewer or CLI. For example:

```sh
mmdc -i diagrams/architecture.mmd -o architecture.svg
mmdc -i diagrams/trust-boundaries.mmd -o trust-boundaries.svg
```

Rendered files are intentionally not committed at this stage so the sources remain the authoritative artifacts and do not imply a preferred documentation toolchain.

## Interpretation

Solid arrows show principal request or result flow. Dotted arrows show control-plane configuration or audit emission. The most important boundary is **reasoning to action**: a model-created proposal has no authority until the deterministic authorization and enforcement path permits it.

Future diagrams may add deployment variants, sequence flows, multi-agent delegation, credential exchange, and isolation patterns once the relevant design choices are sufficiently defined.
