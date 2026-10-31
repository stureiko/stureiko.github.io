# Saina — Airline Revenue Management

`2025 — present · Saina · ML Team Lead / Lead ML Engineer`

## Challenge

Build a revenue-management system that turns booking history and operational data into demand forecasts and seat-allocation recommendations. The engineering challenge extends beyond fitting models: fare-class ordering, departure horizons, uncertainty and physical cabin capacity must agree across data processing, algorithms and service integration.

## My role

I **built the Saina airline revenue management system**, combining team leadership with hands-on ML engineering in a small team. My work spanned model development, data processing, optimisation, training and inference workflows, validation and operational diagnostics.

- Translated airline revenue-management requirements into forecasting and optimisation workflows.
- Developed statistical and gradient-boosted demand forecasting methods and integrated them with seat-allocation calculations.
- Developed model lifecycle and delivery workflows, including validation, versioning, promotion checks and service integration.
- Worked on RL booking-limit policies, simulation and shadow evaluation while separating integration readiness from policy approval.

## System workflow

| Stage | Responsibility |
|---|---|
| Data preparation | Align booking histories, fare classes, departure dates, forecast cut-offs and cabin capacity |
| Demand forecasting | Moving-average and exponential-smoothing baselines alongside LightGBM and CatBoost models |
| Seat optimisation | EMSR-a/b protection levels and nested booking limits based on forecasts, uncertainty, fares and capacity |
| ML lifecycle | Time-based validation, model versioning, MLflow-backed management and champion/challenger checks |
| Delivery | Python/FastAPI services, database-backed batch computations and scheduled Kubernetes workloads |
| Operational validation | Forecast diagnostics, pipeline completeness checks and domain invariants |

## Engineering decisions

### Keep forecast and allocation semantics explicit

A demand estimate, a capacity-constrained forecast and a nested booking limit represent different quantities. I worked on keeping these meanings consistent across data transformations, optimisation outputs, diagnostics and downstream consumers.

The implementation addresses cumulative versus disjoint demand, fare ordering, uncertainty estimates and cabin-level constraints. These details determine whether a mathematically plausible result is operationally meaningful.

### Evaluate candidates against comparable evidence

Training and promotion workflows use time-based validation and baseline comparisons. Champion/challenger checks include forecast-error measures and flight-level diagnostics, supporting decisions that reflect the downstream use of a forecast rather than a single aggregate score.

### Treat delivery and model quality as separate gates

Docker packaging, database integration and Kubernetes jobs establish a delivery path. They do not establish that a model improves revenue. Release checks and domain invariants complement backtesting and model evaluation.

## Reinforcement learning: simulation and shadow evaluation

The project includes a custom booking-limit simulation environment, PPO-based policies, batch recommendation generation and shadow-quality checks. Nested actions respect cabin capacity.

**Shadow recommendations are evaluated alongside existing airline decisions; they do not establish causal revenue uplift.** The reviewed Alrosa model bundle was explicitly a technical validation build, not a policy approved for live seat-allocation decisions. Online business impact requires a separate controlled evaluation.

The workspace also contains customer-specific configurations and delivery paths for Azimut, Nordstar and Alrosa. Their presence is not a claim that every environment has the same maturity or that I owned all components of the wider platform.

## Selected technologies

Python, SQL, Pandas, NumPy, LightGBM, CatBoost, PyTorch, Stable-Baselines3, MLflow, FastAPI, PostgreSQL, Docker and Kubernetes.

## What this project demonstrates

Hands-on ML engineering and team leadership across a complete decision-support workflow: data semantics, forecasting, constrained optimisation, model lifecycle and operational integration.

[Related RL work](reinforcement-learning.md) · [Professional experience](../experience.md)
