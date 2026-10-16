---
name: design-review
description: Checklist for reviewing ASDLC AMS designs (ams/designs/<ID>.md) covering UI/UX, accessibility, API contract, Entra ID authorization, SQL Server data changes, validation and testability. Use when a design arrives with DESIGN-READY.
---

# Design review checklist

Review against the shared skills `ui-design-system`, `solution-design` and `openapi-contract`. Each unmet item is a finding.

## Completeness
- [ ] Problem/goal, current vs proposed behaviour, impact map, open questions present.
- [ ] Solves the ticket as triaged - no unrequested scope (flag scope creep to PM).
- [ ] Acceptance criteria are numbered, testable, and cover happy path, validation errors, authorization (allowed **and** 403), and a11y for UI changes. *(Missing/untestable criteria = BLOCKER.)*

## UI / UX
- [ ] Uses the app's existing component library (shadcn **or** Fluent UI) and patterns; no new library.
- [ ] All states defined: loading, empty, error, success, no-permission.
- [ ] Forms: every field specified; Zod rules match FluentValidation rules (side-by-side table present).
- [ ] WCAG 2.1 AA addressed: labels, keyboard path, focus, contrast, not colour-only.
- [ ] URL/search-param state for filters and paging.

## API contract
- [ ] Endpoints, DTOs, status codes and ProblemDetails errors specified.
- [ ] Classified additive vs breaking; breaking changes have versioning or explicit PM approval. *(Unflagged breaking change = BLOCKER.)*
- [ ] No EF entities exposed; stable operationIds.

## Security (Entra ID)
- [ ] Every new/changed endpoint names its authorization policy and requirement. *(Missing = BLOCKER.)*
- [ ] Resource-level access rules described where users must only see their own data.
- [ ] New app roles/scopes listed as a human action.
- [ ] No PII/VIN exposure in URLs, logs or responses beyond need.

## Data (SQL Server)
- [ ] Schema change sketched with indexes; follows the repo's EF Core **or** Dapper approach.
- [ ] Backward-compatible migration (expand -> migrate -> contract) or downtime called out.
- [ ] Large-table locking / data-volume risk assessed; data-fix has dry-run + rollback.

## Operability
- [ ] Logging, telemetry and health-check impact stated for new dependencies or flows.

## Output
`ams/reviews/<ID>-design.md`: summary verdict, findings in `[SEVERITY] section - problem / why / suggest` form, then a short "Looks good" list so the designer knows what not to change.
