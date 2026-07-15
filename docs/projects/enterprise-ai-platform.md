# Enterprise AI Platform — AtomMind 2.0

`2024 · TVEL Rosatom JSC · AI Architect`

## Challenge

The nuclear-fuel manufacturer needed a unified platform to manage the full
life cycle of ML solutions — from model creation to production operation —
usable by production engineers, not only data scientists, and operating in
a regulated, security-constrained environment.

## My role

End-to-end technical architecture:

- Requirements for the ML life-cycle management system.
- Platform technical architecture and implementation options.
- Design of the MLOps stack.
- Integration of the MLOps subsystem into the overall AtomMind 2.0
  platform.
- Technical implementation design of the MLOps subsystem.

## Architecture highlights

!!! note "Two-level AI-assistant design"
    One assistant tier for production engineers on the shop floor, another
    for model creation and management — different UX, shared platform core.

- MLOps subsystem covering training, validation, deployment and monitoring.
- Designed for national security-compliance requirements (FSTEC) —
  air-gapped operation, controlled dependencies, auditable pipelines.

## Stack

`Kubernetes` · `KubeFlow` · `KServe` · `MLFlow` · `Apache Airflow` ·
`LangChain` · `LangGraph`
