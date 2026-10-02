# Olympus Resource Model Architecture

## Purpose

The Olympus Resource Model provides the canonical representation of entities that Olympus manages, observes, or integrates with across its platform estate.

It provides a common contract that allows the Olympus Control Plane, Command Center, automation, policy systems, and intelligence capabilities to reason about resources consistently.

The model is designed to remain independent of individual cloud providers, runtimes, and execution technologies.

## Architectural Principle

The Olympus Resource Model is the canonical source of truth for platform intent, resource metadata, ownership, and relationships.

Providers and runtimes remain authoritative for actual runtime state.

Conceptually:

```text
                    OLYMPUS RESOURCE
                           |
              +------------+------------+
              |            |            |
           PLATFORM      INTENT       REALITY
           CONTEXT
              |            |            |
          Identity       Spec        Status
          Metadata       Desired     Observed
          Ownership      State        State
          Tags                       Conditions
          Scope                      Reconciliation
          Provenance                 Lifecycle
              |
              +--------------------------+
                                         |
                                  Relationships
                                         |
                                  Management
                                  Governance
                                  Cost
What Is an Olympus Resource?

An Olympus Resource is a uniquely identifiable entity that Olympus needs to manage, observe, relate to, or coordinate as part of the platform estate.

Examples include:

Cloud infrastructure
Networks
Kubernetes clusters
Kubernetes namespaces
Applications
Databases
Messaging systems
DNS records
Certificates
Security resources
Observability resources
GitHub repositories
GitHub Pull Requests
ITSM records
Preview environments

Not every resource is managed by Olympus.

Resource Categories

The initial resource taxonomy includes:

Infrastructure

Examples:

Cloud subscription
Resource group
Virtual network
VPC
Compute resource
Storage
Kubernetes

Examples:

Kubernetes cluster
Namespace
Workload
Service
Ingress
Applications

Examples:

Application
API
Worker
Scheduled workload
Data

Examples:

PostgreSQL
Redis
Object storage
Database cluster
Messaging

Examples:

RabbitMQ
Kafka
Queue
Topic
Networking

Examples:

DNS record
Certificate
Load balancer
Gateway
Identity and Security

Examples:

Identity
Role
Policy
Secret reference
Security resource
Observability

Examples:

Monitoring target
Alert
Dashboard
Observability resource
External Systems

Examples:

GitHub repository
Pull Request
ITSM record
External identity provider
External security system
Ephemeral Resources

Examples:

Preview environment
Per-PR namespace
Temporary test infrastructure
Resource Envelope

Every Olympus resource uses a common conceptual envelope.

Resource
|
+-- Type and Identity
|
+-- Metadata
|   +-- Name
|   +-- Ownership
|   +-- Labels / Tags
|   +-- Environment
|   +-- Placement
|   +-- Provenance
|
+-- Spec
|   +-- Desired State / Intent
|
+-- Status
|   +-- Observed State
|   +-- Conditions
|   +-- Lifecycle
|   +-- Reconciliation
|
+-- Relationships
|
+-- Management
|   +-- Mode
|   +-- Provider
|   +-- Adapter
|   +-- External Reference
|
+-- Governance
|   +-- Policies
|   +-- Security Classification
|   +-- Compliance Profile
|
+-- Cost
|   +-- Estimate
|   +-- Budget
|   +-- Actual Summary
|   +-- Allocation
|   +-- Pricing Source
Type and Identity

Resources have a platform-level type and stable identity.

Conceptually:

apiVersion: platform.olympus.io/v1
kind: Database
id: resource://database/payments-prod

The exact technical representation remains an implementation decision.

Olympus identity must not depend on the provider's native identifier.

Resource Scope

Resources may exist at different scopes.

Examples:

Global
Cloud
Subscription / Account
Region
Environment
Cluster
Namespace
Application
External System

Scope provides context for identity and relationships.

A resource name alone must not be assumed to be globally unique.

Metadata

Metadata provides common platform context.

Examples:

name
owner
team
application
environment
region
cost-center
managed-by
security-classification
compliance-classification

Metadata should be semantically consistent across all providers.

Provider adapters may translate Olympus metadata into native cloud tags or Kubernetes labels.

Ownership

Ownership is a first-class platform concern.

Resources should identify, where applicable:

Individual owner
Team
Application owner
Platform owner
Environment owner

Ownership can be used by:

Authorization
Cost allocation
Governance
Incident response
Routing
Notifications
Spec and Desired State

The spec represents platform intent.

For example:

Database
|
+-- engine: PostgreSQL
+-- version: 16
+-- tier: medium
+-- privateNetwork: true
+-- backup:
|   +-- enabled: true
+-- encryption:
    +-- required: true

Desired state is expected for Olympus-managed resources where Olympus owns the lifecycle.

It is not mandatory for every resource.

Status and Observed State

The status represents what Olympus observes about the resource.

Examples include:

phase: Ready

observed:
  version: 16
  availability: available

conditions:
  Ready: true
  BackupReady: true
  PolicyCompliant: true

Observed information may originate from:

Cloud APIs
Kubernetes APIs
External APIs
Olympus execution mechanisms
Monitoring systems
Conditions

Conditions provide specific state indicators.

Examples:

Ready
Healthy
PolicyCompliant
NetworkReady
BackupReady
Available

Conditions allow the platform to communicate multiple aspects of resource health without reducing the resource to one status value.

Reconciliation

Reconciliation describes Olympus attempting to align desired and observed state.

Conceptually:

Desired State
      |
      v
Reconciler
      |
      v
Observed State

Possible reconciliation states:

Pending
InProgress
Succeeded
Failed
Blocked

A resource may be healthy while reconciliation is not currently active.

Lifecycle

Lifecycle describes the resource's overall existence state.

Possible lifecycle phases include:

Requested
Provisioning
Ready
Updating
Reconciling
Degraded
Failed
Destroying
Deleted
Unknown

Lifecycle and reconciliation are related but distinct.

Operations

Individual operations must be represented separately from the resource's lifecycle.

Example:

Resource:
    phase: Ready

Operation:
    type: Scale
    status: InProgress

This allows a healthy resource to be undergoing an operation without incorrectly changing its entire lifecycle phase to failed or unavailable.

Management Mode

Resources have an explicit management relationship with Olympus.

Managed

Olympus owns the resource lifecycle and may reconcile it against platform desired state.

Observed

Olympus observes the resource but does not own its lifecycle.

Integrated

Olympus coordinates with the resource or external system through an integration contract.

The native external system may remain authoritative.

External References

Managed and integrated resources may carry provider-specific references.

Example:

Olympus:
    resource://database/payments-prod

Azure:
    provider-specific resource identifier

External references are implementation metadata and must not replace Olympus identity.

Relationships

Relationships are first-class.

Examples:

dependsOn
runsOn
contains
uses
belongsTo
createdBy
exposedBy
monitoredBy
ownedBy

Example:

payments-api
    |
    +-- runsOn -------> olympus-prod
    +-- runsIn -------> payments namespace
    +-- dependsOn ----> payments-db
    +-- dependsOn ----> redis
    +-- publishesTo --> rabbitmq
    +-- exposedBy ----> payments.example.com

Relationships should use stable Olympus resource identities.

Resource Graph

The Resource Model enables Olympus to construct a platform resource graph.

Conceptually:

                        olympus-prod
                             |
                       Kubernetes Cluster
                             |
                     +-------+-------+
                     |               |
               payments          customer
                namespace        namespace
                     |
               payments-api
               /      |      \
              /       |       \
             v        v        v
       payments-db   redis   rabbitmq

The graph can support:

Dependency analysis
Impact analysis
Troubleshooting
Security analysis
Cost analysis
Change analysis
AI-assisted investigation
Governance

Resources may reference applicable governance information.

Examples:

policies
securityClassification
complianceProfile

Governance results should be surfaced through status, conditions, findings, or dedicated policy capabilities rather than embedding the complete policy engine in every resource.

Cost and Billing

Cost is a first-class resource concern.

Olympus should distinguish:

Estimated cost

Expected financial impact before or during provisioning.

Actual cost

Observed provider billing information summarized at the resource or allocation level.

Budget

Expected financial boundary.

Allocation

Who or what should receive the cost.

Conceptually:

Cost
|
+-- currency
+-- estimate
+-- budget
+-- actual
+-- allocation
+-- pricingSource
+-- variance

Example:

payments-db

Estimated:
    EUR 145 / month

Budget:
    EUR 200 / month

Actual:
    EUR 81 month-to-date

Allocation:
    Team: payments
    Cost Center: CC-042

Provider billing systems remain authoritative for detailed billing records.

Shared Resource Cost

A shared resource may serve multiple consumers.

Example:

rabbitmq
    |
    +-- payments-api
    +-- customer-api
    +-- notifications-api

The Resource Model must therefore support cost allocation without requiring the resource's entire actual cost to be assigned to every consumer.

Allocation may eventually be based on measurable usage.

Tags and Canonical Metadata

Olympus should maintain a canonical semantic tagging model.

Example:

olympus.io/owner
olympus.io/team
olympus.io/application
olympus.io/environment
olympus.io/cost-center
olympus.io/managed-by
olympus.io/resource-type

The exact tagging namespace and implementation will be defined later.

Provider adapters may translate canonical metadata into:

Azure tags
AWS tags
GCP labels
Kubernetes labels and annotations
OpenStack metadata
Provenance

Every resource should provide enough provenance to understand where its information or intent originated.

Examples:

Git
GitHub
Kubernetes API
Azure API
AWS API
GCP API
OpenStack API
External integration
Olympus generated

Provenance is important for:

Traceability
Drift analysis
Auditing
Resource import
Troubleshooting
Source-of-Truth Model

Olympus deliberately separates platform intent from runtime reality.

                    PLATFORM INTENT
                          |
                    Olympus Resource
                          |
                       Spec
                          |
                          v
                    Control Plane
                          |
                   Reconciliation
                          |
                          v
                 Provider / Runtime
                          |
                          v
                    ACTUAL STATE

Authority is divided as follows:

Olympus Resource Model
    |
    +-- platform identity
    +-- ownership
    +-- relationships
    +-- platform metadata
    +-- platform intent

Provider / Runtime
    |
    +-- actual infrastructure state
    +-- actual runtime state
    +-- native identifiers
    +-- provider-specific operational details

External System
    |
    +-- native external state
    +-- native external lifecycle
Management and Observation Example
Managed Database
Olympus
    |
    +-- desired state
    +-- owner
    +-- cost context
    +-- policies
    |
    +--------> Provider
                    |
                    +-- actual state
Observed Legacy Server
Olympus
    |
    +-- identity
    +-- metadata
    +-- observed state
    +-- relationships
    |
    X-- does not own lifecycle
Integrated GitHub Pull Request
Olympus
    |
    +-- integration metadata
    +-- relationship to application
    +-- relationship to PreviewEnvironment
    |
    +--------> GitHub
                    |
                    +-- PR lifecycle
                    +-- PR status
                    +-- commit
Per-PR Environment

A PreviewEnvironment is a first-class Olympus resource.

Conceptually:

kind: PreviewEnvironment
id: preview/pr-142

metadata:
    owner: payments-team
    environment: ephemeral

spec:
    source: github/pr/142
    targetCluster: olympus-dev
    isolation: namespace
    ttl: 24h

status:
    phase: Ready
    previewURL: ...

It may relate to:

PullRequest
     |
     +-- creates --> PreviewEnvironment
                           |
                           +-- runsOn --> Cluster
                           +-- contains --> Application
                           +-- uses --> Database
                           +-- uses --> Redis

Detailed lifecycle and isolation decisions remain in OLY-001.12.

Resource Versioning

Resource contracts must be versionable.

Conceptually:

platform.olympus.io/v1
platform.olympus.io/v2

Changes to resource schemas must consider:

Backward compatibility
Migration
Conversion
Deprecation
Validation

The implementation mechanism for versioning remains to be decided.

Resource Model and the Control Plane

The Control Plane consumes the Resource Model to make decisions.

                 Resource Model
                       |
        +--------------+--------------+
        |              |              |
      Intent        Reality       Relationships
        |              |              |
        +--------------+--------------+
                       |
                  Control Plane
                       |
        +--------------+--------------+
        |              |              |
      Policy       Workflow     Reconciliation

The Resource Model does not itself become the entire Control Plane.

Resource Model and Command Center

The Command Center consumes the Resource Model to provide:

Resource discovery
Ownership
Status
Dependencies
Cost
Governance
Environment views
Per-PR visibility

Conceptually:

Command Center
      |
      v
Control Plane API
      |
      v
Resource Model

The Command Center does not become the authoritative resource store simply because it displays resource information.

Resource Model and Infrastructure-as-Data

The Resource Model provides the conceptual contract for treating platform state as structured data.

Infrastructure-as-Data will determine how these resource definitions are stored, queried, versioned, reconciled, and exposed to other platform capabilities.

That decision is addressed separately in OLY-001.5.

Resource Model and Execution Technologies

The Resource Model deliberately sits above implementation technologies.

                Olympus Resource
                       |
                 Platform Contract
                       |
          +------------+------------+
          |            |            |
      Terraform    Crossplane    Argo CD
          |            |            |
       Cloud        Resources   Kubernetes
          |
      Ansible

Execution technologies should consume or implement resource contracts rather than redefine the canonical Olympus resource identity.

Resource Model Security Considerations

Resource data may contain sensitive operational information.

The implementation must consider:

Access control
Data classification
Secret references rather than secret values
Sensitive provider identifiers
Tenant boundaries
Auditability
Data minimization

Resources should not embed secrets merely because the underlying resource uses them.

Resource Model Validation

Before a resource type is introduced, it should be possible to answer:

What is the resource?
Who owns it?
Where does it exist?
What does Olympus want?
What actually exists?
Who is authoritative?
What does it depend on?
What depends on it?
How is it managed?
What policies apply?
What does it cost?
Where did the information come from?

These questions form part of the design standard for future Olympus resource types.

Architectural Invariants

The following invariants should remain true:

Olympus resource identity is independent of provider identity.
Providers remain authoritative for actual runtime state.
Managed resources can express platform desired state.
Observed and integrated resources do not require Olympus-owned desired state.
Relationships are first-class.
Ownership and canonical metadata are platform concerns.
Cost context is part of the resource model.
Detailed audit history is separate from the resource.
Provider-specific implementation details remain behind adapters.
Resource contracts are versionable.
Future Considerations

The Resource Model may eventually support:

Resource discovery and import
Automatic dependency discovery
Drift detection
Change impact analysis
Cost anomaly detection
Resource graph queries
Policy-driven resource enrichment
AI-assisted resource investigation
Resource recommendations
Resource lifecycle automation
Cross-cloud migration workflows

These capabilities must preserve the core separation between platform intent and runtime authority.

Related Work Items
OLY-001.2: Define Olympus Control Plane Architecture
OLY-001.3: Define Olympus Command Center Architecture
OLY-001.4: Define Olympus Resource Model
OLY-001.5: Define Infrastructure-as-Data
OLY-001.7: Define Tool Responsibilities
OLY-001.8: Define Security & Trust Boundaries
OLY-001.9: Define Networking Boundaries
OLY-001.10: Define AI / MCP
OLY-001.12: Define Ephemeral / Per-PR Environment Architecture
