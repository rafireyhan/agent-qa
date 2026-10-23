# Sena — portable QA agent

An end-to-end QA workflow for Claude Code, Codex, and Google Antigravity. Sena reads existing requirements and project context, maps an unfamiliar staging app when needed, plans and runs tests, reports bugs, retests fixes, and builds Playwright automation. Requested local artifacts are saved as the work proceeds; choices that affect results are asked once.

---

## Use on each platform

Open this folder as the workspace for native rule and skill discovery. The canonical skill bodies are in `skills/`; `AGENTS.md` routes QA requests and `references/qa-standards.md` holds detailed templates. Platform entry points are small adapters, so the workflow stays the same:

| Platform | Native entry points | Invocation |
|---|---|---|
| Claude Code | `CLAUDE.md`, `.claude-plugin/plugin.json`, `skills/`, `agents/` | Say "Sena, ..." or use `/qa-agent:qa-agent` after plugin install |
| Codex | `AGENTS.md`, `.agents/skills/`, `.codex-plugin/plugin.json` | Say "Sena, ..." or invoke `$qa-agent` when available |
| Antigravity | `.agents/rules/sena.md`, `.agents/skills/`, `.agents/workflows/sena.md` | Say "Sena, ..." or use `/sena` |

The native rules work when this repository is the workspace. For another application, tell Sena its project folder once; QA artifacts go there. A platform plugin install can also make Sena available inside other workspaces while keeping this package's canonical skills and standards together.

The four files in `agents/` define specialist roles. They are not automatically running processes; Sena uses them when a host supports and authorizes delegation, and otherwise carries out the same QA task directly.

### Claude Code plugin install

Run inside Claude Code:

```
/plugin marketplace add rafireyhan/agent-qa
/plugin install qa-agent@qa-agent
/reload-plugins
```

Claude Code exposes plugin skills with the `qa-agent:` namespace: use `/qa-agent:qa-agent` for the full workflow or, for example, `/qa-agent:qa-agent-retest` for one phase. The shorter names in the workflow table below identify the canonical skills; invoke them through the entry point of your platform.

---

## How to use

Two entry points, depending on what you need:

**1. Run the full pipeline** — from requirements or app discovery to automation, with a handoff while developers fix bugs:

```
/qa-agent https://staging.example.com
```

Or enter at any single phase:

| Command | Phase | Produces |
|---------|-------|----------|
| `/qa-agent-app-discovery [url]` | 0. Discover | `{{project_name}}_Context.md` |
| `/qa-agent-prd [url]` | 1. Understand | `PRD.md`, `User_Stories.md` |
| `/qa-agent-documentation` | 2. Plan | `Test_Plan.md`, `Test_Cases.md` |
| `/qa-agent-manual-testing` | 3. Test | Updated `Test_Cases.md` (pass/fail) + `List_Feedback.md` |
| `/qa-smoke-test [script] [url]` | 3b. Smoke Test | `List_Feedback.md` |
| `/qa-agent-retest` | 4. Retest | `Smoke_Test_Report.md` + updated files |
| `/qa-agent-automation` | 5. Automate | Playwright scripts + HTML report |

```
/qa-agent-app-discovery  →  /qa-agent-prd  →  /qa-agent-documentation  →  /qa-agent-manual-testing
                                                      │
                                            *** dev fixes bugs ***
                                                      │
                          /qa-agent-retest  →  /qa-agent-automation
```

**2. Explore for bugs the test cases miss** — run any time, independent of the pipeline:

```
/qa-exploratory-testing https://staging.example.com
```

Structured exploratory testing against a charter — finds edge cases, broken validation, access-control holes, and UX problems that scripted cases don't cover. Details below.

### What every agent does the same way

1. **Read** — checks existing requirements, context, and `qa-docs/*`; asks once for missing access or decisions.
2. **Execute** — runs the task with the appropriate skills (`agent-browser`, `find-bugs`, Playwright, etc.).
3. **Report** — records actual evidence and coverage, with List Feedback statuses and severity consistent across phases.
4. **Save** — writes requested local artifacts; asks before replacing an existing file or taking an unrequested external or destructive action.

### Where output goes

```
qa-docs/
├── PRD.md                  User_Stories.md        Test_Plan.md
├── Test_Cases.md           List_Feedback.md       Smoke_Test_Report.md
tests/playwright/           # config, Page Object Models, spec files
playwright-report/          # index.html automation report
```

---

## The agents

| Agent | Command | Skills used | Output |
|-------|---------|-------------|--------|
| App Discovery | `/qa-agent-app-discovery` | `agent-browser` | `{{project_name}}_Context.md` |
| PRD | `/qa-agent-prd` | `agent-browser` | `PRD.md`, `User_Stories.md` |
| Documentation | `/qa-agent-documentation` | `qa-test-planner`, `test-case-writing` | `Test_Plan.md`, `Test_Cases.md` |
| Manual Testing | `/qa-agent-manual-testing` | `webapp-testing`, `find-bugs`, `agent-browser` | Updated `Test_Cases.md`, `List_Feedback.md` |
| Smoke Testing | `/qa-smoke-test` | `agent-browser`, `webapp-testing`, `find-bugs` | `List_Feedback.md` |
| Retest | `/qa-agent-retest` | `smoke-test`, `ai-regression-testing`, `agent-browser` | `Smoke_Test_Report.md` |
| Automation | `/qa-agent-automation` | `playwright-*` skills | `tests/playwright/`, HTML report |
| Exploratory | `/qa-exploratory-testing` | `exploratory-testing`, `agent-browser`, `find-bugs` | `List_Feedback.md` |

---

## App Discovery agent

`/qa-agent-app-discovery` is the **"learn the app first"** step when application documentation is unavailable. Sena asks for existing documentation first, then maps an undocumented staging app via an available browser tool. It produces a structured `{{project_name}}_Context.md` that records what was observed for later planning.

- **Discovery, not testing.** It maps what exists — roles, navigation tree, features, states — it does not assert pass/fail or file bugs.
- **Every role, every menu.** Walks the full navigation for each role so nothing is missed before test planning starts.
- **Uses the project folder** for the context file and asks about the path only when it is ambiguous or would overwrite a file.

Use it when onboarding a new app; skip it and go straight to `/qa-agent-prd` when the app is already understood.

---

## Exploratory Testing agent

`/qa-exploratory-testing` is charter-driven, not random clicking. It explores freely but always against a mission, prioritized by risk. Core operating rules:

- **Staging only.** Refuses anything that looks like production. Never enters real PII/payment, never does destructive actions without asking.
- **Reads context first**, asks only for what's missing (URL, auth, scope, priority) in one message. Default time box: **45 minutes**.
- **Asks template + language** before writing anything (see [templates](#list-feedback-templates)).
- **Reports as it goes** — surfaces each confirmed bug live with repro steps, expected vs actual, evidence, and a reproducibility header (env, build, browser/viewport, account used).
- **Keeps a coverage log** (Touched / Skipped) so the end summary states exactly what was and wasn't tested.

Two checks it always runs where applicable:

- **Duplicate-data validation** — for every CRUD feature, tries to create/update with an already-existing unique value (email, SKU, name…) and confirms it's rejected, on **both** the UI form and the API request.
- **Broken access control** — where roles or record IDs exist, tries IDOR (another user's record ID) and privilege escalation (admin routes as a lower role). Any success is Critical.

### 22-point coverage checklist

Read at the start of every session; relevant sections applied per feature.

| # | Section | # | Section |
|---|---------|---|---------|
| 1 | Input Validation | 12 | API responses |
| 2 | Duplicate Data | 13 | UI (alignment, overflow, loading) |
| 3 | Boundary Value | 14 | Responsive (desktop/tablet/mobile) |
| 4 | Business Rules | 15 | Session (refresh, expiry, tabs) |
| 5 | CRUD | 16 | Role & Permission |
| 6 | Search | 17 | User Behavior (double/spam click, drag) |
| 7 | Filter | 18 | Error Handling |
| 8 | Sorting | 19 | Security (HTML/script, SQL, traversal) |
| 9 | Pagination | 20 | Data Integrity (UI→API→DB→export) |
| 10 | File Upload | 21 | Workflow (status transitions) |
| 11 | Network | 22 | UX Observation |

**Speed rules:** scan before acting · batch bad inputs into one submit · one pass per screen · ~2-min time-box per section · screenshot only bugs.

---

## List Feedback templates

Any agent that produces `List_Feedback.md` reuses the existing format. If there is no report or explicit choice, Sena asks once which format to use. Columns are never mixed.

**Standard Format (English):** `Bug ID` · `TC ID` · `Bug Title` · `Severity` · `Module` · `Steps to Reproduce` · `Expected` · `Actual` · `Status` · `Dev Notes`

**Kabayan Format (Indonesian):** professional Indonesian bug report, 10 columns A–J:

| Col | Header | Notes |
|-----|--------|-------|
| A | Defect ID | `<AppInitials>-01`, e.g. `LA-01` |
| B | Date | `DD-MMM-YYYY` |
| C | Crede | Role and account alias only; full credentials stay in private project context |
| D | Fitur | Nav path, e.g. `Menu - Submenu - Action` |
| E | Bug Description | Indonesian — observed, expected, steps + optional `Baiknya…` UX note |
| F | Priority | Low / Medium / High / Critical |
| G | Status | Full lifecycle (below) |
| H | QA | Always `Cahya` |
| I | Attachment / Comment From QA | Screenshots + technical notes |
| J | Attachment / Comment From Dev | Blank — filled by developer |

**Status lifecycle:** Open → On Progress → Hold → Ready to Test → Failed / Done → Ready for Deploy

---

## Test case templates (Traceability Matrix)

The Documentation agent can build a Google Sheets Traceability Matrix in **three** templates. Full build procedure, cell-color specs, house-style, and known pitfalls live in [QA standards](references/qa-standards.md) — read the relevant template section before building or fixing a matrix.

| Template | Roles | Formulas | Negatives / test data | Module sheet naming |
|----------|-------|----------|-----------------------|---------------------|
| **CVKC** (`Template Tracebility Matrix CVKC`) | Developer / Tester / CLIENT | COUNTIF summary + per-TC roll-up | positive-only, no test data | `Rxx - Module` / `Kxx - Module` |
| **CVKC V2** | Tester + Client (4 cols each) | roll-up mechanics, 13-col A–M | negatives + `Actual Result` col | codes in col A |
| **TM Kabayan ver Lengkap** | Tester + Client | none (plain text status) | negatives + inline test data | `MO_00x - Module` |

Common to all: statuses default to untested, dates stored as serial numbers with a `DD-MMM-YYYY` display format, and every build is done by `copyPaste`-ing a correctly-formatted block then overwriting text — never editing structure by hand.

---

## Tools

Sena uses the browser, shell, and spreadsheet capabilities available in the host platform. The following optional skills can accelerate specific work; Sena follows the canonical workflow directly when one is unavailable:

`agent-browser` · `exploratory-testing` · `webapp-testing` · `find-bugs` · `qa-test-planner` · `test-case-writing` · `smoke-test` · `ai-regression-testing` · `playwright-cli` · `playwright-best-practices` · `playwright-generate-test` · `playwright-explore-website` · `playwright-automation-fill-in-form`

**Humanizer is bundled.** Sena and all four QA specialists use it for the prose in test cases and bug reports. It improves readability in the chosen language while preserving requirements, steps, test data, observed results, IDs, severity, status, evidence, and the selected template. The bundled [humanizer skill](skills/humanizer/SKILL.md) is upstream v3.1.0 from [blader/humanizer at commit `8c307f3`](https://github.com/blader/humanizer/tree/8c307f367c91bd974002f10f2be2965d642c13df), under its included [MIT license](skills/humanizer/LICENSE). Claude Code also exposes it as `/qa-agent:humanizer`.

---

## Changelog

### 1.5.0

- Bundled humanizer for Sena and all four QA specialists when writing test cases and bug reports, with QA facts and templates protected.

### 1.4.0

- Shared QA rules and native discovery adapters for Claude Code, Codex, and Antigravity.
- Evidence-based execution columns, consistent List Feedback statuses and severity, safer credential placement, and fewer redundant approval pauses.

### 1.3.1

- **New skill & command: `/qa-smoke-test`** — Smoke testing driven by an existing UAT Script / Traceability Matrix (Google Sheets CVKC or local `Test_Cases.md`). Includes preflight parameter checks, ground-truth UI verification, and dual-template List Feedback authoring.
- **Universal QA Pod Orchestration (Sena)**: Integrated `/qa-smoke-test` into `qa-manual` agent responsibilities and Sena dispatch protocols.

### 1.2.3

- **CVKC V2 gains Evidence columns** — the CVKC Versi 2 template now includes per-shot Evidence columns (I / M, merged per shot) documented in the Documentation agent.

### 1.2.2

- **New template: CVKC Versi 2** — a 13-column (A–M) variant of the CVKC matrix with Tester + Client only, an `Actual Result` column, and negative cases, alongside the original CVKC. See [test case templates](#test-case-templates-traceability-matrix).

### 1.2.1

- **Exposed `/exploratory-testing`** — the standalone exploratory-testing skill/command (freely explore a staging app → session report) is now declared in `plugin.yaml`, so it installs as a real command alongside the charter-driven `/qa-exploratory-testing`.

### 1.2.0

- **New agent: `/qa-agent-app-discovery`** — a discovery phase (Phase 0) that runs before `/qa-agent-prd`. Explores a new staging app end-to-end across every role and menu and writes `{{project_name}}_Context.md` as the baseline the PRD and test planning build on. Registered in `plugin.yaml` (`provides_commands` + `provides_skills`) and documented above.

If you already have `qa-agent` installed, pick up this release with:

```
/plugin marketplace update qa-agent
/plugin update qa-agent@qa-agent
/reload-plugins
```

### 1.1.0

- Added `/qa-exploratory-testing` — charter-driven exploratory testing agent.

---

## License

MIT
