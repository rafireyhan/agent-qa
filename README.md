# QA Agent — Claude Code Plugin

End-to-end QA workflow with 7 specialized agents. From staging URL to Playwright automation scripts, with user approval gates at every step.

## Workflow

```
/qa-agent-prd              →  PRD.md + User_Stories.md
/qa-agent-documentation    →  Test_Plan.md + Test_Cases.md
/qa-agent-manual-testing   →  Test_Cases.md (status) + List_Feedback.md
        *** Dev fixes bugs ***
/qa-agent-retest           →  Smoke_Test_Report.md + updated files
/qa-agent-automation       →  Playwright scripts + HTML report
```

Run the full cycle with `/qa-agent`, or enter at any phase individually.

Use `/qa-exploratory-testing` at any point for structured exploratory testing — finds bugs outside the test cases.

---

## Installation

Run these commands inside Claude Code:

**Step 1 — Add the marketplace**
```
/plugin marketplace add rafireyhan/agent-qa
```

**Step 2 — Install the plugin**
```
/plugin install qa-agent@qa-agent
```

**Step 3 — Reload plugins**
```
/reload-plugins
```

The `/qa-agent` commands will be available immediately in any project.

---

## Commands

| Command | Phase | Description |
|---------|-------|-------------|
| `/qa-agent [url]` | Full cycle | Runs all phases in sequence with approval gates |
| `/qa-agent-prd [url]` | Phase 1 | Explore staging app → write `PRD.md` + `User_Stories.md` |
| `/qa-agent-documentation` | Phase 2 | Read PRD/User Stories → write `Test_Plan.md` + `Test_Cases.md` |
| `/qa-agent-manual-testing` | Phase 3 | Execute every test case → update pass/fail + write `List_Feedback.md` |
| `/qa-agent-retest` | Phase 4 | Retest failures, run smoke + regression checks |
| `/qa-agent-automation` | Phase 5 | Generate Playwright scripts + HTML report |
| `/qa-exploratory-testing [url]` | Any time | Structured exploratory testing using SBTM charters + 22-point checklist → `List_Feedback.md` |

---

## Agents & Skills

| Agent | Skills Used | Output |
|-------|-------------|--------|
| QA PRD | `agent-browser` | `qa-docs/PRD.md`, `qa-docs/User_Stories.md` |
| QA Documentation | `qa-test-planner`, `test-case-writing` | `qa-docs/Test_Plan.md`, `qa-docs/Test_Cases.md` |
| QA Manual Testing | `webapp-testing`, `find-bugs`, `agent-browser` | Updated `Test_Cases.md`, `qa-docs/List_Feedback.md` |
| QA Retest | `smoke-test`, `ai-regression-testing`, `agent-browser` | `qa-docs/Smoke_Test_Report.md`, updated files |
| QA Automation | `playwright-cli`, `playwright-best-practices`, `playwright-generate-test`, `playwright-explore-website`, `playwright-automation-fill-in-form` | `tests/playwright/`, `playwright-report/index.html` |
| QA Exploratory Testing | `exploratory-testing`, `agent-browser`, `find-bugs` | `qa-docs/List_Feedback.md` |

---

## Output Files

All documentation is written to `qa-docs/` in your project:

```
qa-docs/
├── PRD.md                           # Product Requirements Document
├── User_Stories.md                  # User Stories by epic
├── Test_Plan.md                     # Test scope, approach, entry/exit criteria
├── Test_Cases.md                    # All test cases with Passed/Failed status
├── List_Feedback.md                 # Bug report (from manual testing or exploratory testing)
└── Smoke_Test_Report.md             # Smoke test results (after retest phase)

tests/playwright/
├── playwright.config.ts
├── pages/                           # Page Object Models
└── tests/                           # Spec files (one per module)

playwright-report/
└── index.html                       # HTML automation report
```

---

## How It Works

Every agent follows the same pattern:

1. **Ask** — collect required inputs (URL, file paths) before doing anything
2. **Execute** — run the QA task using the appropriate skills
3. **Approval gate** — show a summary and ask for your review
4. **Save** — only write files after explicit approval

No file is written without your sign-off.

---

## QA Exploratory Testing Agent

`/qa-exploratory-testing` is a full structured exploratory testing agent, not just a free-form exploration tool. It runs a 7-step workflow:

1. **Gather Inputs** — URL, credentials, module scope
2. **Template Selection** — choose bug report format before testing starts
3. **Plan Session** — write a test charter using SBTM methodology + FEW HICCUPS/HICCUPS heuristics
4. **Explore** — execute using `agent-browser` + `find-bugs`, guided by `Checklist.md`
5. **Compile** — expand one-line notes into full bug descriptions per chosen template
6. **Approval Gate** — review full bug list before saving
7. **Save** — write or append to `qa-docs/List_Feedback.md`

### Checklist.md — 22-Point Coverage

The agent reads `Checklist.md` at the start of every session and applies all relevant sections per feature:

| # | Section | What it covers |
|---|---------|----------------|
| 1 | Input Validation | Empty, max length, special chars, unicode, emoji, autofill |
| 2 | Duplicate Data | Email, phone, NIK — case sensitivity, hidden spaces |
| 3 | Boundary Value | Min-1, Min, Min+1, Max-1, Max, Max+1 |
| 4 | Business Rules | Date, age, status transitions, approval workflow |
| 5 | CRUD | Create → Read → Update → Delete → Restore |
| 6 | Search | Keyword, partial, special chars, SQL keywords |
| 7 | Filter | Single, multiple, reset, empty, invalid |
| 8 | Sorting | Asc, desc, alphabet, number, date, empty values |
| 9 | Pagination | First, last, next, prev, page sizes, empty pages |
| 10 | File Upload | Type, size, filename, integrity (corrupted, renamed ext) |
| 11 | Network | Failed requests, payload, status, timeout, duplicates |
| 12 | API | Status code, error/success messages, response structure |
| 13 | UI | Alignment, overflow, broken UI, icons, loading states |
| 14 | Responsive | Desktop, tablet, mobile, landscape, portrait |
| 15 | Session | Refresh, logout, expiry, browser back/forward, tabs |
| 16 | Role & Permission | Admin, manager, staff, guest, unauthorized |
| 17 | User Behavior | Double-click, spam click, rapid type, ESC, drag & drop |
| 18 | Error Handling | Error messages, infinite loading, blank/white screen |
| 19 | Security | HTML/script tags, SQL keywords, traversal patterns |
| 20 | Data Integrity | UI → API → DB → audit log → export → notification |
| 21 | Workflow | Status transitions: draft → submit → approve → reject |
| 22 | UX Observation | Confusing flows, excessive clicks, poor labels/feedback |

**Speed rules:** scan before acting, batch inputs, one pass per screen, 2-min time-box per section, screenshot only bugs. Target: **20 min per feature**.

---

## List Feedback Templates

When any agent produces `List_Feedback.md`, it always pauses and asks which template to use. Two formats are available:

### Standard Format (English)

| Column | Description |
|--------|-------------|
| Bug ID | `BUG-001`, `BUG-002`, … |
| TC ID | Linked test case ID, or `—` for exploratory bugs |
| Bug Title | Short English title |
| Severity | Critical / High / Medium / Low |
| Module | Feature area |
| Steps to Reproduce | Numbered steps |
| Expected Behavior | What should happen |
| Actual Behavior | What actually happened |
| Status | Open (default) |
| Dev Notes | Blank — filled by developer |

### Kabayan Format (Indonesian)

Professional Indonesian-language bug report with 10 columns:

| Col | Header | Notes |
|-----|--------|-------|
| A | Defect ID | `<AppInitials>-01`, e.g. `LA-01` for LaCi RW |
| B | Date | `DD-MMM-YYYY`, e.g. `12-Jul-2026` |
| C | Crede | Role or username/password used during testing |
| D | Fitur | Hierarchical nav path, e.g. `Menu - Submenu - Action` |
| E | Bug Description | Indonesian — observed behavior, expected, steps + optional `Baiknya...` UX note |
| F | Priority | Low / Medium / High / Critical |
| G | Status | Open → On Progress → Ready to Test → Done (full lifecycle) |
| H | QA | Always `Cahya` |
| I | Attachment / Comment From QA | Screenshots + technical notes |
| J | Attachment / Comment From Dev | Blank — filled by developer |

**Status lifecycle:** Open → On Progress → Hold → Ready to Test → Failed / Done → Ready for Deploy

---

## Test Case Template: Traceability Matrix CVKC

The QA Documentation agent builds test cases using the standard template: **"Template Tracebility Matrix CVKC"** (typo intentional — matches original).

**Template spreadsheet:** `1cOELnwdDg4BdH8wXLI4jpVdtaOcqvMqC_XEfftQ57NU`

### Structure

- **Sheet 1:** List Feedback Internal (bug/feedback recap)
- **Sheet 2+:** Module sheets named `Rxx - Module Name` (e.g. R01, R02)

Each module sheet contains:
- **SC blocks** — Scenario grouping (`#SCxx`)
- **TC blocks** — Test Cases with step-by-step rows (`#TCxx`)

### Status Defaults

| Role | Default | Valid values |
|------|---------|-------------|
| Developer | `Pending` | Pending / Failed / Success |
| Tester | `Need Test` | Need Test / Failed / Success |
| CLIENT | `Pending` | Pending / Failed / Success |

**Date format:** `DD-MM-YYYY` (e.g. `2-Jul-2026`)

### Cell Colors

| Row type | Color |
|----------|-------|
| SC header (`#SCxx`) | Orange |
| TC header (`#TCxx`) | Red |
| Sub-header | Teal |
| Col-labels | Teal (D–L), white (A–C) |
| Step rows | Blue |
| Separator / empty | White |

### Build Rules (Google Sheets API)

- Always `duplicateSheet` from R01 then `copyPaste` — never build structure manually
- `time.sleep(0.4)` before every API call (quota failures return exit code 0 with empty JSON — silent)
- `insertDimension` loses cell color — always follow with `copyPaste PASTE_FORMAT` from adjacent correct row

Full build rules and pitfalls are documented in `CLAUDE.md` at the project root.

---

## Prerequisites

The following Claude Code skills must be installed:

- `agent-browser`
- `exploratory-testing`
- `webapp-testing`
- `find-bugs`
- `qa-test-planner`
- `test-case-writing`
- `smoke-test`
- `ai-regression-testing`
- `playwright-cli`
- `playwright-best-practices`
- `playwright-generate-test`
- `playwright-explore-website`
- `playwright-automation-fill-in-form`

---

## License

MIT
