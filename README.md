# QA Agent — Claude Code Plugin

End-to-end QA workflow with 6 specialized agents. From staging URL to Playwright automation scripts, with user approval gates at every step.

## Workflow

```
/qa-agent-prd              →  PRD.md + User_Stories.md
/qa-agent-documentation    →  Test_Plan.md + Test_Cases.md
/qa-agent-manual-testing   →  Test_Cases.md (status) + List_Feedback.md
        *** Dev fixes bugs ***
/qa-agent-retest           →  Smoke_Test_Report.md + updated files
/qa-agent-automation       →  Playwright scripts + HTML report
```

Run the full cycle at once with `/qa-agent`, or enter at any phase individually.

Use `/exploratory-testing` at any point for unscripted ad-hoc exploration.

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
| `/qa-agent [url]` | Full cycle | Runs all 5 phases in sequence with approval gates |
| `/qa-agent-prd [url]` | Phase 1 | Explore staging app → write `PRD.md` + `User_Stories.md` |
| `/qa-agent-documentation` | Phase 2 | Read PRD/User Stories → write `Test_Plan.md` + `Test_Cases.md` |
| `/qa-agent-manual-testing` | Phase 3 | Execute every test case → update pass/fail + write `List_Feedback.md` |
| `/qa-agent-retest` | Phase 4 | Retest failures, run smoke + regression checks |
| `/qa-agent-automation` | Phase 5 | Generate Playwright scripts + HTML report |
| `/exploratory-testing [url]` | Any time | Ad-hoc unscripted exploration → session report with findings |

---

## Agents & Skills

| Agent | Skills Used | Output |
|-------|-------------|--------|
| QA PRD | `agent-browser` | `qa-docs/PRD.md`, `qa-docs/User_Stories.md` |
| QA Documentation | `qa-test-planner`, `test-case-writing` | `qa-docs/Test_Plan.md`, `qa-docs/Test_Cases.md` |
| QA Manual Testing | `webapp-testing`, `find-bugs`, `agent-browser` | Updated `Test_Cases.md`, `qa-docs/List_Feedback.md` |
| QA Retest | `smoke-test`, `ai-regression-testing`, `agent-browser` | `qa-docs/Smoke_Test_Report.md`, updated files |
| QA Automation | `playwright-cli`, `playwright-best-practices`, `playwright-generate-test`, `playwright-explore-website`, `playwright-automation-fill-in-form` | `tests/playwright/`, `playwright-report/index.html` |
| Exploratory Testing | `agent-browser` | `qa-docs/Exploratory_Session_[date].md` |

---

## Output Files

All documentation is written to `qa-docs/` in your project:

```
qa-docs/
├── PRD.md                           # Product Requirements Document
├── User_Stories.md                  # User Stories by epic
├── Test_Plan.md                     # Test scope, approach, entry/exit criteria
├── Test_Cases.md                    # All test cases with Passed/Failed status
├── List_Feedback.md                 # Bug report with severity and status
├── Smoke_Test_Report.md             # Smoke test results (after retest phase)
└── Exploratory_Session_[date].md    # Ad-hoc session findings (if run)

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

## Test Case Template: Traceability Matrix CVKC

The QA Documentation agent builds test cases using your project's standard template: **"Template Tracebility Matrix CVKC"** (typo intentional — matches original).

**Template spreadsheet:** `1cOELnwdDg4BdH8wXLI4jpVdtaOcqvMqC_XEfftQ57NU`

### Structure

- **Sheet 1:** List Feedback Internal (bug/feedback recap)
- **Sheet 2+:** Module sheets named `Rxx - Module Name` (e.g. R01, R02)

Each module sheet contains:
- **SC blocks** — Scenario/Scenario-Class grouping (`#SCxx`)
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
