---
name: code-review
description: Checklist for reviewing ASDLC AMS code changes on the React 19 + TypeScript + RTK Query frontend and ASP.NET Core + SQL Server + Entra ID backend - correctness, security, contract, tests, operability. Use when a branch arrives with CODE-READY.
---

# Code review checklist

Standards to review against: shared skills `react-frontend`, `aspnetcore-backend`, `dotnet-testing`, `openapi-contract`, `test-strategy`, and the approved design `ams/designs/<ID>.md`.

## Process
1. `git fetch` and check out `<branch>` at the given SHA. Confirm it is based on current `main`.
2. Read `git diff --stat main...HEAD`, then the full diff file by file.
3. Run: frontend `npm ci`, lint, type-check, `npx vitest run`; backend `dotnet build`, `dotnet test`. Record results (a red build = BLOCKER).
4. Write `ams/reviews/<ID>-code.md`.

## Correctness
- [ ] Implements the approved design and every acceptance criterion; nothing extra.
- [ ] INC/PRB: root cause actually addressed (not just the symptom); regression test reproduces the original defect.
- [ ] Edge cases: nulls, empty lists, paging bounds, concurrency (EF concurrency tokens / rowversion where updates can race), time zones (`DateTimeOffset`).
- [ ] Async/await correct; `CancellationToken` passed through; no `.Result`/`.Wait()`.

## Security (BLOCKER if violated)
- [ ] Every new/changed endpoint has the designed `[Authorize(Policy=...)]`; no stray `[AllowAnonymous]`.
- [ ] Resource-level checks enforced server-side, not only hidden in the UI.
- [ ] SQL: parameterized only (Dapper) / no `FromSqlRaw` with interpolated input (EF).
- [ ] No secrets, tokens, PII or VINs in code, config, logs or test data. Tokens not stored in localStorage by app code.
- [ ] Input validated server-side with FluentValidation; ProblemDetails do not leak internals.
- [ ] Frontend: no `dangerouslySetInnerHTML` with untrusted data; external links safe.

## Contract
- [ ] OpenAPI regenerated; RTK Query client regenerated and committed; generated files not hand-edited.
- [ ] Breaking changes match what the design approved.
- [ ] Zod schemas, MSW handlers and FluentValidation rules agree with the DTOs.

## Frontend quality
- [ ] Server data via RTK Query (no `useEffect` fetches, no duplicated server state in slices); tags invalidate correctly.
- [ ] Forms use React Hook Form + zodResolver and map server validation errors.
- [ ] Accessible markup (labels, roles, focus) via the component library; no hard-coded colours.
- [ ] No `any`, no `!` to silence types, stable list keys.

## Backend quality
- [ ] Thin controllers; DTOs not entities in responses; `[ProducesResponseType]` complete.
- [ ] EF: `AsNoTracking` for reads, projection, no N+1; migration + idempotent script present. Dapper: explicit columns, versioned script.
- [ ] Health checks updated for new dependencies; structured Serilog logging; OTel spans for new external calls.

## Tests
- [ ] Right layer per `test-strategy`; Vitest + MSW for UI logic, Testcontainers integration tests for API + DB.
- [ ] Authorization tests: 401, 403 and allowed paths.
- [ ] Tests are deterministic and independent; no sleeps, no shared mutable data.

## Output
`ams/reviews/<ID>-code.md`:
```
Commit reviewed: <sha>   Verdict: CHANGES | APPROVED
Build/tests: <commands + results>
Findings: [SEVERITY] file:line - problem / Why / Suggest
Focus for human review: <authZ, SQL/migrations, data scripts, contract changes, risky logic>
```
