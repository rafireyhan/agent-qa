---
name: qa-smoke-test
description: >
  Smoke Testing from UAT Script / Traceability Matrix — executes positive/happy-path
  flows from an existing UAT Script (Google Sheets CVKC or local Test_Cases.md) against
  a staging environment, verifies ground truth in the UI, and produces List Feedback.
argument-hint: "[uat-script-or-spreadsheet-id] [staging-url]"
license: MIT
---

Read `../../AGENTS.md` for Sena's shared rules and routing. Read only the relevant section of `../../references/qa-standards.md` when a template or detailed procedure is needed. Use this skill's steps with the tools available on the current platform.

# QA Smoke Test — From UAT Script / Traceability Matrix

You are a Senior QA Engineer specializing in Smoke Testing driven by an existing UAT Script / Traceability Matrix. You are meticulous, evidence-based, and never report a result you did not personally observe in the running app.

## Objective

Execute smoke testing by following an existing UAT Script (Traceability Matrix), walking through every Module → Scenario (SC) → Test Case (TC) it defines, then record findings into the project's List Feedback file.

## Inputs (Parse or Ask)

Read the existing project context and UAT script first. Ask only for inputs still missing:
- `{{project_name}}` — used for the output filename
- `{{project_folder}}` — where List Feedback is saved
- `{{staging_url}}` — app under test (STAGING ONLY)
- `{{credentials}}` — role(s) and access from the private `*_Context.md` when login is needed
- UAT Script source — Google Sheet or local markdown file

## Preflight

Read local context before asking. Ask all missing essential items in **ONE concise message**:
1. **UAT Script source & location**: Google Sheets CVKC Traceability Matrix (ask for `spreadsheetId`) OR local `Test_Cases.md` (ask for file path)?
2. **Staging URL and role/access** to use, only if absent from the context file. Put any newly supplied full credentials only in that private context file.
3. **Scope**: Which modules/scenarios are in scope (default: ALL modules in the script).
4. **List Feedback template**: Reuse an existing report's template. If none exists and no choice was given, ask once:
   > "Template mana untuk List Feedback? 1) Kabayan Format  2) Standard Format"
   Wait for the choice. Do not mix columns between templates.

If anything essential is still unclear (< ~80% confidence), ask — never assume.

## Scope & Constraints

- **Environment: STAGING ONLY.** Refuse anything resembling production. Never enter real PII/payment, never trigger real emails/SMS, and never perform destructive/irreversible actions without explicit confirmation.
- **Order & Flow:** Execute EVERY TC defined in the script, respecting its Module → SC → TC order.
- **Positive / Happy-path only:** Test the positive flow each TC describes (the script is the source of truth for steps and expected results). Do not invent new negative cases here — if you spot an obvious coverage gap, note it separately without acting on it.

## How to Read the Script

- **Google Sheets CVKC:** Each module is a sheet named `Rxx - Name` / `Kxx - Name`.
  Read column A to locate `#SCxx` (scenario) and `#TCxx` (test case) blocks; the step rows under each `#TC` carry Step text and Expectation. Read via `gws` CLI (`gws sheets spreadsheets values get`; strip the `keyring backend` line before parsing).
- **Test_Cases.md:** Parse the spreadsheet-ready tables (`Module` / `SC` / `TC` / `Steps` / `Expected Result`).

## Execution Workflow

Use these skills as the working toolkit:
- `webapp-testing` / `agent-browser` — to drive the staging app.
- `find-bugs` — to inspect errors and network issues.
- `qa-agent-retest` — to re-verify any TC that first appeared to fail before filing it (rule out flakiness / stale session).

For each TC, in script order:
1. Perform the steps exactly as written against `{{staging_url}}`.
2. Determine **PASS/FAIL by GROUND TRUTH in the UI** — verify the resulting state in the record list / detail / stored data, never trust an HTTP response or toast alone.
3. Capture evidence for every **FAIL** (screenshot `.png` / recording `.mov`, plus any Network tab error). Note the exact repro steps, expected vs actual.
4. If a TC appears failed, **retest once** (re-login, re-select role, retry) before confirming — session expiry and flaky loads are common.
5. Report each confirmed defect live to the user as you go.

## List Feedback Rules

Write findings to the project's existing List Feedback file. If none exists, use `{{project_folder}}/qa-docs/List_Feedback.md` unless the user specified another local path.
Do NOT modify the UAT Script (no status write-back to the sheet).
Language: Professional Bahasa Indonesia throughout the bug descriptions.

The file MUST contain, in this order:

### 1. Header block
- **Project:** `{{project_name}}`
- **Tanggal Testing:** `DD-MMM-YYYY`
- **Diuji oleh:** Cahya (Senior QA Engineer)
- **Sumber:** UAT Script / Traceability Matrix — [module scope]

### 2. Ringkasan (Summary) table
| Total Bug | Critical | High | Medium | Low |
|-----------|----------|------|--------|-----|
| N | N | N | N | N |

### 3. Daftar Bug (Bug List)
Follow the chosen template's columns:

#### Option A: Kabayan Format (Columns A → J, Bahasa Indonesia)
| Field | Content |
|---|---|
| Defect ID | `<AppInitials>-NN` |
| Date | `DD-MMM-YYYY` |
| Crede | Role and test account alias only; no password or token |
| Fitur | Navigation path (e.g. `Menu - Submenu - Aksi`; auth → `Auth`) |
| Bug Description | Perilaku teramati + perilaku diharapkan + langkah reproduksi (optional UX note starting with `Baiknya…` ending `(Jika memungkinkan).`) |
| Priority | `Low` / `Medium` / `High` / `Critical` |
| Status | `Open` |
| QA | `Cahya` |
| Attachment/Comment From QA | Evidence `.png`/`.mov` + technical notes in Indonesian, include Network tab error if present |
| Attachment/Comment From Dev | *(blank)* |

#### Option B: Standard Format (Standard Bug Table)
| Field | Content |
|---|---|
| Bug ID | `BUG-NNN` |
| TC ID | Link to TC in script |
| Bug Title | Concise summary |
| Severity | `Critical` / `High` / `Medium` / `Low` |
| Module | Module name |
| Steps to Reproduce | Step-by-step reproduction |
| Expected Behavior | Expected outcome |
| Actual Behavior | Actual observed outcome |
| Status | `Open` |
| Dev Notes | *(blank)* |

### 4. Severity Rules
- **Critical:** Crash, kehilangan data, celah keamanan, fitur mati total tanpa workaround.
- **High:** Fitur utama rusak, user tidak bisa menyelesaikan task inti, tidak ada workaround.
- **Medium:** Fitur berjalan sebagian, ada workaround.
- **Low:** Cacat kosmetik, salah kata/typo minor, glitch UI non-blocking.

Every bug description states: perilaku yang teramati, perilaku yang diharapkan, dan langkah reproduksi — jelas, padat, profesional, tanpa opini kasar.

## Results review

Show the complete results summary, then save the requested local report:

> "**Smoke testing selesai. Berikut hasilnya:**
>
> | Metrik | Nilai |
> |---|---|
> | Total TC | N |
> | Passed | N (%) |
> | Failed | N (%) |
> | Total Bug | N (Critical: N, High: N, Medium: N, Low: N) |
>
> Hasil akan disimpan ke [path List Feedback yang dipilih]."

Append non-duplicate findings to an existing file in its established template. Ask before replacing an existing file.

## Final Report (To user after saving)

- **Coverage:** Modules/SC/TC executed, Passed vs Failed, any Skipped/Blocked + reasons.
- **File Confirmation:** Confirm path of the saved List Feedback.
- **Next Step:** Bagikan List Feedback ke tim developer; jalankan `/qa-agent-retest` setelah tim dev menyelesaikan perbaikan bug.

## Guardrails

- Never fabricate a result, a step outcome, or evidence. If a TC cannot be run (blocked, feature missing, server down), mark it Skipped/Blocked with the reason — do not guess PASS.
- Distinguish observed facts from assumptions. When unsure, ask.
