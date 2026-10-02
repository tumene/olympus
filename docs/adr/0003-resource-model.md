# ADR-0003: Olympus Resource Model

## Status

Accepted

## Date

2026-10-02

## Context

Olympus is designed as a cloud-agnostic Internal Developer Platform and operational control plane.

The platform needs a consistent way to represent infrastructure, Kubernetes resources, applications, platform services, external systems, and ephemeral environments across multiple providers and runtimes.

A provider-specific resource model would tightly couple Olympus to individual technologies and make cross-platform management difficult.

A single universal schema containing provider-specific fields for every possible resource would create excessive complexity and weaken the platform abstraction.

Olympus therefore requires a common resource contract with resource-specific specifications and status information.

The model must also distinguish between platform intent and runtime reality.

## Decision

Olympus will use a canonical Resource Model based on a common resource envelope with resource-specific specifications and status.

The Olympus Resource Model is the canonical source of truth for:

- Platform resource identity
- Platform metadata
- Ownership
- Platform relationships
- Platform intent / desired state for Olympus-managed resources
- Management mode
- Provenance
- Platform governance references
- Resource-level financial context

Underlying providers and runtimes remain authoritative for:

- Actual runtime state
- Provider-specific operational state
- Runtime-generated identifiers
- Provider-specific execution details
- Native billing records
- Native audit records

Olympus will observe and reconcile runtime state against platform intent where Olympus has management authority.

## Canonical Resource Envelope

An Olympus resource will conceptually contain:

- Type and identity
- Metadata
- Ownership
- Scope and placement
- Spec / desired intent
- Status / observed state
- Conditions
- Reconciliation state
- Relationships
- Management information
- Governance information
- Cost and billing context
- Provenance

Audit history will be maintained as a separate capability and may be referenced by resources rather than embedded as a complete history within each resource.

## Management Modes

Olympus resources will support explicit management modes:

### Managed

Olympus has lifecycle authority over the resource.

Managed resources normally have platform desired state and are subject to Olympus reconciliation.

### Observed

Olympus inventories and observes the resource without assuming lifecycle authority.

Desired state may be absent because an external system remains authoritative.

### Integrated

Olympus coordinates with the resource or external system through a defined integration contract.

The external system may remain authoritative for the native lifecycle and state.

## State Model

The resource model separates:

### Desired State

What Olympus intends for a managed resource.

### Observed State

What Olympus currently observes from the runtime, provider, or external system.

### Conditions

Specific state indicators such as:

- Ready
- Healthy
- PolicyCompliant
- BackupReady
- NetworkReady

### Reconciliation State

The state of Olympus attempting to align desired state with observed state.

Conceptual reconciliation states include:

- Pending
- InProgress
- Succeeded
- Failed
- Blocked

### Lifecycle State

The current resource lifecycle phase.

Conceptual lifecycle phases include:

- Requested
- Provisioning
- Ready
- Updating
- Reconciling
- Degraded
- Failed
- Destroying
- Deleted
- Unknown

### Operation State

Individual operations are modeled separately from the resource's overall lifecycle state.

For example, a resource may be:

- Lifecycle: Ready
- Operation: Scaling
- Operation status: InProgress

## Resource Identity

Olympus will maintain a platform identity independent of provider identity.

A resource may therefore have:

- Olympus resource identifier
- Resource kind
- Resource name
- Resource scope
- Provider
- Provider-specific external reference

Olympus identity must not depend on the provider-specific identifier.

## Scope

Resources may exist at different scopes, including:

- Global
- Cloud
- Account / subscription
- Region
- Cluster
- Namespace
- Environment
- Application
- External-system scope

Scope is part of the resource identity model and allows resources with the same local name to remain unambiguous.

## Ownership and Metadata

Resource metadata will support platform-wide semantics for:

- Owner
- Team
- Application
- Environment
- Cost center
- Managed by
- Security classification
- Compliance classification
- Region / location
- Platform labels and tags

Olympus will define canonical semantic metadata where practical.

Provider-specific adapters may translate canonical metadata into provider-specific tags or labels.

## Relationships

Relationships are first-class parts of the resource model.

Examples include:

- dependsOn
- runsOn
- contains
- uses
- belongsTo
- createdBy
- exposedBy
- monitoredBy
- ownedBy

Relationships must reference stable Olympus resource identities.

## Governance

Resources may reference applicable:

- Policies
- Security classifications
- Compliance profiles
- Governance requirements

Policy evaluation results should be represented through resource status, conditions, findings, or dedicated governance capabilities rather than embedding an entire policy engine inside each resource.

## Cost and Billing

Cost and financial context are first-class resource concerns.

Where supported, Olympus should represent:

- Estimated cost
- Actual cost summary
- Currency
- Budget
- Threshold
- Cost center
- Allocation
- Pricing source
- Cost variance

Olympus resource cost information is a platform-level financial view.

Native provider billing systems remain authoritative for detailed billing records.

Shared resources may support cost allocation across consumers where sufficient usage data exists.

## Provenance

Olympus resources must be traceable to the source from which their information or intent originated.

Possible provenance sources include:

- Git
- GitHub
- Kubernetes API
- Cloud provider APIs
- External systems
- Olympus-generated resources
- Other approved platform integrations

Provider and execution details are recorded as provenance or management information rather than defining the resource's core identity.

## Provider Independence

The core Olympus Resource Model must remain independent of:

- Azure
- AWS
- GCP
- OpenStack
- Kubernetes
- Terraform
- Crossplane
- Ansible
- Argo CD

Provider-specific capabilities are represented through resource-specific specifications, provider adapters, external references, or execution metadata.

## Source-of-Truth Model

The following authority model applies:

| Concern | Authoritative source |
|---|---|
| Olympus resource identity | Olympus Resource Model |
| Olympus metadata and ownership | Olympus Resource Model |
| Olympus relationships | Olympus Resource Model |
| Platform intent | Olympus Resource Model / declared source |
| Provider actual state | Provider |
| Kubernetes runtime state | Kubernetes |
| External-system native state | External system |
| Detailed provider billing | Provider billing system |
| Detailed audit history | Olympus audit capability and relevant authoritative systems |

Olympus does not replace provider or runtime systems as the authoritative source of actual runtime reality.

## Consequences

### Positive

- Provides a consistent platform abstraction across providers
- Enables cross-cloud resource discovery
- Supports resource dependency graphs
- Separates intent from runtime reality
- Enables reconciliation and drift detection
- Supports cost, governance, and ownership visibility
- Allows the Command Center and AI capabilities to consume common resource contracts
- Keeps provider-specific implementations behind adapters

### Negative

- The resource model becomes a foundational platform contract
- Resource schemas require versioning and backward-compatibility discipline
- Relationships and reconciliation introduce additional platform complexity
- Accurate observed state depends on provider and runtime integrations
- Cost information may vary in freshness and precision across providers

## Rejected Alternatives

### Provider-native resource model

Using Azure, AWS, GCP, or another provider's native resource model as the Olympus canonical model.

Rejected because it would couple Olympus to a specific provider and make cloud-agnostic behavior difficult.

### Kubernetes-only resource model

Representing every Olympus resource exclusively as Kubernetes resources or CRDs.

Rejected because Olympus must represent resources that exist outside Kubernetes, including cloud infrastructure, external systems, billing, and enterprise integrations.

### Universal provider-specific schema

Creating one large schema containing fields for every provider and resource type.

Rejected because it would create unnecessary coupling and complexity while weakening the common platform contract.

### Inventory-only model

Maintaining only an inventory of resources without platform intent.

Rejected because managed resources require desired state, reconciliation, lifecycle coordination, and policy enforcement.

## Architectural Principles

- Every Olympus resource has a stable platform identity.
- Managed resources may declare desired state.
- Observed and integrated resources may not have Olympus-owned desired state.
- Providers and runtimes remain authoritative for actual runtime state.
- Relationships are first-class.
- Canonical metadata is provider-independent where practical.
- Cost and billing context are first-class resource concerns.
- Detailed audit history remains a separate capability.
- Provider-specific implementation details remain behind platform boundaries.
- Resource contracts must be versionable.

## Related Work Items

- OLY-001.2: Define Olympus Control Plane Architecture
- OLY-001.3: Define Olympus Command Center Architecture
- OLY-001.4: Define Olympus Resource Model
- OLY-001.5: Define Infrastructure-as-Data
- OLY-001.7: Define Tool Responsibilities
- OLY-001.8: Define Security & Trust Boundaries
- OLY-001.9: Define Networking Boundaries
- OLY-001.10: Define AI / MCP
- OLY-001.12: Define Ephemeral / Per-PR Environment Architecture