# OCR Platform for Technical Documents

`2024 · DM-Solutions · ML Team Lead`

## Challenge

Automated recognition of technical text in engineering documentation —
domain-specific notation, tables and drawings that general-purpose OCR
handles poorly.

## My role

- Led development and automation of ML pipelines for technical text
  recognition.
- Built pipelines for model quality assurance — recognition quality is
  measured continuously, not assumed.
- Model testing and preparation for production deployment.

## Architecture highlights

- Automated end-to-end pipeline: ingestion → preprocessing → recognition →
  quality scoring.
- QA pipeline as a first-class component: every model version is validated
  against a reference corpus before promotion.

## Stack

`PyTorch` · `Transformers` · `OCR` · `Apache Airflow` · `Docker` · `MLFlow`
