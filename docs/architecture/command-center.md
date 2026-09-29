# Olympus Command Center Architecture

## Purpose

The Olympus Command Center is the primary experience layer for interacting with the Olympus platform.

It provides humans and authorized automation with a unified way to discover, understand, and request operations against platform resources.

The Command Center does not independently implement platform lifecycle logic. It consumes the contracts exposed by the Olympus Control Plane.

## Architectural Principle

The Command Center is a client of the Olympus Control Plane.

All platform operations initiated through the Command Center must pass through the appropriate:

- Authentication
- Authorization
- Policy evaluation
- Approval controls
- Control Plane workflows

The Command Center must not bypass the Control Plane to directly manipulate infrastructure or runtime systems.

## High-Level Architecture

```mermaid
flowchart TD
    Users[Users / Operators]

    Users --> UI[Web UI]
    Users --> CLI[CLI]
    Users --> API[API Clients]
    Users --> ChatOps[ChatOps]
    Users --> AI[AI / MCP]

    UI --> CPAPI[Control Plane API]
    CLI --> CPAPI
    API --> CPAPI
    ChatOps --> CPAPI
    AI --> CPAPI

    CPAPI --> Auth[Authentication / Authorization]
    Auth --> Policy[Policy Engine]
    Policy --> Approval[Approval Workflow]
    Approval --> CP[Olympus Control Plane]

    CP --> Resource[Resource Model]
    CP --> Workflow[Platform Workflows]
    CP --> Reconcile[Reconciliation]
    CP --> Audit[Audit / Events]

    CP --> Exec[Execution / Integration Layer]

    Exec --> Terraform[Terraform]
    Exec --> Crossplane[Crossplane]
    Exec --> Ansible[Ansible]
    Exec --> Argo[Argo CD]
    Exec --> External[External Systems]

## Primary User Personas

### Developer

Primary needs:

- Create and manage development environments
- Create and inspect per-PR environments
- Deploy application versions
- View application health
- Inspect relevant logs and metrics
- Understand application dependencies

Developers should only have access to resources and operations permitted by their identity, team, environment, and platform policies.

### Platform Engineer

Primary needs:

- Manage platform infrastructure
- Manage clusters and platform services
- Investigate platform health
- Manage platform configuration
- Perform controlled operational actions
- Review resource dependencies and drift

### SRE / Operations

Primary needs:

- Monitor service health
- Investigate incidents
- Trace dependencies
- Inspect logs, metrics, events, and changes
- Execute approved operational procedures
- Support recovery and remediation

### Security / Compliance

Primary needs:

- Review security posture
- Review policy violations
- Review identity and access
- Inspect audit history
- Review compliance controls
- Investigate security events

### Administrator / Approver

Primary needs:

- Review high-risk operations
- Approve or reject sensitive changes
- Manage platform governance
- Manage privileged access

### AI Agent

Primary needs:

- Query platform information
- Correlate resources, events, logs, and metrics
- Investigate incidents
- Propose remediation
- Request authorized operations

AI agents must operate through controlled Olympus interfaces and must not bypass platform authorization, policy, or approval mechanisms.

## Command Center Capabilities

The Command Center should organize information around platform concepts rather than implementation technologies.

### Resources

The Command Center should provide visibility into:

- Applications
- Environments
- Clusters
- Infrastructure
- Networking
- Data services
- Messaging
- Security resources
- Observability resources
- External systems

### Operations

The Command Center may expose operations such as:

- Provision
- Deploy
- Update
- Scale
- Restart
- Roll back
- Reconcile
- Destroy

Operations are requests to the Control Plane and are subject to authentication, authorization, policy, and approval.

### Visibility

The Command Center should provide:

- Resource status
- Desired state
- Observed state
- Health
- Dependencies
- Ownership
- Events
- Logs
- Metrics
- Security findings
- Compliance status
- Cost information
- Audit history

### Resource Graph

The Command Center should eventually provide a relationship-oriented view of resources.

Example:

payments-api
    |
    +-- runs-on --------> olympus-prod
    |                         |
    |                         +-- Kubernetes Cluster
    |
    +-- runs-in --------> payments namespace
    |
    +-- depends-on -----> payments-db
    |
    +-- depends-on -----> redis
    |
    +-- publishes-to ---> rabbitmq
    |
    +-- exposed-by -----> payments.example.com
    |
    +-- monitored-by ---> observability platform

The resource graph is intended to support:

- Impact analysis
- Troubleshooting
- Security analysis
- Cost analysis
- Dependency analysis
- AI-assisted investigation

The resource model and relationship semantics are defined separately in OLY-001.4.

### Per-PR Environments

Per-PR environments are a first-class Olympus platform capability.

The Command Center should provide visibility into:

- Pull request
- Repository
- Commit
- Application
- Environment
- Target cluster
- Namespace
- Preview URL
- Provisioning status
- Resource status
- Security status
- Cost
- TTL
- Lifecycle events

Example:

PR #142
    |
    +-- Preview Environment
            |
            +-- Cluster: olympus-dev
            +-- Namespace: pr-142
            +-- Application: payments-api
            +-- Status: Healthy
            +-- Preview URL
            +-- Cost
            +-- Lifecycle / TTL

The detailed lifecycle, isolation, networking, security, provisioning, and cleanup architecture for per-PR environments is defined separately in OLY-001.12.

### Authorization and Operations

The Command Center follows this conceptual request flow:

User / Automation
       |
       v
Command Center
       |
       v
Authentication
       |
       v
Authorization
       |
       v
Policy Evaluation
       |
       +----> Denied
       |
       +----> Approved
       |
       +----> Approval Required
                   |
                   v
             Human Approval
                   |
                   v
             Control Plane
                   |
                   v
          Execution / Adapter

The Command Center must not provide a direct administrative path around these controls.

Authorization determines what a caller may request.

Policy determines whether the requested operation is permitted.

Approval determines whether an additional human authorization step is required.

### Interfaces

The following interfaces may consume the Olympus Control Plane:

Interface	Primary Use
Web UI	Human platform experience
CLI	Developer and operator workflows
API	Programmatic integration
ChatOps	Operational collaboration
AI / MCP	AI-assisted discovery and operations
Automation	Programmatic platform workflows

All interfaces should use the same underlying Olympus resource and operation contracts.

### Command Center Boundaries

The Command Center owns
- User experience
- Resource discovery
- Visualization
- Operation requests
- Presentation of platform state
- Interaction workflows

The Command Center does not own
- Cloud provider implementation
- Terraform state
- Kubernetes runtime state
- Infrastructure credentials
- Secrets
- Provider-specific provisioning logic
- Independent lifecycle logic
- Policy bypass mechanisms

The Command Center is an experience layer and does not become an alternative control system.

### Relationship With the Control Plane

The Command Center and Control Plane are separate architectural responsibilities.

The Command Center answers:

How do humans and authorized clients interact with Olympus?

The Control Plane answers:

What should happen, is it allowed to happen, and how should Olympus coordinate it?

Conceptually:

                    COMMAND CENTER
                  Experience Layer
                          |
                          | Control Plane API
                          v
                   AUTHENTICATION
                          |
                          v
                   AUTHORIZATION
                          |
                          v
                       POLICY
                          |
                          v
                      APPROVAL
                          |
                          v
                    CONTROL PLANE
                   Decision / State
                          |
             +------------+------------+
             |            |            |
             v            v            v
        Resource       Workflow    Reconciliation
          Model

The Control Plane remains the canonical platform interaction and orchestration layer.

### Relationship With Execution Systems

Execution technologies are implementation mechanisms behind the Control Plane.

Conceptually:

                 Olympus Control Plane
                          |
                   Platform Contract
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
          Terraform   Crossplane   Argo CD
              |           |           |
              v           v           v
           Cloud       External    Kubernetes
         Resources     Resources    Workloads

                    Ansible
                       |
                       v
              OS / Configuration

The Command Center does not directly invoke provider APIs as an alternative execution path.

### AI and MCP

AI-assisted operations are treated as another controlled client of Olympus.

Conceptually:

                         AI Agent
                            |
                           MCP
                            |
                            v
                    Control Plane API
                            |
                 +----------+----------+
                 |          |          |
                 v          v          v
             Resource     Policy     Workflow
              Model

AI agents may:

- Query resources
- Investigate incidents
- Correlate logs and metrics
- Analyze dependencies
- Identify potential causes
- Propose remediation
- Request authorized operations

AI agents must not:

- Bypass authorization
- Bypass policy
- Bypass required approvals
- Access credentials directly
- Execute unrestricted destructive operations

AI-assisted operations are governed by the same platform controls as other Olympus clients.

### Per-Environment Experience

The Command Center should provide a consistent experience across:

- Development
- Staging
- Production
- Ephemeral / per-PR environments

For each environment, the user should be able to understand:

Environment
    |
    +-- Owner
    +-- Lifecycle
    +-- Cluster
    +-- Applications
    +-- Networking
    +-- Data
    +-- Security
    +-- Observability
    +-- Cost
    +-- Recent Changes
    +-- Events

Environment-specific permissions and policies must be enforced through the Control Plane.

### External Systems

The Command Center may present selected external systems through Olympus integrations.

Examples include:

- GitHub
- ITSM platforms
- Identity providers
- DNS providers
- Security platforms
- Notification systems
- External monitoring systems

External systems remain authoritative for their own native data and lifecycle unless Olympus explicitly assumes management responsibility through a defined platform contract.

### Availability and Reliability

The Command Center is an operator interface and must remain logically separate from the runtime systems it manages.

Therefore:

- Failure of the Command Center should not directly stop running workloads.
- Control Plane availability is a separate concern from Command Center availability.
- Operational actions should fail safely when required Control Plane services are unavailable.
- The Command Center should clearly distinguish current state from stale or unavailable information.
- Long-running operations should expose operation status rather than depend on a persistent UI session.
- The Command Center should support safe retry behavior for idempotent operations.

### Security Principles

The Command Center should follow:

- Least privilege
- Strong authentication
- Explicit authorization
- Sensitive-data minimization
- No direct credential exposure
- No uncontrolled infrastructure access
- Auditable operations
- Human approval for high-risk actions
- Secure session management
- Secure API communication
- AI actions subject to the same governance controls as other clients

The Command Center should never expose secrets simply because a connected platform resource contains them.

### Audit and Traceability

Operations initiated through the Command Center should be traceable.

An operation should be associated where applicable with:

- Identity of the requester
- Team
- Resource
- Requested operation
- Request timestamp
- Policy decision
- Approval decision
- Execution system
- Operation result
- Correlation ID
- Related Git commit or Pull Request
- Resulting resource state

This allows operators to answer:

Who requested the change?

What was requested?

Why was it allowed?

Which systems executed it?

What happened afterward?

### Cost Visibility

The Command Center should provide cost information where supported by the underlying platform.

Examples include:

- Resource cost
- Environment cost
- Application cost
- Cluster cost
- Per-PR environment cost
- Cost trends
- Cost allocation by owner/team

Cost information should be treated as an operational signal and should not be assumed to be perfectly real-time.

### Compliance and Governance Visibility

The Command Center should provide visibility into applicable governance controls.

Examples include:

- Policy violations
- Security posture
- Compliance control status
- Required resource metadata
- Encryption status
- Backup status
- Access configuration
- Audit history

The Command Center presents governance information; enforcement belongs to the appropriate Olympus policy and control mechanisms.

### Failure and Degraded-State Experience

The Command Center must clearly distinguish between:

- Healthy
- Degraded
- Provisioning
- Reconciling
- Failed
- Blocked by policy
- Awaiting approval
- Unavailable
- Unknown
- Stale

Users should be able to determine whether a problem originates from:

- Command Center
- Control Plane
- Policy engine
- Execution system
- Cloud provider
- Kubernetes
- External dependency

### Future Considerations

The Command Center may eventually support:

- Multi-tenant views
- Advanced topology visualization
- Cost forecasting
- Compliance dashboards
- Policy remediation workflows
- Change impact analysis
- AI-assisted incident investigation
- AI-generated remediation proposals
- Controlled autonomous remediation
- Additional cloud providers
- Additional external-system integrations
- Platform health based on defined operational signals

Future capabilities must continue to consume Olympus platform contracts rather than introduce independent control paths.

### Related Work Items
- OLY-001.2: Define Olympus Control Plane Architecture
- OLY-001.3: Define Olympus Command Center Architecture
- OLY-001.4: Define Olympus Resource Model
- OLY-001.5: Define Infrastructure-as-Data
- OLY-001.7: Define Tool Responsibilities
- OLY-001.8: Define Security & Trust Boundaries
- OLY-001.9: Define Networking Boundaries
- OLY-001.10: Define AI / MCP
- OLY-001.12: Define Ephemeral / Per-PR Environment Architecture
