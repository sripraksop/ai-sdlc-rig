---
name: test-strategy
description: ASDLC AMS test pyramid - which test type (Vitest+MSW, xUnit+Testcontainers, Playwright) covers what, and the evidence required before QA sign-off. Use when planning tests for a ticket, deciding where a test belongs, or reviewing test coverage.
---

# Test strategy

| Layer | Tool | Owner | Covers |
|---|---|---|---|
| Frontend unit / component | **Vitest** + React Testing Library + **MSW** | Dev | Components, hooks, Zod schemas, reducers, RTK Query behaviour against mocked HTTP |
| Backend unit | xUnit (or the repo's runner) | Dev | Domain logic, FluentValidation validators, authorization handlers |
| Backend integration | `WebApplicationFactory` + **Testcontainers for .NET** (SQL Server) | Dev | Controllers end-to-end in-process, real SQL Server, EF Core/Dapper queries, migrations, auth policies, ProblemDetails |
| API (black box) | **Playwright** `APIRequestContext` | QA | Deployed API: status codes, contract, authZ per role |
| End-to-end UI | **Playwright** | QA | Critical user journeys in a real browser against a real backend |
| Accessibility | Playwright + `@axe-core/playwright` | QA | WCAG 2.1 AA on changed screens |

## Rules
- Push tests **down** the pyramid: if a behaviour can be proven by a Vitest or xUnit test, do not cover it only in Playwright.
- Every defect fix ships with a test at the lowest layer that would have caught it; QA adds a Playwright regression only for user-journey defects.
- Every acceptance criterion maps to at least one test (record the mapping in `ams/tests/<ID>.md`).
- Authorization is tested negatively too: a user **without** the role/policy gets 403.
- No test depends on another test's data. Integration tests create their own data in a fresh container/database; Playwright tests seed via API or fixtures.
- Flaky tests are defects - quarantine with a ticket, do not retry-until-green.

## Evidence for QA sign-off
- Commands run and pass/fail counts for: `vitest run`, `dotnet test`, `npx playwright test` (relevant projects/tags).
- Playwright HTML report / trace for any failure that was investigated.
- Acceptance-criteria -> test mapping, residual risks, and what was **not** tested.
