# QA Agent — Claude Code Plugin

An end-to-end QA workflow for Claude Code. Point it at a staging URL and it goes from exploring the app → writing docs → executing tests → reporting bugs → generating Playwright automation. Every file is written only after you approve it.

---

## Install

Run inside Claude Code:

```
/plugin marketplace add rafireyhan/agent-qa
/plugin install qa-agent@qa-agent
/reload-plugins
```

The `/qa-agent*` commands are then available in any project. See [Prerequisites](#prerequisites) for the skills it depends on.

---

## How to use

Two entry points, depending on what you need:

**1. Run the full pipeline** — from URL to automation, with an approval gate between every phase:

```
/qa-agent https://staging.example.com
```

Or enter at any single phase:

| Command | Phase | Produces |
|---------|-------|----------|
| `/qa-agent-prd [url]` | 1. Understand | `PRD.md`, `User_Stories.md` |
| `/qa-agent-documentation` | 2. Plan | `Test_Plan.md`, `Test_Cases.md` |
| `/qa-agent-manual-testing` | 3. Test | Updated `Test_Cases.md` (pass/fail) + `List_Feedback.md` |
| `/qa-agent-retest` | 4. Retest | `Smoke_Test_Report.md` + updated files |
| `/qa-agent-automation` | 5. Automate | Playwright scripts + HTML report |

```
/qa-agent-prd  →  /qa-agent-documentation  →  /qa-agent-manual-testing
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

1. **Ask** — collects required inputs (URL, credentials, scope) before doing anything, and reads existing `qa-docs/*` so it never re-asks what's already known.
2. **Execute** — runs the task with the appropriate skills (`agent-browser`, `find-bugs`, Playwright, etc.).
3. **Approval gate** — shows a summary and waits for your review.
4. **Save** — writes files only after explicit sign-off. **No file is written without you.**

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
| PRD | `/qa-agent-prd` | `agent-browser` | `PRD.md`, `User_Stories.md` |
| Documentation | `/qa-agent-documentation` | `qa-test-planner`, `test-case-writing` | `Test_Plan.md`, `Test_Cases.md` |
| Manual Testing | `/qa-agent-manual-testing` | `webapp-testing`, `find-bugs`, `agent-browser` | Updated `Test_Cases.md`, `List_Feedback.md` |
| Retest | `/qa-agent-retest` | `smoke-test`, `ai-regression-testing`, `agent-browser` | `Smoke_Test_Report.md` |
| Automation | `/qa-agent-automation` | `playwright-*` skills | `tests/playwright/`, HTML report |
| Exploratory | `/qa-exploratory-testing` | `exploratory-testing`, `agent-browser`, `find-bugs` | `List_Feedback.md` |

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

Any agent that produces `List_Feedback.md` pauses and asks which format to use — no column mixing.

**Standard Format (English):** `Bug ID` · `TC ID` · `Bug Title` · `Severity` · `Module` · `Steps to Reproduce` · `Expected` · `Actual` · `Status` · `Dev Notes`

**Kabayan Format (Indonesian):** professional Indonesian bug report, 10 columns A–J:

| Col | Header | Notes |
|-----|--------|-------|
| A | Defect ID | `<AppInitials>-01`, e.g. `LA-01` |
| B | Date | `DD-MMM-YYYY` |
| C | Crede | Role or account used during testing |
| D | Fitur | Nav path, e.g. `Menu - Submenu - Action` |
| E | Bug Description | Indonesian — observed, expected, steps + optional `Baiknya…` UX note |
| F | Priority | Low / Medium / High / Critical |
| G | Status | Full lifecycle (below) |
| H | QA | Always `Cahya` |
| I | Attachment / Comment From QA | Screenshots + technical notes |
| J | Attachment / Comment From Dev | Blank — filled by developer |

**Status lifecycle:** Open → On Progress → Hold → Ready to Test → Failed / Done → Ready for Deploy

---

## Test case template (CVKC)

The Documentation agent builds test cases using the **"Template Tracebility Matrix CVKC"** (typo intentional — matches the original). Sheet 1 is a feedback recap; Sheet 2+ are module sheets (`Rxx - Module Name`) with `#SCxx` scenario blocks and `#TCxx` test-case blocks. Statuses default to Developer `Pending` / Tester `Need Test` / CLIENT `Pending`.

The full Google Sheets build procedure, cell-color spec, and known pitfalls (quota-silent failures, `insertDimension` color loss) live in [`CLAUDE.md`](CLAUDE.md) — read it before building or fixing a matrix.

---

## Prerequisites

These Claude Code skills must be installed:

`agent-browser` · `exploratory-testing` · `webapp-testing` · `find-bugs` · `qa-test-planner` · `test-case-writing` · `smoke-test` · `ai-regression-testing` · `playwright-cli` · `playwright-best-practices` · `playwright-generate-test` · `playwright-explore-website` · `playwright-automation-fill-in-form`

---

## License

MIT
