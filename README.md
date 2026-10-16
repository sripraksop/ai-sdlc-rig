# ASDLC AMS rig (OpenRig)

Five-seat Application Maintenance & Support team with a design review, code review and a mandatory **human code review** gate before QA. The human talks **only to the PM**.

```
                     HUMAN (operator)
                          │  only channel
                          ▼
                 mgmt-pm@asdlc  (Project Manager, lead)
                          │ delegates_to ↓   escalates_to ↑
   ┌──────────────┬───────┴──────┬──────────────┐
designer ──▶ reviewer (Codex) ◀── dev           qa
                          (delivery pod)
```

## Ticket flow
```
PM triage
  └─▶ Designer ── DESIGN-READY ──▶ Reviewer ── DESIGN-APPROVED ──▶ Dev        (design step only if needed)
                  ◀─ DESIGN-CHANGES ─┘
Dev ── CODE-READY ──▶ Reviewer ── CODE-APPROVED ──▶ PM ──▶ HUMAN code review
        ◀─ CODE-CHANGES ─┘                               │
                                    APPROVE ─────────────┤──▶ PM ── HUMAN-APPROVED (sha) ──▶ QA
                                    CHANGES ─▶ PM ── HUMAN-CHANGES ──▶ Dev (back through review)
QA ── QA-PASS ──▶ PM ──▶ release-readiness ──▶ Ready for Release
   ── QA-FAIL ──▶ Dev (fix goes back through code review + human review)
```

| Seat | Runtime | Role |
|---|---|---|
| `mgmt-pm@asdlc` | claude-code | Sole human contact; intake, triage, SLA, delegation, runs the human code-review gate |
| `delivery-designer@asdlc` | claude-code | UI/UX + solution design, impact analysis |
| `delivery-reviewer@asdlc` | **codex** | Reviews designs and code; approves to dev / PM |
| `delivery-dev@asdlc` | claude-code | Fixes, enhancements, root cause |
| `delivery-qa@asdlc` | claude-code | Playwright test design + execution, regression, sign-off |

## Layout
```
rig.yaml                          RigSpec (pods, members, edges)
skills/                           shared skills (discovered at launch)
  ams-ticket-workflow/            lifecycle, statuses, human-in-the-loop rules, SLA, hand-off format
  openapi-contract/               OpenAPI -> RTK Query codegen contract rules
  test-strategy/                  test pyramid: Vitest+MSW, xUnit+Testcontainers, Playwright
  ui-design-system/               shadcn/Fluent UI, RHF+Zod forms, WCAG 2.1 AA
  solution-design/                API/authZ/data/observability design
  react-frontend/                 React 19, RTK Query, router, Vitest+MSW conventions
  aspnetcore-backend/             ASP.NET Core, Entra ID, EF Core/Dapper, Serilog, OTel
  dotnet-testing/                 WebApplicationFactory + Testcontainers
agents/<pm|designer|reviewer|dev|qa>/
  agent.yaml                      AgentSpec
  guidance/role.md                role instructions (merged into CLAUDE.md / AGENTS.md at boot)
  skills/<id>/SKILL.md            role-only skills (declared in agent.yaml)
```

## Skills per seat
| Seat | Shared skills | Role-only skills |
|---|---|---|
| PM | ams-ticket-workflow | release-readiness |
| Designer | ams-ticket-workflow, openapi-contract, ui-design-system, solution-design | - |
| Reviewer | ams-ticket-workflow, openapi-contract, test-strategy, ui-design-system, solution-design, react-frontend, aspnetcore-backend, dotnet-testing | design-review, code-review |
| Dev | ams-ticket-workflow, openapi-contract, test-strategy, react-frontend, aspnetcore-backend, dotnet-testing | - |
| QA | ams-ticket-workflow, openapi-contract, test-strategy | playwright-e2e |

## Run (macOS / Linux / WSL - needs tmux)
1. Put this folder in (or next to) the application repo. Member `cwd: "."` is the folder containing `rig.yaml`; point it at the repo if different.
2. Prerequisites: Claude Code **and Codex** CLIs signed in (`rig setup --providers claude,codex` or per `rig setup --help`), Node.js (`npx playwright install --with-deps`), .NET 8/9 SDK, Docker (Testcontainers), per-role Entra ID test accounts as `E2E_<ROLE>_USER` / `E2E_<ROLE>_PASSWORD` env vars.
3. Validate:
   ```
   for a in pm designer reviewer dev qa; do rig agent validate ./agents/$a; done
   rig spec validate rig.yaml
   rig spec preflight rig.yaml
   ```
4. Launch:
   ```
   rig up asdlc --cwd . --plan
   rig up asdlc --cwd .
   rig ps --nodes --rig asdlc
   ```
5. Work **only through the PM** - attach to its session (via `rig tui`) or send to it:
   ```
   rig send mgmt-pm@asdlc 'New INC: dealer portal order search times out since this morning. Triage and assign.'
   ```
   When the PM shows a HUMAN CODE REVIEW package, reply in the PM session with `APPROVE <ID>` or `CHANGES <ID>: <comments>`.
6. Optional - stop accidental typing into non-PM seats:
   ```
   rig seat set-typing-guard delivery-designer@asdlc --enabled true --reason "Human talks to PM only"
   rig seat set-typing-guard delivery-reviewer@asdlc --enabled true --reason "Human talks to PM only"
   rig seat set-typing-guard delivery-dev@asdlc --enabled true --reason "Human talks to PM only"
   rig seat set-typing-guard delivery-qa@asdlc --enabled true --reason "Human talks to PM only"
   ```
7. Stop and resume later:
   ```
   rig down --snapshot
   rig up asdlc
   ```

## Customise
- **SLA matrix, statuses, review-round limit** - `skills/ams-ticket-workflow/SKILL.md`.
- **Review strictness** - `agents/reviewer/skills/{design-review,code-review}/SKILL.md`.
- **Applications in scope, environments** - add a section to each `guidance/role.md` (or a shared guidance file).
