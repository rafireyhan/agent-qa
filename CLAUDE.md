# QA Agent Plugin — Project Context

## Test Case Template: Traceability Matrix CVKC

When building OR editing test cases in this template, follow all rules below exactly.

**Template name:** "Template Tracebility Matrix CVKC" (typo intentional — matches original)
**Template spreadsheet:** `1cOELnwdDg4BdH8wXLI4jpVdtaOcqvMqC_XEfftQ57NU`

---

### Structure

- **Sheet 1**: List Feedback Internal (bug/feedback recap)
- **Sheet 2+**: Module sheets named `Rxx - Module Name` (zero-padded, e.g. R01, R02) or `Kxx - Module Name`

**Module sheet rows:**
- Row 1: Module header — B1 = module code + name; C1–H1 = status labels
- Rows 2–4: Summary formulas (Developer / Tester / CLIENT) — range-based `COUNTIFS`/`COUNTIF`. They **auto-update** as SC/TC/step rows are inserted or deleted, provided new rows stay within the formula range (e.g. `$A$6:$A738`). `Case` = `COUNTIF(A:A,"#TC*")`; `Scenario` = `COUNTIF(A:A,"#SC*")`. A scenario may hold multiple TCs, so **Case ≥ Scenario is valid**.
- Row 5: Empty separator
- Row 6+: SC and TC blocks

**SC block (2 rows):** `#SCxx` in A, SC name in B; then empty row. Counter resets to SC01 per module. One SC may contain multiple TCs.

**TC block (3 + n_steps + 1 rows):**
- Row 0: `#TCxx` in A, TC name in B, aggregate roll-up formulas in D/G/J. TC counter resets to TC01 per SC.
- Row 1: Sub-header (`#`, Step, Expectation, Developer, Tester, CLIENT…)
- Row 2: Col-labels (Status, Date Tested, Note × 3)
- Rows 3…: Steps — `[step_num, step_text, expectation, dev_status, dev_date, dev_note, tester_status, tester_date, tester_note, client_status, client_date, client_note]`
- Last row: Empty separator.

**Aggregate roll-up formula (D/G/J on the TC-header row)** — rolls step statuses into one TC status:
```
D (Developer): =IF(COUNTIF(D{s}:D{e},"Failed")>0,"Failed",IF(COUNTIF(D{s}:D{e},"Pending")>0,"Pending",IF(COUNTA(D{s}:D{e})=0,"","Success")))
G (Tester):    same, but "Need Test" instead of "Pending"
J (CLIENT):    same as Developer ("Pending")
```
**Widen `{s}:{e}` to span ALL step rows of the TC.** The template base ships covering only the first 2 steps — always extend it, or a failure in step 3+ won't roll up. copyPaste shifts the relative refs but still leaves them 2 rows wide — re-set them. Easiest: read column A, find each `#TC` header and its contiguous numeric step rows, generate the range from actual positions.

Dates are stored as **serial numbers** (e.g. `46227` = 24-Jul-2026) with a `DD-MMM-YYYY` display format.

---

### Status Defaults

| Role | Default | Valid values |
|------|---------|-------------|
| Developer | `Pending` | Pending / Failed / Success |
| Tester | `Need Test` | Need Test / Failed / Success |
| CLIENT | `Pending` | Pending / Failed / Success |

**Date format:** `DD-MMM-YYYY` (e.g. `24-Jul-2026`)

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

### Test Case Authoring Rules (the CONTENT of the steps)

- **Positive / happy-path only** unless asked otherwise (K04/K08 convention). Don't add negative/validation cases unless requested — flag missing coverage separately instead.
- **List EVERY form field.** Whenever a step opens or fills a form, enumerate all fields (grouped/numbered) in BOTH the Step and Expectation — never just "fill the form". Include fields that are **auto-filled or conditional**; note the source (e.g. "→ terisi otomatis").
- **Branching forms → one TC per branch, kept symmetric.** When a control reshapes the form (e.g. `Pemohon sudah punya akun? Belum/Ya`), give each branch its own TC under the same SC, same step shape. Fold the branch choice **inline** into the single "Isi form" step with a marker (`[Belum]` / `[Ya]`) — not a separate "select option" step — and list that branch's fields (manual fields for Belum; "Cari User" + auto-filled fields for Ya).
- **Discover features by walking every status.** Row/detail actions are often gated by record status; reading current data is not enough. Create/transition records through each status to reveal all actions (e.g. Konfirmasi/Tolak show only on Pending/Approved). A status value in a filter dropdown signals a transition action exists somewhere — don't assume "just a flag".
- **Verify live before writing steps.** Click the real flow first — mandatory sub-steps are easy to miss (e.g. Tolak/Reject requires a rejection-reason field).

---

### Build Approach (Google Sheets API via gws CLI)

Never build structure manually — always **`copyPaste` an existing, correctly-formatted block, then overwrite only the text.** copyPaste carries formatting, relative-adjusted formulas, and status/date cells in one shot.

**A. New module sheet** — `duplicateSheet` → `copyPaste` from R01:
1. Delete the old module sheet (free the name)
2. `duplicateSheet` from R01 - Marketing Pages (sheetId `904716640`) at the correct tab index
3. Clear `A5:Z1000` (removes R01 content, keeps rows 1–4 summary)
4. Update B1 with the new module name
5. Fix E2 and E4 summary formulas to count `"Pending"` (not `"Need Test"`)
6. **SC blocks**: `copyPaste` R01 rows 5–6 (0-indexed), overwrite A/B with `#SCxx` and name
7. **TC blocks**: `copyPaste` R01 rows 7–15 (0-indexed, 9 rows), then adjust step count (see C)
8. Overwrite A/B of the TC header with `#TCxx` and name; **widen the aggregate formula to all steps**
9. Overwrite step rows with correct values and defaults

**B. Editing / appending to an existing sheet** (targeted only — never download/edit/upload):
- Append an SC/TC: `copyPaste` an existing same-shape block into empty rows below, then overwrite text via `values.batchUpdate`.
- Insert into the middle: `insertDimension` (`inheritFromBefore:true`) to open rows, then `copyPaste PASTE_FORMAT` from an adjacent same-type row (insert does NOT copy colors), then set values.
- Remove a step: `deleteDimension`.
- On insert/delete, Google **auto-adjusts** aggregate & summary formula ranges — verify after, but usually no manual fix needed.

**C. Adjusting step count inside a copied TC block:**
- **fewer steps**: `deleteDimension` the excess step rows
- **more steps**: `insertDimension` (`inheritFromBefore:true`), then immediately `copyPaste PASTE_FORMAT` from the step row above

**gws CLI mechanics:**
- Structural → `gws sheets spreadsheets batchUpdate --json '{"requests":[...]}'` (insertDimension, deleteDimension, copyPaste, duplicateSheet). Validate first with `--dry-run`.
- Cell values → `gws sheets spreadsheets values batchUpdate` with `valueInputOption: USER_ENTERED` (so formulas & dates parse). Batch many ranges in one call.
- gws prints `Using keyring backend: keyring` — strip it (`grep -v 'keyring backend'`) before parsing JSON.
- Pace API calls — quota failures can return exit code 0 with empty/silent JSON. (In Python scripts: `time.sleep(0.4)` before every call.)

---

### Known Formatting Pitfalls
1. **`insertDimension` loses cell color** — fix with `copyPaste PASTE_FORMAT` from an adjacent correct step row immediately after insert
2. **Failed `copyPaste` leaks source colors** — if quota hit, target rows keep the source's formatting; fix with `copyPaste PASTE_FORMAT` from a correct reference row of the same type
3. **Aggregate formula too narrow** — after any copyPaste/insert it may cover only the first 2 steps; widen to all steps
4. **Partial TC fixes leave step rows wrong** — always also fix step-row colors after any partial TC-header fix

**Fix always:** `copyPaste PASTE_FORMAT` from a visually-correct row of the same type in the same sheet. Never hardcode RGB.

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

Also confirm: SC/TC numbering is sequential with no duplicates; each TC-header aggregate formula spans all its steps; summary counts (Case / Scenario / statuses) updated correctly.

---

### Reference Scripts & Examples
- `fix_r03.py` / `fix_r04.py` — duplicateSheet approach in Python (BerUang project, spreadsheet `1ZNsDkJQWjE0H48UPH_ax1bwSos59KuXIJnbJwP-DUdg`)
- Worked example, gws CLI editing an existing sheet (append SC/TC, branch TCs, widen formulas): **K08 - Appointment** in `1-aahhHKmr5o3aAW_jvysPNEa-mVZ7MT0BUnHBrzZv0U`

---

## Test Case Template: TM Kabayan ver Lengkap

A **separate, simpler** traceability-matrix template — do NOT confuse with CVKC. No formulas, no roll-ups, includes negative cases, inline test data. When the user names this template, follow the rules below.

**Reference example (visual + structure master):** `13MrZy4agm1J0X91OZ7uLbOG-YF1DTcy5hchD2aYybcs` ("Traceability Matrix SKK Migas - PDSI"). copyPaste blocks from here when building.

### Structure

- **One sheet per module**, named `MO_00x - Module Name` (zero-padded).
- **Header block (rows 1–5, teal band `RGB 0.486/0.655/0.808`, bold, plain text — NO formulas):**
  | Row | A (label) | C (value) |
  |---|---|---|
  | 1 | Test Case ID | module code (`MO_010`) |
  | 2 | Tester's Name | testing team name |
  | 3 | Test Case Description | e.g. "Tes User Management" |
  | 4 | Enviroment : URL | "Staging <App> : <url>" *(keep the source typo "Enviroment")* |
  | 5 | Test Case (Success/Fail/Not Executed) | static legend text only |
- (blank row) → **SC block**, repeated per scenario:
  - `SC 0x` in A, scenario name in B
  - `Tujuan` in A, objective sentence in B
  - (blank row)
  - **2-row teal column-header band** (see columns below)
  - TC rows (see layouts below)
  - (one blank separator row before the next SC; TCs within an SC are NOT separated)

### Columns (A–M, 13 cols)

`Step | Test Case | Jenis Test Case | Test Data | Test Step | Expected Result | Actual Result | Tester[Status | Date Tested] | Client[Status | Date Tested] | Evidence | Notes`

Column-header band spans 2 rows: A–G, L, M merged vertically across both rows; **Tester** (H–I) and **Client** (J–K) have a merged top label with a `Status | Date Tested` sub-row beneath.

### TC layouts (both valid, pick per TC)

- **Single-row TC** — simple linear flow: `#TC 0x` in A, all steps numbered `1.\n2.\n3.` inside ONE *Test Step* cell, one *Expected Result* cell.
- **Multi-row TC** — form-fill / multi-step flows where each step has a distinct expectation: a `#TC 0x` header row, then continuation rows with **A–D blank**, each carrying one *Test Step* + its own *Expected Result*.

### Rules & mechanics

- **Includes negative cases.** `Jenis Test Case` dropdown = `[Positif, Negatif]` — write both happy-path and negative/validation cases. (Opposite of the CVKC positive-only rule.)
- **Inline test data.** The `Test Data` column carries credentials/inputs (e.g. `username : dummy.pdsi\npassword : *****`). (Opposite of the CVKC no-test-data rule.)
- **No formulas anywhere** — no summary COUNTIF, no aggregate roll-up. Status is plain text.
- **Status:** Tester col `H` + Client col `J`, dropdown `[Pass, Failed]`; **default = blank** for untested rows. No status color-coding (Pass cells stay white). Only two roles — **Tester + Client** (no Developer).
- **Dates:** serial numbers, display `DD/MMM/YYYY` (e.g. `21/Oct/2025`).
- `#TC` numbering resets to 01 at every new SC.
- **Build via gws CLI:** `copyPaste` an existing correctly-formatted SC/TC block from the reference sheet, then overwrite text via `values.batchUpdate` (`USER_ENTERED`). Same targeted-edit discipline as CVKC — never download/edit/upload.

### House-style rules (Cahya — apply on every build; override the SKK reference where they conflict)
- **Multi-row TC by default**: each source step = its own row with its own Expected Result; keep inline `N.` numbering in Test Step; #TC header row carries step 1.
- **Merge A/B/C/D vertically per multi-step TC** (one merge per column across the TC's step rows) so #TC / Test Case / Jenis / Test Data each span the whole TC.
- **Wrap = default on all content cells** (`wrapStrategy: WRAP`, `verticalAlignment: MIDDLE`). Reference ships Test Step (E) unwrapped → always set WRAP.
- **Set explicit ROW HEIGHTS from content** (see gotcha below) so nothing is clipped.
- **Expected Result ≠ Actual Result** — never mirror. Actual = a natural paraphrase, same substance → still Pass. Method: collect unique Expected *heads* (text before first `\n`), author one observed-outcome variant each ("Berhasil menampilkan X"→"X tampil"; "Menampilkan Form:"→"Form tampil:"; "Berhasil login"→"Login berhasil dilakukan"), preserve form field bullet-lists verbatim (paraphrase head only). Keep a growing head→actual MAP and extend per new module.
- **Status/date/Actual repeat on EVERY step row** of a multi-row TC. When recording a passed run: Tester=Pass + test date; Client left blank.
- **Sheet naming:** keep the source module codes for traceability (don't force `MO_00x`) if converting an existing matrix; ask.
- **Status baseline for a plain conversion (no live test):** carry the CVKC source status (usually Success) over as `Tester = Pass + a baseline date` (confirm the date with the user). For a **designed-but-not-live-tested** TC (e.g. negatives authored on paper), leave **Tester blank** and phrase Actual Result as "(pending — belum ditest live)" so it's clearly distinguishable from a verified pass. Never mark Pass without evidence.
- Default column widths that work (px): A66 B330 C145 D260 E333 F282 G263 H146 I98 J130 K110 L208 M208.

### Build mechanics (gws CLI + Python)
- If a target file is pre-seeded with reference sheets, **repurpose one as the formatting master** (rename, overwrite header C1:C4, rebuild SC/TC region). To make a 2nd module fast, **duplicateSheet a finished module sheet** as the new base. ⚠️ Not every seeded "MO_00x" sheet is the full Lengkap layout — some (e.g. a "Dashboard" variant) use fewer columns / `TS` not `SC`; verify structure before using as master.
- Units: scaffold = 5 rows [SC, Tujuan, blank, col-header-top, col-header-sub]; data = 1 formatted data row (carries `Positif/Negatif` + `Pass/Failed` validation). copyPaste a single data row into an N-row destination tiles it.
- Robust rebuild (avoids leftover ref formatting on separators): copyPaste scaffold+data to a **scratch area** far below → `unmergeCells`+`repeatCell` clear (fields `userEnteredFormat,userEnteredValue,dataValidation`, cell `{}`) the build area → stamp scaffolds+data from scratch (PASTE_NORMAL) → write all values (USER_ENTERED) → clear scratch. Layout: header 1–5, blank row 6, first SC scaffold at row 7; per SC = 5 scaffold + N data + 1 separator.
- **Parse the source matrix programmatically** — don't hand-transcribe: `#SCxx`/`#TCxx`/`#`(subheader)/digit-in-colA(step). Renumber SC sequential, `#TC 0x` reset per SC, derive Tujuan per SC.
- Date serial example: `18-Aug-2026`=`46252` (24-Jul-2026=46227, 24-Aug-2026=46258).

#### Canonical script pipeline (battle-tested; recreate & adapt per project)
Run in order; each is a small Python script driving gws. Adapt constants per file: SRC id + sheet, DST id, DST sheetId, DST name, TUJUAN dict (1 objective/SC), DATE serial, and (for the paraphrase) a fresh MAP.
1. **build** — rename target sheet, preserve scaffold(rows7–11)+data-row(12) to scratch, unmerge+clear build area, stamp scaffolds+data, parse source, write all values (multi-row TCs; A=`#TC 0x`, B name, C `Positif`, D blank, E `N. step`, F=Expectation, G=Expectation initially, H `Pass`, I date). Add a `--parse-only` guard; verify SC/TC/step counts before the full run.
2. **merge_wrap** — merge A/B/C/D vertically per multi-step TC + WRAP/MIDDLE on all content.
3. **actual** — Expected≠Actual: extract unique Expected heads, author a paraphrase MAP (assert all mapped, no KeyError), write col G; preserve bullet tails.
4. **set_heights** — explicit per-row heights from E/F/G (see gotcha).
Then delete any leftover seeded template sheet; verify.

#### Adding a negative TC to an existing built sheet (helper pattern)
`insertDimension` N step rows at the target position (inheritFromBefore) → `copyPaste` PASTE_NORMAL from one existing data row (carries borders + `[Positif,Negatif]`/`[Pass,Failed]` validation; a single-row source won't copy the A-D merge, which is what you want) → `mergeCells` A/B/C/D across the N rows → write values (A `#TC 0x`, B name, C `Negatif`, D test-data, E/F/G, H status, I date — H/I blank for designed/pending) → set heights. **When inserting into several SCs, go bottom-up** (highest row first) so earlier inserts don't shift later target rows.

#### Decomposing multi-file conversions (parallel subagents)
Independent file conversions (one module sheet each) parallelize cleanly. Dispatch one general-purpose subagent per file. **Cold subagents need a fully self-contained prompt**: confirmed params, the full house-style, the destination's seeded-template situation, and an explicit pointer to the reusable scripts (paths) with "read + adapt, don't reinvent" — including that they must author their own paraphrase MAP + TUJUAN. Warn them of the gws gotchas and "no concurrent sheet-mutations".

### Gotchas (bit us — reusable)
- **gws:** parse **stdout only** (merging stderr corrupts JSON); reads can transiently return empty / HTTP 500 → retry 3–5× before asserting; strip the `keyring backend` line; pace calls (`time.sleep(0.4)`).
- **Row height / auto-fit:** `autoResizeDimensions` (ROWS) does **NOT** grow rows in this template — wrapped multi-line cells stay ~20px and get clipped, and it fails **even after unmerging** the A–D merges. Fix = compute height explicitly per row from content and set via `updateDimensionProperties`: `max_lines` across E/F/G where a cell's lines = Σ over `\n`-segments of `ceil(len(seg)/chars_per_line)`, `chars_per_line ≈ colwidth_px/7`; `height = max(21, max_lines*15 + 8)`. Heights depend on column widths → **recompute if widths change**.
- **Concurrency:** never run two sheet-mutating scripts at once (e.g. a backgrounded run + a foreground rerun) — races produce duplicate/overlapping merges (saw 128 A–D merges instead of 88). Run one at a time; verify final merge count == 4 × (#multi-step TCs).

## QA Agents & Skills

### QA Agent Pipeline (`qa-agent:*`)
Use these in order for a full QA cycle:

| Agent | Invoke with | What it does |
|-------|------------|-------------|
| `qa-agent-app-discovery` | `/qa-agent-app-discovery` | "Learn the app first" — explores a new staging app end-to-end across every role/menu → writes `{{project_name}}_Context.md` (discovery baseline before PRD). Confirms save path before writing. |
| `qa-agent-prd` | `/qa-agent-prd` | Explores staging app → writes `PRD.md` + `User_Stories.md`. Pauses for approval before saving. |
| `qa-agent-documentation` | `/qa-agent-documentation` | Reads `PRD.md` + `User_Stories.md` → generates `Test_Plan.md` + `Test_Cases.md` (spreadsheet-ready tables). Pauses for approval before saving. |
| `qa-agent-manual-testing` | `/qa-agent-manual-testing` | Executes every row in `Test_Cases.md` against staging → updates Pass/Fail + produces `List_Feedback.md`. Pauses for approval before saving. |
| `qa-agent-retest` | `/qa-agent-retest` | Re-tests previously failed cases. |
| `qa-agent-automation` | `/qa-agent-automation` | Automated test execution. |
| `qa-agent` | `/qa-agent` | General-purpose QA agent. |
| `qa-exploratory-testing` | `/qa-exploratory-testing` | Structured exploratory testing on a live URL using `exploratory-testing` (SBTM/charters/heuristics) + `agent-browser` + `find-bugs` → creates/updates `List_Feedback.md`. Asks template selection before saving. |

### Supporting QA Skills (standalone or used internally by agents above)

| Skill | Invoke with | What it does |
|-------|------------|-------------|
| `qa-test-planner` | `/qa-test-planner` | Plans test strategy |
| `test-case-writing` | `/test-case-writing` | Writes test cases |
| `webapp-testing` | `/webapp-testing` | Tests web apps |
| `smoke-test` | `/smoke-test` | Quick smoke test run |
| `find-bugs` | `/find-bugs` | Finds bugs and security issues in current branch diff |
| `agent-browser` | `/agent-browser` | Browser automation — navigate, click, fill forms, screenshot, scrape |

### Exploratory Testing Principles (`/qa-exploratory-testing`)

Durable operating rules for exploratory testing across all projects. The full
step-by-step workflow lives in the project skill
(`agent-qa/skills/qa-exploratory-testing/SKILL.md`) — this is the summary that
must always hold:

- **Charter-driven, not random.** Explore freely (no scripts), but every
  session runs against a one-paragraph charter and 3–5 heuristics, prioritized
  by risk (auth/checkout/payment > recently changed > rest).
- **Staging only.** Refuse anything that looks like production. Never enter real
  PII/payment, never send real emails/SMS, never do destructive/irreversible
  actions without asking first.
- **Read context first.** Check any injected context + `qa-docs/PRD.md`,
  `User_Stories.md`, `Test_Cases.md` (explore the gaps), and existing
  `List_Feedback.md` (don't refile). Ask only for what's missing, in one message.
  Default time box: **45 minutes**.
- **Template + language selection is mandatory** before writing anything (ask
  both; see Template Selection Rule below). Session summaries are always Bahasa
  Indonesia.
- **Duplicate-data validation is mandatory for every CRUD feature** — try to
  create/update using an already-existing unique value and confirm it's
  rejected; test both the UI form and the API request when possible.
- **Broken access control** — whenever roles or record IDs exist, test IDOR
  (swap another user's record ID) and privilege escalation (hit admin routes as
  a lower role). Any success = Critical.
- **Report as you go.** Surface each confirmed bug live with repro steps,
  expected vs actual, evidence, and a reproducibility header (env URL, build,
  browser/viewport, account/role used). Write the file only once, at the
  approval gate.
- **Coverage log.** Keep a live Touched/Skipped list; the end summary's
  covered/not-covered is read off it. If a tool can't do something (e.g.
  `agent-browser` viewport resize), mark it Skipped (no-tool) — never claim it
  was tested.

### Cycle-Based Exploratory Testing (default operating model)

Run exploratory testing as **three sequential cycles**, each its own SBTM
session (own charter, own debrief), all sharing **one ledger** (defects +
coverage). **Default scope = Cycle 1 only; pause and get explicit approval
before each next cycle** — the user may stop after any cycle. Full workflow
lives in the `qa-exploratory-testing` skill.

- **Cycle 0 — Recon (quick, shared):** read `qa-docs/*`, map screens, record
  statuses, and endpoints/IDs from the Network tab; write charters; open the ledger.
- **Cycle 1 — UI / End-user (DEFAULT):** act as a real end user through the UI
  only (type/click/submit). **Source of truth for user-facing behavior** —
  ground-truth every claim against stored data / the record list, never the HTTP
  response. The ~90% UX/E2E budget lives here.
- **Cycle 2 — API / Contract & robustness (after approval):** "what does the UI
  hide?" For each validation the UI enforced in Cycle 1, send the request
  directly (bypassing the client) and check whether the **server** enforces it
  too; plus mass assignment, parameter tampering, hidden endpoints.
- **Cycle 3 — Security / Adversarial (after approval):** assume hostile — IDOR/
  BOLA, privilege escalation, XSS/CSRF, info disclosure. Cycle 2 = *trust but
  verify the contract*; Cycle 3 = *assume hostile*. IDOR/access-control lives here.

**Cross-cycle rules:**
- **UI-first is mandatory.** A finding from a direct API/`fetch` call is NOT a
  user-facing bug (the client may validate what the server doesn't). Verify
  through the UI first; label bypass findings as robustness (Cycle 2) or security
  (Cycle 3), never as a bug a normal user hits. See feedback-verify-validation-via-ui-not-api.
- **Dedup by linking/annotating — never silently skip.** A later cycle that
  re-hits a filed issue **annotates** the existing defect with the new layer of
  evidence (e.g. "UI blocks it; server accepts it when bypassed → root cause
  server-side") and **re-rates severity** — it does not refile and does not drop it.
- **Tag every defect** with found-in cycle (UI/API/SEC) and root-cause layer
  (client/server/both). **Gate between cycles:** debrief + coverage + approval.
- **Data hygiene:** track created records in the ledger; clean up via the app's
  delete, else flag exact IDs for admin cleanup at the end.

---

## List Feedback Templates

### Template Selection Rule (REQUIRED)

**Never generate a List Feedback immediately.** This rule applies whenever List Feedback output is triggered by:
- `/qa-agent-manual-testing`
- `/qa-exploratory-testing [url]`
- `/qa-agent-retest`
- Any explicit request to "build list feedback" or "buat list feedback"

**Always pause and ask first:**
> "Template mana yang ingin digunakan untuk List Feedback?"
> 1. List Feedback Kabayan Format
> 2. Standard Format

Wait for the user to select before generating anything. Generate strictly following the chosen template — no column mixing.

---

### Available Templates

#### 1. Standard Format

Simple English-language bug report. Columns:

| Column | Description |
|--------|-------------|
| Bug ID | `BUG-001`, `BUG-002`, … |
| TC ID | Linked test case ID, or `—` if from exploratory testing |
| Bug Title | Short English title |
| Severity | Critical / High / Medium / Low |
| Module | Feature area name |
| Steps to Reproduce | Numbered steps |
| Expected Behavior | What should happen |
| Actual Behavior | What actually happened |
| Status | Open (default) |
| Dev Notes | Left blank for developer |

---

#### 2. List Feedback Kabayan Format

Professional Indonesian-language bug report. Created per-project in a new Google Spreadsheet (user provides the target spreadsheet). Visual reference for colors and chip formatting: spreadsheet `1GiJ800TVEL4RxBnNX6aguvC8qw3jIiRa` ("List Feedback" sheet — Omni Bogor 2026).

**Defect ID format:** `<AppInitials>-<NN>` — first two letters of the app name, e.g. Aepsilon → `AE-01`, AE-02.

**Columns (A → J):**

| Col | Header | Rules |
|-----|--------|-------|
| A | Defect ID | `<AppInitials>-01`, `<AppInitials>-02`, … |
| B | Date | `DD-MMM-YYYY`, e.g. `12-Jul-2026` |
| C | Crede | Role (e.g. Administrator, Staff, Customer) or username + password used during testing |
| D | Fitur | Hierarchical path from top nav to action, e.g. `Menu A - Submenu A - Create`. Auth → `Auth` |
| E | Bug Description | Professional Indonesian. Include: observed behavior, expected behavior, steps to reproduce. Optional UX recommendation: starts with `Baiknya...`, ends with `(Jika memungkinkan).` |
| F | Priority | Dropdown: Low / Medium / High / Critical — use predefined color-coded chip from Omni Bogor reference |
| G | Status | Dropdown — use predefined color-coded chip from Omni Bogor reference (see lifecycle below) |
| H | QA | Always `Cahya` — use existing formatted chip |
| I | Attachment / Comment From QA | Screenshots (.png) or screen recordings (.mov) + technical notes in Indonesian. Include Network tab errors when available |
| J | Attachment / Comment From Dev | Left blank — filled by developer |

**Priority chip colors (from Omni Bogor reference):**
- Low → blue/teal
- Normal → yellow
- High → orange
- Critical → red

**Status lifecycle:**

| Status | Set by | Meaning |
|--------|--------|---------|
| Open | QA | Default for all new bugs |
| On Progress | Developer | Dev is investigating or fixing |
| Hold | QA or Dev | Blocked — awaiting clarification, or prior stage not yet complete |
| Ready to Test | Developer | Fix complete, ready for QA retest |
| Failed | QA | Retest done — bug still present |
| Done | QA | Retest passed — bug resolved |
| Ready for Deploy | Developer | Fix approved, awaiting deployment |

**Output location:** `qa-docs/List_Feedback.md` (markdown) or a new Google Spreadsheet tab when building in Sheets via gws CLI.

---

## Retest Approach (Google Spreadsheet-based bug tracking)

When the user asks to retest bugs tracked in a Google Spreadsheet, follow this approach exactly.

### Setup

1. **Read the spreadsheet first.** Use `gws sheets spreadsheets values get` on the "List Feedback" sheet to read all rows. Identify which rows have Status = "Ready to Test" — those are the bugs to retest.
2. **Note the row number** of each "Ready to Test" bug. The G column is always the Status column. Map bug ID → row number before touching the browser.
3. **Ask the user for credentials and the target URL** if not already provided.

### Retest Execution (per bug)

For each "Ready to Test" bug:
1. Read the bug description carefully — understand what was broken and what the expected behavior is.
2. Open `agent-browser`, navigate to the relevant module/page.
3. Reproduce the exact scenario described in the bug report.
4. Observe whether the bug still exists.
5. Immediately update the spreadsheet cell:
   - **Done** → bug is fixed, behavior matches expected
   - **Failed** → bug still present

### Spreadsheet Updates (CRITICAL RULES)

- **Always use targeted `batchUpdate`** — never download, edit, re-upload.
- Update only the specific Status cell (column G) for each bug.
- Batch multiple updates into a single `batchUpdate` call when possible.
- Command pattern:
  ```bash
  gws sheets spreadsheets values batchUpdate \
    --params '{"spreadsheetId": "<ID>"}' \
    --json '{"valueInputOption": "RAW", "data": [{"range": "List Feedback!G<row>", "values": [["Done"]]}]}'
  ```
- Confirm the update was applied before moving to the next bug.

### Common Browser Patterns

**El-dropdown action menus** (Element UI / Vue apps):
- Action buttons are often inside `.el-dropdown [role=button]` — click that to open the menu
- Menu items appear as `[role=menuitem]`
- Do not try to click menu items before the dropdown is open

**React controlled number inputs (spinbutton)**:
- Direct `.value =` assignment won't trigger React state update
- Use native setter: `Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype,'value').set.call(input, val); input.dispatchEvent(new Event('input',{bubbles:true}))`

**Cascading/multi-level dropdowns** (wilayah, region selectors):
- Each level must be fully selected before the next level unlocks
- Click the wrapper element (`generic[ref=eX] clickable`), not the inner combobox
- Wait for the next dropdown to populate before proceeding

**Session expiry mid-form**:
- If a form submission returns a server error or hangs, the session may have expired
- Navigate back to sign-in, re-enter credentials, re-select role (role dropdowns often reset after errors), then retry
- Never assume the form is still valid after a server error

### When the Server is Unstable

- If login hangs on "Loading" for >30 seconds, the backend API may be intermittently down
- `curl -s <url>` returning 200 does not mean API endpoints are healthy — the frontend shell loads independently
- Wait 1–2 minutes and retry login once before reporting the server as down
- Do not loop indefinitely — after 2 failed login attempts, pause and inform the user

### After All Retests

Report a summary to the user:
- Total bugs retested
- How many are Done vs Failed
- Any bugs skipped and why

Then confirm all spreadsheet updates were applied.
