# AtomMind — Industrial AI Platform

`2024 — present · DM-Solutions · TVEL Rosatom JSC · Chief Architect`

## Challenge

Design a shared industrial AI platform that connects enterprise data, ML development, model delivery and AI-assisted engineering workflows under security and deployment constraints. The architecture must support multiple engineering teams while keeping responsibilities, integration contracts and delivery scope clear.

## My role

As **Chief Architect**, I owned the overall platform architecture and its main design decisions. My work covered subsystem boundaries, technology selection, integration patterns and the translation of requirements into designs and implementation plans.

- Defined the platform structure using C4 context, container and component models in Structurizr DSL.
- Recorded alternatives and trade-offs in architecture decision records, supported by sequence diagrams and integration contracts.
- Designed the relationship between the industrial platform, ML workspaces, model lifecycle services and AI-Hub.
- Reconciled the target architecture with implementation evidence and documented remaining gaps.
- Defined security requirements aligned with FSTEC requirements, including policy enforcement, controlled data access and audit integration.

## Architecture highlights

### ML development and delivery

The architecture connects JupyterHub and code-server workspaces with MLflow tracking and model management, Airflow tasks, lakeFS dataset versioning and controlled software dependencies.

For serving, I documented **FastAPI plus Helm** as a simpler Kubernetes delivery approach. The trade-off is explicit: fewer specialised operators, with inference contracts and scale-to-zero behaviour requiring separate design. The implemented packaging path includes a FastAPI wrapper, Gitea Actions builds, Harbor artefacts and Helm deployment.

### Governed access to models and tools

AI-Hub provides controlled model and MCP tool access through gateways, registry-backed AI profiles, OPA decisions, quotas, data filtering and model-integrity checks. The reviewed implementation includes plain Envoy with Go control services, a Go/PostgreSQL registry and inference Helm charts.

[Read more about the AI assistant architecture](ai-assistants.md).

### Architecture that follows implementation evidence

The model distinguishes target capabilities from existing services and development configurations. This helps make release scope and outstanding integration work visible without treating every architecture component as delivered.

| Area | Evidence and status |
|---|---|
| Platform architecture | C4 models, decision records, integration contracts and deployment designs |
| ML workspaces and delivery | Implementation evidence for workspace services, tracking, data versioning and model packaging |
| Agent orchestration | Python/Temporal workflows and a custom pipeline runtime; LangGraph is a documented design choice, absent from the reviewed runtime |
| Document processing | Full OCR/document-ingestion pipeline remains planned; separate OCR project experience is described elsewhere |
| Feature store and AutoML | Planned capabilities |
| Security | Designed controls and implemented mechanisms; this case study does not claim completed certification or uniform enforcement across all environments |

## AWS reference design

I also developed an AWS translation and Terraform reference covering EKS, networking, storage, streaming, ML services and observability. This is an **undeployed reference exercise**, separate from evidence of production AWS operation.

## Selected technologies

**Architecture:** C4, Structurizr DSL, architecture decision records.

**Platform and ML:** Kubernetes, Helm, FastAPI, MLflow, Airflow, JupyterHub, lakeFS, Gitea, Harbor.

**Governed AI:** Envoy, Go, PostgreSQL, OPA, MCP, Python, Temporal.

## Delivery perspective

This project demonstrates architecture ownership, technical trade-off analysis and implementation alignment. It is an ongoing platform programme with components at different maturity levels, rather than a claim that the entire target architecture is in production.
