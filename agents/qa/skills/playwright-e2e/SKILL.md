---
name: playwright-e2e
description: Playwright (TypeScript) automation standards for ASDLC AMS - E2E UI tests, API tests with APIRequestContext, Microsoft Entra ID login via setup project + storageState, accessibility with axe, traces and reports. Use when writing, running, debugging or reviewing Playwright tests.
---

# Playwright automation

## Project layout (match the repo if one exists)
```
e2e/
  playwright.config.ts
  auth.setup.ts            # signs in once per role, saves storageState
  fixtures/                # test.extend fixtures: page objects, api clients, seeded data
  pages/                   # page objects (one per screen)
  tests/ui/*.spec.ts
  tests/api/*.spec.ts
  .auth/                   # storageState files - gitignored
```

## Config essentials
- `baseURL` from env (`E2E_BASE_URL`); API base from `E2E_API_URL`.
- Projects: `setup` (auth) -> `chromium` (+ `firefox`/`webkit` for smoke if required), each with `dependencies: ['setup']` and `use.storageState`.
- `trace: 'on-first-retry'`, `screenshot: 'only-on-failure'`, `video: 'retain-on-failure'`; `retries: 2` in CI only; HTML + JUnit reporters for CI.
- `webServer` only for local runs against Vite (`npm run dev`) when targeting a local stack.

## Authentication (Microsoft Entra ID)
- `auth.setup.ts` signs in through the real Entra login page with **dedicated test accounts per role** (no MFA, conditional-access exempt - an admin task to request via the PM), then `page.context().storageState({ path: '.auth/<role>.json' })`.
- Credentials only from environment variables / CI secrets (`E2E_<ROLE>_USER`, `E2E_<ROLE>_PASSWORD`). Never commit them, never print them, keep `.auth/` in `.gitignore`.
- Role-specific tests: `test.use({ storageState: '.auth/approver.json' })`.
- API tests: get a token via the same signed-in context or a client-credentials app registration provided for testing; never use a real person's account.

## Writing tests
- Locators: `getByRole`, `getByLabel`, `getByText`, then `getByTestId` as last resort. No CSS/XPath tied to Tailwind classes or Fluent internals.
- Web-first assertions only (`await expect(locator).toBeVisible()`, `toHaveText`, `toHaveURL`); **no `waitForTimeout`**.
- Each test independent: seed its own data through the API in a fixture and clean up after; unique names (timestamp/uuid).
- Page objects expose user intentions (`ordersPage.search('ABC')`), not raw locators.
- Tag tests: `@smoke`, `@regression`, `@a11y`, and the ticket id (`@INC1234`) so they can be filtered with `--grep`.
- Use `page.route()` to mock only third-party or hard-to-reach failure cases (e.g. simulate a 500 to check the error state); core journeys run against the real API.

## API tests
- `request.newContext({ baseURL, extraHTTPHeaders: { Authorization } })` or the `request` fixture.
- Assert status, ProblemDetails shape on errors, and response fields per the OpenAPI contract (see `openapi-contract`).
- Authorization matrix: each role x endpoint -> expected 200/403; unauthenticated -> 401.

## Accessibility
`@axe-core/playwright`: `new AxeBuilder({ page }).withTags(['wcag2a','wcag2aa','wcag21aa']).analyze()` on every new/changed screen, expect zero violations (or documented, ticketed exceptions).

## Running & debugging
- `npx playwright test --grep @smoke` / `--project=chromium` / `tests/ui/orders.spec.ts`
- Debug: `npx playwright test --debug`, `--ui`; failures: `npx playwright show-trace <trace.zip>`, `npx playwright show-report`.
- A failure is only reported to dev with: steps, expected vs actual, trace/screenshot path, environment and build.

## Flakiness
Investigate with trace before retrying. Fix the locator/wait/data cause; if it is an app race condition, that is a defect for dev. Quarantine with `test.fixme` + ticket, never silently delete.
