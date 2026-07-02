# QA Agent — Claude Code Plugin

End-to-end QA workflow with 5 specialized agents. From staging URL to Playwright automation scripts, with user approval gates at every step.

## Workflow

```
/qa-agent-prd          →  PRD.md + User_Stories.md
/qa-agent-documentation →  Test_Plan.md + Test_Cases.md
/qa-agent-manual-testing →  Test_Cases.md (status) + List_Feedback.md
        *** Dev fixes bugs ***
/qa-agent-retest       →  Smoke_Test_Report.md + updated files
/qa-agent-automation   →  Playwright scripts + HTML report
```

Run the full cycle at once with `/qa-agent`, or enter at any phase individually.

## Installation

```
/plugin install github:rafireyhan/agent-qa
```

## Commands

| Command | Description |
|---------|-------------|
| `/qa-agent [url]` | Full cycle — all 5 phases with approval gates |
| `/qa-agent-prd [url]` | Phase 1 — explore app, write PRD.md + User_Stories.md |
| `/qa-agent-documentation` | Phase 2 — write Test_Plan.md + Test_Cases.md |
| `/qa-agent-manual-testing` | Phase 3 — execute tests, update status, write List_Feedback.md |
| `/qa-agent-retest` | Phase 4 — retest failures, run smoke + regression tests |
| `/qa-agent-automation` | Phase 5 — generate Playwright scripts + HTML report |

## Agents & Skills

| Agent | Skills Used | Output |
|-------|-------------|--------|
| QA PRD | `agent-browser` | `qa-docs/PRD.md`, `qa-docs/User_Stories.md` |
| QA Documentation | `qa-test-planner`, `test-case-writing` | `qa-docs/Test_Plan.md`, `qa-docs/Test_Cases.md` |
| QA Manual Testing | `webapp-testing`, `find-bugs`, `agent-browser` | Updated `Test_Cases.md`, `qa-docs/List_Feedback.md` |
| QA Retest | `smoke-test`, `ai-regression-testing`, `agent-browser` | `qa-docs/Smoke_Test_Report.md`, updated files |
| QA Automation | `playwright-cli`, `playwright-best-practices`, `playwright-generate-test`, `playwright-explore-website`, `playwright-automation-fill-in-form` | `tests/playwright/`, `playwright-report/index.html` |

## Output Files

All documentation is written to `qa-docs/` in your project:

```
qa-docs/
├── PRD.md                  # Product Requirements Document
├── User_Stories.md         # User Stories by epic
├── Test_Plan.md            # Test scope, approach, entry/exit criteria
├── Test_Cases.md           # All test cases with Passed/Failed status
├── List_Feedback.md        # Bug report with severity and status
└── Smoke_Test_Report.md    # Smoke test results (after retest phase)

tests/playwright/
├── playwright.config.ts
├── pages/                  # Page Object Models
└── tests/                  # Spec files (one per module)

playwright-report/
└── index.html              # HTML automation report
```

## How It Works

Every agent follows the same pattern:

1. **Ask** — collect required inputs (URL, file paths) before doing anything
2. **Execute** — run the QA task using the appropriate skills
3. **Approval gate** — show a summary and ask for your review
4. **Save** — only write files after explicit approval

No file is written without your sign-off.

## Prerequisites

The following Claude Code plugins/skills must be installed:

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

## License

MIT
