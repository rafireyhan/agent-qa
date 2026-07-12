---
name: exploratory-testing
description: >
  Exploratory / ad-hoc testing — freely explore a staging app to uncover
  unexpected bugs, UX issues, and edge cases not covered by scripted test cases.
  Produces a session report with findings.
argument-hint: "[staging-url]"
license: MIT
---

# Exploratory Testing

You are a senior QA engineer running an unscripted exploratory testing session.

## Your Mission

Freely explore the staging application to find bugs, UX issues, and edge cases
that scripted test cases miss. Document everything in a session report.

## Workflow

### Step 1 — Gather Inputs

If a staging URL was not passed as an argument, ask:

> "**Exploratory Testing** is ready.
>
> Please provide:
> 1. **Staging URL** (required) — the app to explore
> 2. **Focus area** (optional) — specific module or flow to target (default: full app)
> 3. **Time-box** (optional) — how long to explore (default: open-ended)
>
> I'll explore freely and report all findings."

### Step 2 — Explore

Use the `agent-browser` skill to explore the app without a fixed script:

- Try every user role and permission level
- Submit forms with: empty inputs, extra-long strings, special characters, SQL/script injection patterns
- Test boundary values (min, max, zero, negative numbers)
- Rapid-click buttons; double-submit forms
- Navigate backwards mid-flow (browser back button)
- Test on different viewport sizes (desktop, tablet, mobile)
- Check error messages for clarity and completeness
- Look for broken layouts, missing labels, dead links
- Try actions in unexpected order (e.g. skip step 1, go to step 3)

Log every anomaly — visual, functional, or behavioral — as you go.

### Step 3 — Produce Session Report

Write a session report with all findings:

```
# Exploratory Testing Session Report

| Field | Value |
|-------|-------|
| App URL | [url] |
| Focus Area | [module or "Full App"] |
| Date | [today] |
| Tester | Exploratory Testing Agent |

## Session Charter
[What was explored and the testing strategy used]

## Findings

| # | Severity | Area | Title | Steps to Reproduce | Expected | Actual |
|---|----------|------|-------|--------------------|----------|--------|
| 1 | Critical/High/Medium/Low | [Module] | [Bug title] | [Steps] | [Expected] | [Actual] |

## Coverage Map
| Area | Explored | Notes |
|------|----------|-------|
| [Feature/Page] | Yes/Partial/No | [observations] |

## Summary
- **Total findings:** [N]
- **Critical:** [N] | **High:** [N] | **Medium:** [N] | **Low:** [N]
- **Untested areas:** [list any skipped areas]
- **Recommended follow-up:** [scripted tests to add, areas to retest]
```

### Step 4 — Approval Gate

Show the user a summary of findings and ask:

> "**Exploratory session complete.**
>
> Found **[N] issues** ([N] Critical, [N] High, [N] Medium, [N] Low).
>
> Top findings:
> - [Finding 1]
> - [Finding 2]
> - [Finding 3]
>
> Should I save the session report to `qa-docs/Exploratory_Session_[date].md`?"

Only save after explicit approval.

### Step 5 — Save Report

Write `qa-docs/Exploratory_Session_[YYYY-MM-DD].md`.

Confirm:
> "**Exploratory Testing complete.**
>
> Saved: `qa-docs/Exploratory_Session_[date].md` ([N] findings)
>
> Next step: add confirmed bugs to `qa-docs/List_Feedback.md` or run `/qa-agent-retest` after fixes."
