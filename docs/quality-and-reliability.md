# Quality and Reliability Strategy

Quality in an operational system is broader than “the page opens.” The platform must preserve permissions, workflow integrity, data relationships, and recoverability while continuing to serve day-to-day work.

## Verification layers

| Layer | What is verified | Typical evidence |
|---|---|---|
| Domain rules | Calculations, validations, and allowed state transitions | Focused unit and workflow tests |
| Persistence | Constraints, relationships, transaction boundaries, migrations | PostgreSQL integration tests and migration checks |
| API contracts | Request/response schemas, error behavior, authorization | Endpoint tests and generated contract review |
| Interface | Responsive layout, role-aware actions, form behavior | Component checks and critical-path browser testing |
| End-to-end flow | Order-to-fulfilment continuity and exception paths | Scenario-based regression suites |
| Release | Configuration, secrets presence, service readiness | Preflight, health probes, and smoke tests |
| Recovery | Database and uploaded-file restoration | Checksummed backup bundles and restore rehearsal |

## Risk-based test design

Testing effort follows business risk rather than treating every screen equally.

- **High risk:** authorization, monetary calculations, irreversible actions, workflow transitions, imports, and data repairs.
- **Medium risk:** filters, exports, reporting aggregations, and bulk operations.
- **Lower risk:** presentational changes that do not alter stored data or permissions.

For stateful workflows, tests cover both the happy path and interruption cases—for example cancellation before preparation, cancellation inside a batch, and rejection of actions after a terminal state.

## Definition of done

A change is considered ready when:

1. Acceptance rules and affected roles are explicit.
2. Backend rules, API contracts, and interface behavior agree.
3. Relevant automated tests pass and critical paths are manually reviewed.
4. Database changes are additive or have a verified recovery strategy.
5. Logs and audit records provide enough context to diagnose failure.
6. Production configuration is validated without exposing secrets.
7. Backup, health, smoke, and data-conservation checks are defined for release.

## Production correction principles

When live data requires repair:

1. Identify the exact records and dependencies with read-only diagnostics.
2. Take and verify a database and persistent-storage backup.
3. Run the repair in dry-run mode and record the proposed changes.
4. Apply the smallest transactional correction possible.
5. Compare pre/post counts and business invariants.
6. Run a global dry run again to confirm no candidates remain.
7. Preserve a concise repair report for later audit.

This approach separates urgent recovery from improvisation: speed comes from prepared controls, not from skipping them.
