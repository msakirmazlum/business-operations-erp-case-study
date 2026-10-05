# Architecture Decision Highlights

This document summarizes selected engineering decisions behind a representative business operations platform. The examples are intentionally generalized: they demonstrate reasoning and trade-offs without exposing client-specific source code, infrastructure, or business data.

## ADR-01 — Start with a modular monolith

**Context**
The product covers several connected domains—orders, preparation, shipment, customer records, activities, and reporting—but they share the same operational lifecycle and transactional data.

**Decision**
Use a modular monolith with explicit domain boundaries instead of introducing distributed services prematurely.

**Why**

- Cross-domain transactions remain consistent.
- Deployment, observability, and local development stay manageable.
- Modules can still evolve independently through service and repository boundaries.

**Trade-off**
The application requires disciplined module ownership. If independent scaling or team boundaries become dominant constraints, selected modules can later be extracted behind stable contracts.

## ADR-02 — Keep workflow rules on the server

**Context**
Operational records move through dependent states. A UI-only restriction can be bypassed by another client, a stale browser session, or a direct API request.

**Decision**
Model allowed state transitions explicitly and enforce them in the application service layer. The frontend communicates available actions; the backend remains the authority.

**Why**

- Invalid transitions are rejected consistently.
- Authorization and business rules are evaluated together.
- Regression tests can exercise the workflow without relying on browser behavior.

**Trade-off**
The same lifecycle must be represented coherently across domain code, API contracts, and the interface. Contract generation and focused workflow tests reduce drift.

## ADR-03 — Treat imports as an integration boundary

**Context**
Operational data may arrive through spreadsheets or external exports with inconsistent naming, identifiers, formats, and partial fields.

**Decision**
Use a staged import pipeline: parse, normalize, validate, reconcile, preview, then commit. Preserve source identifiers and make repeat execution safe wherever possible.

**Why**

- Bad input is isolated before it reaches core tables.
- Duplicate and conflicting records can be reviewed deliberately.
- Import results become explainable and reproducible.

**Trade-off**
The pipeline adds implementation effort, but it prevents opaque one-off scripts from becoming an uncontrolled production dependency.

## ADR-04 — Correct history without erasing accountability

**Context**
Real operations occasionally create duplicates or attach records to the wrong master entity. Simply deleting rows can break references and erase evidence.

**Decision**
Perform corrections as verified, transactional reconciliation operations. Move dependent records first, validate conservation counts, retain audit evidence, and remove or deactivate the obsolete master only after post-checks pass.

**Why**

- Related business history remains connected.
- Corrections are reviewable and reversible through backups.
- Data loss is detected before a change is accepted.

**Trade-off**
Repair tooling is more deliberate than manual database editing. That added friction is intentional for production safety.

## ADR-05 — Release with evidence, not assumption

**Context**
Application code, database migrations, configuration, and persistent files must change without losing live business records.

**Decision**
Use additive migrations, environment preflight checks, verified database and storage backups, health probes, smoke tests, and post-release conservation checks.

**Why**

- A release has objective pass/fail gates.
- Rollback inputs are verified before the maintenance window.
- Data preservation is checked independently from application health.

**Trade-off**
Deployments include more controls than a simple rebuild-and-restart command, but failures become diagnosable and recovery paths remain explicit.
