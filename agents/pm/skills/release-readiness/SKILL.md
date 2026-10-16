---
name: release-readiness
description: Release readiness checklist for ASDLC AMS changes on the React + ASP.NET Core + SQL Server stack. Use before marking any ticket Ready for Release or when preparing a release summary for human approval.
---

# Release readiness checklist

Mark a ticket **Ready for Release** only when every applicable item is confirmed, with evidence from the dev/QA hand-offs.

## Quality gates
- [ ] Design review `DESIGN-APPROVED` (if the ticket had a design) - `ams/reviews/<ID>-design.md`.
- [ ] Code review `CODE-APPROVED` - `ams/reviews/<ID>-code.md`.
- [ ] Human code review `HUMAN-APPROVED` recorded in `ams/tickets/<ID>.md` (approver, time, SHA).
- [ ] The SHA the human approved == the SHA QA tested == the SHA being released.
- [ ] QA sent `QA-PASS` with evidence (see `test-strategy` skill): Vitest, `dotnet test`, Playwright results.
- [ ] Every acceptance criterion maps to a passing test.
- [ ] No open P1/P2 defects against the ticket.

## API & contract
- [ ] OpenAPI change classified additive vs breaking; breaking changes explicitly approved.
- [ ] Regenerated RTK Query client committed with the backend change.

## Database (SQL Server)
- [ ] Schema changes delivered as reviewed scripts (EF Core `migrations script --idempotent` or versioned SQL for Dapper).
- [ ] Backward compatible with the currently deployed app version (expand -> migrate -> contract), or downtime approved.
- [ ] Data-fix scripts have a dry-run mode and a rollback script.

## Security
- [ ] New/changed endpoints have an Entra ID authorization policy; no unintended `[AllowAnonymous]`.
- [ ] New app roles/scopes registered in Entra ID (human action - list them).
- [ ] No secrets, PII or VINs in code, logs or test data.

## Operability
- [ ] Health checks (`/health/live`, `/health/ready`) still pass; new dependencies added to readiness checks.
- [ ] Serilog/OpenTelemetry: new flows log with correlation and appear in Application Insights; no sensitive data logged.
- [ ] Config/feature flags documented per environment.

## Release summary (send to the operator)
```
RELEASE: <ticket IDs>
CHANGE: <what changes for users>
DEPLOY ORDER: <db script -> api -> ui>
CONFIG/ENTRA CHANGES: <list or none>
TEST EVIDENCE: <summary + report links>
ROLLBACK: <steps>
RISKS: <list>
```
