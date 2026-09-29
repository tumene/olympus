# ADR-0002: Control Plane-Native Architecture

## Status

Accepted

## Date

2026-09-29

## Context

Olympus is intended to operate as a cloud-agnostic Internal Developer Platform and operational control plane.

The platform will need to support multiple interaction mechanisms, including a web interface, CLI, APIs, automation, ChatOps, and AI-assisted operations.

Implementing platform logic independently inside each interface would create duplicated behavior, inconsistent authorization, fragmented policy enforcement, and multiple sources of operational truth.

Olympus therefore requires a single canonical platform interaction layer.

## Decision

Olympus will use a Control Plane-native architecture.

The Olympus Control Plane will be the canonical platform interaction and orchestration layer.

The Control Plane will expose well-defined contracts for:

- Resource management
- Desired state
- Observed state
- Reconciliation
- Policy
- Authorization
- Workflow
- Approvals
- Dependencies
- Audit
- Platform operations

The Command Center will provide the unified operator experience over the Control Plane.

The Control Plane will support multiple clients and interfaces, including:

- Web UI
- CLI
- APIs
- ChatOps
- AI agents through MCP
- Platform automation

All clients will consume the same platform contracts and must not implement independent platform lifecycle logic.

Execution systems such as Terraform, Crossplane, Ansible, and Argo CD will be treated as specialized implementation mechanisms behind the Control Plane.

## Architectural Model

```text
                       OLYMPUS
                          │
                  ┌───────▼────────┐
                  │  CONTROL PLANE │
                  └───────┬────────┘
                          │
                  Control Plane API
                          │
        ┌─────────┬───────┼───────┬─────────┐
        │         │       │       │         │
      Web UI     CLI     API    AI/MCP   Automation
        │         │       │       │         │
        └─────────┴───────┴───────┴─────────┘
                          │
                          ▼
                Execution / Adapters
                          │
             ┌────────────┼────────────┐
             │            │            │
         Terraform    Crossplane    Argo CD
             │            │            │
          Ansible     Cloud APIs   Kubernetes

## Consequences

### Positive

- Provides one canonical platform interaction model
- Prevents duplication of platform lifecycle logic across interfaces
- Enables consistent authentication, authorization, policy, and approval controls
- Allows Web UI, CLI, APIs, ChatOps, and AI to evolve independently
- Creates a stable abstraction above implementation technologies
- Allows execution technologies to be replaced without changing the user-facing platform contract
- Provides a natural foundation for AI-assisted operations through MCP

### Negative

- The Control Plane becomes a critical platform component
- Control Plane API and resource contracts require careful design
- The Control Plane introduces additional architectural and operational complexity
- Availability, security, scalability, and recovery of the Control Plane become important platform concerns
- Integration failures between the Control Plane and execution systems must be handled explicitly

## Rejected Alternatives

### Web-first architecture

A web application would act as the primary implementation of platform logic, with other interfaces consuming the web application's capabilities.

**Rejected because:** platform behavior would become tightly coupled to the web application and would encourage duplication when additional interfaces are introduced.

### Independent interface implementations

Each interface would implement its own platform operations and lifecycle logic.

**Rejected because:** this could result in inconsistent behavior, authorization, policy enforcement, and resource semantics.

### Tool-first architecture

Terraform, Crossplane, Ansible, Kubernetes, or Argo CD would effectively become the primary Olympus control interface.

**Rejected because:** these technologies solve specific infrastructure, configuration, reconciliation, or workload-management problems. They should not define the Olympus user-facing platform contract.

## Architectural Principles

- Interfaces are clients of the Control Plane, not independent control systems.
- Platform lifecycle logic belongs in the Control Plane rather than individual interfaces.
- All operations must pass through appropriate authentication, authorization, policy, and approval controls.
- The Control Plane owns platform intent and orchestration but does not replace the authoritative runtime systems.
- Execution technologies remain replaceable implementation mechanisms where practical.
- AI agents must interact through controlled platform interfaces and must not bypass platform governance.

## Related Work Items

- OLY-001.2: Define Olympus Control Plane Architecture
- OLY-001.3: Define Olympus Command Center Architecture
- OLY-001.4: Define Olympus Resource Model
- OLY-001.5: Define Infrastructure-as-Data
- OLY-001.10: Define AI / MCP