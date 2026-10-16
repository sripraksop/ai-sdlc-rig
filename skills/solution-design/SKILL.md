---
name: solution-design
description: Solution and impact-analysis design for ASDLC AMS changes spanning React frontend, ASP.NET Core API, Entra ID authorization and SQL Server. Use when a CR or ticket changes an API, data model, permissions or integrations.
---

# Solution design (full stack)

Write the technical part of `ams/designs/<ID>.md` with these sections. Keep it short; dev will implement it.

## 1. Impact map
List each touched layer and file area:
- UI routes/components, Redux slices, RTK Query endpoints
- API controllers/actions (by route + operationId)
- Validators (FluentValidation), authorization policies
- Data: tables, views, stored procs, EF Core entities or Dapper queries
- Cross-cutting: health checks, logging/telemetry, config, other consuming systems/reports

## 2. API contract
Request/response DTOs, status codes, ProblemDetails errors - follow `openapi-contract`. State **additive vs breaking**.

## 3. Security (Microsoft Entra ID)
- Which **authorization policy** protects each endpoint and what it requires (app role, scope `scp`, or group claim). Prefer named policies (`"Orders.Write"`) over inline `[Authorize(Roles=...)]`.
- Any new Entra app roles/scopes needed -> list as a human action for the PM.
- Resource-level checks (e.g. user can only edit their dealer's orders) -> an `IAuthorizationHandler`, described here.

## 4. Data (SQL Server)
- Schema change as DDL sketch; indexes for new query patterns.
- Data access: match the repo - **EF Core** (entity + migration) or **Dapper** (parameterized SQL). Do not introduce the other.
- Migration plan must be backward compatible with the running version (expand -> migrate -> contract). Note data volume / lock risk for large tables.

## 5. Validation
Rules table: field | rule | FluentValidation (server) | Zod (client). They must agree.

## 6. Observability
New log events (Serilog structured properties, no PII), custom OpenTelemetry spans/metrics if a new external call or long-running step is added, readiness health check for any new dependency.

## 7. Acceptance criteria
Numbered, testable, covering happy path, validation errors, authorization (allowed **and** 403), and a11y for UI changes.
