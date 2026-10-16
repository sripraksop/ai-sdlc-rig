---
name: ui-design-system
description: UI/UX design rules for ASDLC AMS React apps built with Tailwind + Radix/shadcn or Fluent UI, React Hook Form + Zod forms, WCAG 2.1 AA. Use when designing or reviewing screens, forms, flows or component changes.
---

# UI design system

## Detect the component library first
Check `package.json`: `@fluentui/react-components` => **Fluent UI v9**; `@radix-ui/*` + `tailwindcss` + `components/ui/*` => **shadcn/ui**. Design with the library the app already uses; never mix both in one app.

| Need | shadcn/ui (Radix + Tailwind) | Fluent UI v9 |
|---|---|---|
| Theming | CSS variables in Tailwind config | `FluentProvider` + tokens |
| Dialog | `Dialog` / `AlertDialog` | `Dialog` |
| Data table | `Table` + TanStack Table | `DataGrid` |
| Form field | `Form`, `FormField`, `FormMessage` | `Field` (label + validationMessage) |
| Toast | `Sonner` / `Toast` | `Toaster` |

Reuse existing components; propose a new shared component only when a pattern repeats 3+ times.

## Designing a screen
For each change produce in `ams/designs/<ID>.md`:
- **Route** (TanStack Router or React Router - match the repo) and how the user gets there; URL search params for filters/paging so views are linkable.
- **States**: loading (skeleton), empty, error (ProblemDetails message), success, no-permission (403).
- **Layout**: responsive breakpoints (Tailwind `sm/md/lg` or Fluent breakpoints), what collapses on small screens.
- **Data**: which API operations it calls (by operationId - see `openapi-contract`) and when the cache refreshes.

## Forms (React Hook Form + Zod)
- Specify each field: label, type, required, constraints, default, help text. These become **one Zod schema** that matches the backend FluentValidation rules - list them side by side in the design.
- Validate on blur, re-validate on change after first error; show server `ValidationProblemDetails` errors on the matching field.
- Disable submit while submitting; confirm destructive actions with an AlertDialog/Dialog.

## Accessibility (WCAG 2.1 AA) - acceptance criteria, not nice-to-have
- Every input has a visible label; errors linked via `aria-describedby` (library form components do this - do not bypass them).
- Full keyboard path, visible focus, logical tab order; dialogs trap focus and restore it on close.
- Contrast >= 4.5:1 text, 3:1 UI; never use colour alone for status.
- Use semantic roles so QA can target them with Playwright `getByRole` / `getByLabel`.

## Hand-off
Design doc includes numbered acceptance criteria (UI + a11y) QA can test directly, plus a screenshot/mockup or ASCII wireframe.
