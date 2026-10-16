---
name: openapi-contract
description: API contract rules for the ASDLC AMS stack - ASP.NET Core OpenAPI document is the source of truth, RTK Query hooks are generated from it. Use when designing, changing, implementing or testing any HTTP endpoint or the frontend code that calls it.
---

# OpenAPI contract (backend -> frontend)

The **backend OpenAPI document is the single source of truth**. Frontend API code is generated, never hand-written.

```
ASP.NET Core controllers + DTOs
   -> OpenAPI JSON (Microsoft.AspNetCore.OpenApi on .NET 9, Swashbuckle or NSwag on .NET 8)
   -> @rtk-query/codegen-openapi
   -> typed RTK Query endpoints + hooks (useGetOrderQuery, useUpdateOrderMutation ...)
```

## Detect what the repo uses
- Generator: look for `AddOpenApi()` / `MapOpenApi()` (Microsoft.AspNetCore.OpenApi), `AddSwaggerGen()` (Swashbuckle) or `AddOpenApiDocument()` (NSwag) in `Program.cs`. Use the one already there; do not add a second.
- Codegen config: `openapi-config.ts` / `.cts` (or similar) in the frontend. Check `package.json` scripts for the codegen command.

## Designing an endpoint
- Resource-oriented routes: `GET /api/v1/orders/{id}`, `POST /api/v1/orders`. Version in the path.
- Every action declares its responses: `[ProducesResponseType<OrderDto>(200)]`, `[ProducesResponseType<ValidationProblemDetails>(400)]`, `404`, `401/403` as applicable.
- Errors use RFC 9457 **ProblemDetails** (`AddProblemDetails()`); validation errors use `ValidationProblemDetails` with field keys matching DTO property names (camelCase) so the frontend can map them onto React Hook Form fields.
- Stable `operationId`s (controller action names) - they become RTK Query hook names. Renaming one is a breaking change.
- Use DTOs/records, never EF entities, in the contract. Dates as ISO-8601 strings (`DateTimeOffset`), money as decimal strings or minor units - state which.
- Paging: `?page=&pageSize=` returning `{ items, totalCount, page, pageSize }`.
- Security: each operation declares the Entra ID scheme/scope; anonymous endpoints are explicit (`[AllowAnonymous]`) and rare.

## Changing an endpoint
1. Classify: **additive** (new optional field/endpoint) vs **breaking** (rename, remove, type change, new required field). Breaking changes need a new version or PM approval.
2. Update DTOs + attributes -> regenerate the OpenAPI doc -> rerun RTK Query codegen -> fix TypeScript compile errors -> update Zod schemas and MSW handlers that mirror the contract.
3. Commit the regenerated client in the same branch as the backend change.

## Frontend side
- Zod schemas validate **form input and untrusted data**; generated TS types describe the API. Keep the Zod schema for a form aligned with the request DTO and its FluentValidation rules (same required fields, lengths, formats).
- RTK Query: use `providesTags` / `invalidatesTags` per entity for cache invalidation (add via `enhanceEndpoints` - do not edit generated files).

## Testing the contract
- Backend integration tests assert status codes and ProblemDetails shape.
- Playwright API tests (QA) hit the real endpoints and assert against the documented schema.
- MSW handlers in frontend tests must return shapes that match the generated types (type the handler responses with them).
