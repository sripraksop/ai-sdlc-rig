---
name: dotnet-testing
description: Backend testing for ASDLC AMS - unit tests and integration tests with WebApplicationFactory and Testcontainers for .NET (SQL Server), including Entra ID authorization tests. Use when adding or fixing backend tests or reproducing a backend defect.
---

# .NET testing

## Unit tests
- Use the repo's runner (xUnit is the default assumption) and assertion library already referenced.
- Test validators directly with `TestValidate()` (FluentValidation.TestHelper); test authorization handlers with a constructed `ClaimsPrincipal`.

## Integration tests (WebApplicationFactory + Testcontainers)
- Real SQL Server via **`Testcontainers.MsSql`** (`new MsSqlBuilder().Build()`), started once per test collection (`IAsyncLifetime` + collection fixture). Requires Docker on the machine/CI agent.
- Point the app at the container by overriding the connection string in `WebApplicationFactory<Program>.ConfigureWebHost`.
- Apply schema the same way production does: EF Core `Database.MigrateAsync()` or run the versioned SQL scripts (Dapper). This also tests the migrations.
- Isolate data per test (unique keys, or reset with Respawn if the repo uses it). No reliance on test order.

## Testing auth without real Entra ID
- Replace the JWT scheme in the test host with a **test authentication handler** that builds claims (roles, `scp`, oid) from a header or per-test setup.
- For each protected endpoint assert: no token -> 401, wrong role/scope -> 403, correct role -> success. The real policies must stay registered - only authentication is faked.

## What to assert
- Status code + `ProblemDetails` / `ValidationProblemDetails` shape for errors.
- Response body matches the DTO (deserialize into the DTO type).
- Database state after writes (query via a separate context/connection).

## Reproducing a defect
Write a failing integration test first, then fix. Keep it as the regression test.

## Run
`dotnet test` (filter with `--filter` when iterating). Report counts and any skipped tests in the hand-off.
