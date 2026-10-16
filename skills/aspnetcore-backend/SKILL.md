---
name: aspnetcore-backend
description: Implementation conventions for the ASDLC AMS backend - ASP.NET Core 8/9 controllers on Kestrel, Microsoft Entra ID auth with authorization policies, Dapper or EF Core on SQL Server, FluentValidation, OpenAPI, Health Checks, Serilog, OpenTelemetry + Application Insights. Use when writing, fixing or reviewing backend code.
---

# ASP.NET Core backend

## Detect before you write
Check `*.csproj` (`TargetFramework` net8.0 / net9.0, package references) and `Program.cs`. Follow what exists: **EF Core vs Dapper**, **Swashbuckle vs NSwag vs Microsoft.AspNetCore.OpenApi**, test runner. Do not introduce a competing library.

## Controllers
- `[ApiController]`, attribute routing `[Route("api/v1/[controller]")]`, thin actions: validate -> call service -> map to DTO.
- Return `ActionResult<T>` / `Results<Ok<T>, NotFound, ...>` with `[ProducesResponseType]` for every status (feeds OpenAPI - see `openapi-contract`).
- Errors: `AddProblemDetails()` + global exception handler (`IExceptionHandler`); never leak stack traces or SQL errors.
- Async all the way, pass `CancellationToken` from the action to data access.

## Authentication & authorization (Microsoft Entra ID)
- Use the existing `Microsoft.Identity.Web` setup (`AddMicrosoftIdentityWebApi`) - config from `AzureAd` section, never hard-coded tenant/client secrets.
- Protect with **named policies**: `[Authorize(Policy = "Orders.Write")]`; define policies in one place (`AddAuthorizationBuilder().AddPolicy(...)` with `RequireRole` / `RequireScope`/claims).
- Default to authenticated: a fallback policy requiring an authenticated user; `[AllowAnonymous]` only for health/OpenAPI as agreed.
- Resource-based checks via `IAuthorizationService` / `AuthorizationHandler<TRequirement, TResource>`.

## Validation (FluentValidation)
- One `AbstractValidator<TRequest>` per request DTO, registered via `AddValidatorsFromAssemblyContaining<...>()`.
- Invoke explicitly (endpoint filter / action filter / service) and return `ValidationProblemDetails`; the old `FluentValidation.AspNetCore` auto-validation package is deprecated - don't add it.
- Rules must match the frontend Zod schema.

## Data access (SQL Server)
- **EF Core**: `DbContext` scoped, `AsNoTracking()` for reads, project to DTOs with `Select`, avoid N+1 (`Include` or projection). Schema changes via migrations; produce `dotnet ef migrations script --idempotent` for review.
- **Dapper**: parameterized SQL only (never string-concatenate input), `CommandDefinition` with cancellation token, explicit column lists, versioned SQL scripts for schema changes.
- Connection strings from configuration / Key Vault; use transactions for multi-statement writes.

## Operability
- **Health checks**: `/health/live` (process only) and `/health/ready` (SQL Server + other dependencies, tagged). Add a check for any new dependency.
- **Serilog**: structured logging with message templates (`Log.Information("Order {OrderId} updated", id)`), request logging middleware, enrich with correlation/trace id. Never log tokens, PII or VINs.
- **OpenTelemetry + Application Insights**: keep the existing `UseAzureMonitor()` / OTel setup; add an `ActivitySource` span for new external calls or long operations, and metrics via `Meter` where useful.
- Kestrel limits/timeouts come from configuration; don't change them without PM approval.

## Testing
See `dotnet-testing`. Run `dotnet build -warnaserror` (if the repo does) and `dotnet test` before hand-off.

## Done means
Builds clean, tests green, OpenAPI regenerated, RTK Query client regenerated, policies applied, health/readiness unaffected.
