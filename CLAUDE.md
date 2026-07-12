# QA Agent Plugin — Project Context

## Test Case Template: Traceability Matrix CVKC

When building test cases using this template, follow all rules below exactly.

**Template name:** "Template Tracebility Matrix CVKC" (typo is intentional — matches original)
**Template spreadsheet:** `1cOELnwdDg4BdH8wXLI4jpVdtaOcqvMqC_XEfftQ57NU`

---

### Structure

- **Sheet 1**: List Feedback Internal (bug/feedback recap)
- **Sheet 2+**: Module sheets named `Rxx - Module Name` (zero-padded, e.g. R01, R02)

**Module sheet rows:**
- Row 1: Module header — B1 = module code + name; C1–H1 = status labels
- Rows 2–4: Summary COUNTIFS (Developer / Tester / CLIENT)
- Row 5: Empty separator
- Row 6+: SC and TC blocks

**SC block (2 rows):** `#SCxx` in A, SC name in B; then empty row. Counter resets to SC01 per module.

**TC block (3 + n_steps + 1 rows):**
- Row 0: `#TCxx` in A, TC name in B, aggregate formulas in D/G/J
- Row 1: Sub-header (`#`, Step, Expectation, Developer, Tester, CLIENT…)
- Row 2: Col-labels (Status, Date Tested, Note × 3)
- Rows 3…: Steps — `[step_num, step_text, expectation, dev_status, dev_date, dev_note, tester_status, tester_date, tester_note, client_status, client_date, client_note]`
- Last row: Empty separator. Counter resets to TC01 per SC.

---

### Status Defaults

| Role | Default | Valid values |
|------|---------|-------------|
| Developer | `Pending` | Pending / Failed / Success |
| Tester | `Need Test` | Need Test / Failed / Success |
| CLIENT | `Pending` | Pending / Failed / Success |

**Date format:** `DD-MM-YYYY` (e.g. `2-Jul-2026`)

---

### Cell Colors (backgroundColor RGB, 0–1 float)

| Row type | Color | RGB |
|----------|-------|-----|
| SC header (`#SCxx`) | Orange | A=(0.95,0.76,0.20) B–L=(1.00,0.90,0.60) |
| TC header (`#TCxx`) | Red | A=(0.92,0.60,0.60) B–L=(0.96,0.80,0.80) |
| Sub-header | Teal | (0.46,0.65,0.69) |
| Col-labels | Teal D–L, white A–C | A–C are merged from sub-header above — do not touch |
| Step rows | Blue | (0.86,0.90,0.95) all columns |
| Separator / empty | White | no background |

---

### Build Approach (Google Sheets API via gws CLI)

Always use **`duplicateSheet` → `copyPaste` from R01**. Never build structure manually.

1. Delete the old module sheet (free the name)
2. `duplicateSheet` from R01 - Marketing Pages (sheetId `904716640`) at the correct tab index
3. Clear `A5:Z1000` (removes R01 content, keeps rows 1–4 summary)
4. Update B1 with new module name
5. Fix E2 and E4 summary formulas to count `"Pending"` (not `"Need Test"`)
6. **SC blocks**: `copyPaste` R01 rows 5–6 (0-indexed), overwrite A/B with `#SCxx` and name
7. **TC blocks**: `copyPaste` R01 rows 7–15 (0-indexed, 9 rows), then:
   - **<5 steps**: `deleteDimension` excess step rows
   - **>5 steps**: `insertDimension` at `current_row + 8` with `inheritFromBefore: true`, then immediately `copyPaste PASTE_FORMAT` from the step row above (insertDimension does NOT copy colors)
8. Overwrite A/B of TC header with `#TCxx` and name
9. Overwrite step rows with correct values and defaults

**`time.sleep(0.4)` before every API call** — quota failures return exit code 0 with empty JSON (completely silent).

---

### Known Formatting Pitfalls

1. **`insertDimension` loses cell color** — fix with `copyPaste PASTE_FORMAT` from adjacent correct step row immediately after insert
2. **Failed `copyPaste` leaks R01 colors** — if quota hit, target rows keep R01 formatting; fix with `copyPaste PASTE_FORMAT` from correct reference row of same type
3. **`copy_3_rows` partial fix leaves step rows wrong** — always also fix step row colors after any partial TC header fix

**Fix always:** `copyPaste PASTE_FORMAT` from a visually-correct row of the same type in the same sheet. Never hardcode RGB values.

---

### Post-Build Verification (required after every build/fix)

Fetch `userEnteredFormat.backgroundColor` for the full content range and confirm:
1. SC header rows → orange
2. Empty rows after SC header → white
3. TC header rows → red
4. Sub-header rows → teal
5. Col-labels rows → teal on D–L, white on A–C
6. All step rows → blue (most common failure)
7. Separator rows between TCs → white
8. Rows beyond last content → white, no ghost formatting

---

### Reference Scripts (BerUang project)
- `fix_r03.py` — duplicateSheet approach, current standard
- `fix_r04.py` — same pattern, good reference
- Target spreadsheet: `1ZNsDkJQWjE0H48UPH_ax1bwSos59KuXIJnbJwP-DUdg`

---

## QA Agents & Skills

### QA Agent Pipeline (`qa-agent:*`)

| Agent | Invoke with | What it does |
|-------|------------|-------------|
| `qa-agent-prd` | `/qa-agent-prd` | Explores staging app → writes `PRD.md` + `User_Stories.md`. Pauses for approval before saving. |
| `qa-agent-documentation` | `/qa-agent-documentation` | Reads `PRD.md` + `User_Stories.md` → generates `Test_Plan.md` + `Test_Cases.md`. Pauses for approval before saving. |
| `qa-agent-manual-testing` | `/qa-agent-manual-testing` | Executes every row in `Test_Cases.md` against staging → updates Pass/Fail + produces `List_Feedback.md`. Pauses for approval before saving. |
| `qa-agent-retest` | `/qa-agent-retest` | Re-tests previously failed cases. |
| `qa-agent-automation` | `/qa-agent-automation` | Automated test execution via Playwright. |
| `qa-agent` | `/qa-agent` | General-purpose QA agent / orchestrator. |

### Supporting QA Skills

| Skill | Invoke with | What it does |
|-------|------------|-------------|
| `qa-test-planner` | `/qa-test-planner` | Plans test strategy |
| `test-case-writing` | `/test-case-writing` | Writes test cases |
| `webapp-testing` | `/webapp-testing` | Tests web apps |
| `exploratory-testing` | `/exploratory-testing` | Exploratory / ad-hoc testing |
| `smoke-test` | `/smoke-test` | Quick smoke test run |
| `find-bugs` | `/find-bugs` | Finds bugs and security issues in current branch diff |
| `agent-browser` | `/agent-browser` | Browser automation — navigate, click, fill forms, screenshot, scrape |
