---
name: react-frontend
description: Implementation conventions for the ASDLC AMS frontend - React 19, TypeScript, Vite, TanStack Router or React Router, Redux Toolkit + RTK Query, Zod, React Hook Form, Tailwind + shadcn/Radix or Fluent UI, Vitest, MSW. Use when writing, fixing or reviewing frontend code.
---

# React frontend

## Detect before you write
Read `package.json`, `vite.config.ts`, `tsconfig.json` and the `src/` layout. Use whatever the repo already chose: **TanStack Router vs React Router**, **shadcn vs Fluent UI**. Never add a second router, state library or component library.

## TypeScript
- `strict: true`; no `any` (use `unknown` + narrowing / Zod). No non-null `!` to silence errors.
- Types for API data come from the **generated RTK Query client** (see `openapi-contract`); do not redeclare them.

## React 19
- Function components only. `ref` is a regular prop - no new `forwardRef`.
- Server state lives in **RTK Query**, not `useEffect` + `fetch`. Do not copy query data into Redux slices or local state.
- `useActionState` / `useOptimistic` are fine for local form actions, but data mutations still go through RTK Query mutations.
- Keys must be stable IDs, never array indexes for dynamic lists.

## State
- **RTK Query** for all server data; `providesTags`/`invalidatesTags` per entity, added via `enhanceEndpoints` (never edit generated files).
- **Redux Toolkit slices** only for real client state (UI prefs, wizard progress). Use typed hooks `useAppDispatch` / `useAppSelector`.
- URL state (filters, paging, selected tab) lives in route search params - TanStack Router: `validateSearch` with a Zod schema; React Router: `useSearchParams`.

## Forms
- **React Hook Form + `zodResolver`**; schema in `*.schema.ts`, form type via `z.infer`.
- Map server `ValidationProblemDetails.errors` onto fields with `setError`.
- Use the component library's form primitives (shadcn `Form`/`FormField`, Fluent `Field`) so labels and errors are accessible.

## Auth (Entra ID)
- Use the repo's existing MSAL setup (`@azure/msal-react` / `@azure/msal-browser`); tokens are attached in the RTK Query `baseQuery` `prepareHeaders`. Never store tokens in localStorage yourself or log them.
- Hide UI the user cannot use, but never rely on it - the API enforces policies.

## Styling
- Tailwind utilities + `cn()` helper (shadcn) **or** Fluent tokens / `makeStyles` (Fluent). No inline style objects for theming, no hard-coded colours.

## Testing (Vitest + RTL + MSW)
- Test behaviour through the DOM: `screen.getByRole`, `getByLabelText`; `userEvent` over `fireEvent`.
- Mock HTTP with **MSW v2** (`http.get(...)`, `HttpResponse.json(...)`), typed with the generated API types - do not mock RTK Query hooks.
- Render with a test helper that provides store + router + theme provider.
- Cover: loading, error (ProblemDetails), empty, success, and validation messages.
- Run before hand-off: `npm run lint`, `npm run typecheck` (or `tsc --noEmit`), `npx vitest run`.

## Done means
Type-check clean, lint clean, Vitest green, no console errors/warnings in tests, generated client up to date.
