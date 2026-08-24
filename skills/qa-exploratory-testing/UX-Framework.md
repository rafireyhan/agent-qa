# UX / Manual Exploratory Testing Framework & Task List

> A reusable playbook for the **UX / manual** side of exploratory testing.
> This is the **complement** to the security/API pass — run both for full
> coverage, weighted ~90% UX / E2E, ~10% security.

---

## 1. Why this exists — the two lenses

We test the same app but ask different questions. Neither alone is complete.

| | Security / API lens | **UX / Manual lens (this doc)** |
|---|---|---|
| Core question | "How can this be **abused**?" | "Does this **work & feel right** for a real user?" |
| Layer watched | Network, responses, DB, auth codes | The **rendered page** + the user's workflow |
| Account used | Cross-role, low-priv attacker | The intended user, normal flow |
| Path | Off-happy-path / crafted requests | Happy path + realistic mistakes |
| Catches | Access control, XSS, injection, info leaks | Layout, workflow, validation UX, data persistence, states, wording |
| Misses | UX friction, visual defects | Auth holes, injection, info leaks |
| Budget | ~10% (spot-check) | ~90% (main event) |

**Rule of thumb:** if a bug is invisible unless you open DevTools/Network → it's
the security lens. If a bug is something a user would *see, feel, or get stuck
on* → it's this lens.

---

## 2. Before the session (setup checklist)

- [ ] Confirm **staging only** (never prod). Note env URL + build/date.
- [ ] Get **all accounts/roles** you'll use (admin, operator, applicant, etc.).
- [ ] Write a **one-paragraph charter**: which module, what user goal, what risks.
- [ ] Prepare **test data**: a valid record, an edge record, a file to upload (valid + oversized + wrong-type).
- [ ] Open **browser DevTools** (Console tab) — even in a UX pass, console errors reveal broken JS.
- [ ] Decide a **time box** (default 45–60 min/module) and keep a live Touched/Skipped log.
- [ ] Have a place for **screenshots** (bugs only) and one-line notes.

---

## 3. UX heuristics — memory hook: **"FLOW-VISED"**

Run each feature through these lenses:

- **F**ields — every input: empty, too long, wrong format, special chars, whitespace, paste.
- **L**ists — sort, search, filter, pagination, empty state.
- **O**perations — full CRUD: Create → Read → **Edit (does data reload?)** → Delete → (Restore).
- **W**orkflow — multi-step flows, Back/Cancel/Batal, where buttons *actually* go.
- **V**isual — alignment, overflow, broken/missing images, icons, colors, responsive.
- **I**nteraction cues — cursor, hover, disabled/loading states, affordances.
- **S**tates — empty / loading / error / success / no-data / no-image.
- **E**rror messages — friendly? localized? specific? (vs raw/technical/wrong language).
- **D**ata consistency — dropdowns reflect current data; language switch changes ALL text; dashboards show real data.

---

## 4. The Task Lists (check every applicable item per feature)

Each item shows **what "pass" looks like** and, where relevant, the **common
failure pattern** to watch for.

### A. Lists / Tables (every index page)
- [ ] **Sort order sane** — newest/most-relevant first (or a clear default). *Common failure: new records appended to the **bottom**, so users can't find what they just created.*
- [ ] Column sort (asc/desc) works on each sortable header.
- [ ] Row actions (Edit/Delete/Toggle) present & correct per row.
- [ ] Long text wraps/truncates cleanly (no layout break).
- [ ] Empty list shows a friendly "no data" state (not a blank/broken table).

### B. Search / Filter / Pagination
- [ ] **Search matches what the user expects** (e.g., by Name). *Common failure: search only matches one hidden field, so an obvious name query returns "not found".*
- [ ] Search: partial, uppercase/lowercase, trailing spaces, special chars.
- [ ] Filter: single, multiple, **reset**, and **invalid combinations**.
- [ ] **Cross-field filter validation** (e.g., End date ≥ Start date). *Common failure: end-before-start accepted with no validation.*
- [ ] Pagination: first/last/next/prev; change page size.
- [ ] **Delete the only row on page 2** → app returns to a valid page (not an empty view). *Common failure: user is left staring at an empty page instead of page 1.*

### C. Forms — Create & Edit
- [ ] Create happy-path saves and shows success feedback.
- [ ] **Open Edit → all previously saved data is loaded into the fields.** *Common failure: a saved field (description, ID number, etc.) comes back empty on Edit.*
- [ ] Edit → change → save → re-open → change persisted.
- [ ] Required fields enforced with clear messages.
- [ ] Cancel/Back discards changes and **returns to the right place** (see F).
- [ ] Rich-text (WYSIWYG): every toolbar feature renders correctly on the display page (see the separate Rich-Text testing method).

### D. Input Validation (per field)
- [ ] Empty / required.
- [ ] **Max length** — very long input handled gracefully (message, not a crash/raw error). *Common failure: over-long input surfaces a raw server/error alert instead of a friendly validation message.*
- [ ] **Every field that needs a limit has one** — don't assume; test each. *Common failure: one field silently accepts unlimited characters while its siblings are capped.*
- [ ] Format (email, phone, URL, date), whitespace-only, emoji/unicode, paste, autofill.

### E. File Upload
- [ ] Accepted types only (reject exe/js/wrong types).
- [ ] **Size limit enforced at save, not just visually** — try a file **over the stated limit**. *Common failure: the UI hint states a limit but an oversized file still saves; or the client blocks it while the server does not.*
- [ ] Filename with spaces/unicode/long name.
- [ ] Clear feedback on reject (why it failed) and on success (preview).

### F. Navigation & Buttons
- [ ] **Every button goes where its label promises** — Save, Cancel, Back, breadcrumb. *Common failure: Cancel/Back navigates to an unrelated menu instead of the previous page.*
- [ ] Breadcrumbs reflect the real location.
- [ ] Browser Back/Forward don't break state.
- [ ] Deep-link / refresh mid-flow behaves.

### G. Data Consistency
- [ ] Dropdowns/pickers show **only currently valid** options — or label stale ones. *Common failure: a deactivated/archived item still appears as a selectable option with no "inactive" label.*
- [ ] Parent/child relationships stay in sync.
- [ ] A change in module A reflects in module B where it should.

### H. States (empty / loading / error / no-image)
- [ ] **No-image / no-data renders a proper placeholder, not a broken image.** *Common failure: a missing thumbnail shows a broken-image icon inside a large empty banner.*
- [ ] Loading indicators appear for slow actions (no frozen UI).
- [ ] Error state is readable and recoverable (see I).
- [ ] Zero-results and first-run states look intentional.

### I. Feedback & Notifications
- [ ] **Every action gives feedback** — Create/Edit/Delete/Toggle all confirm success. *Common failure: destructive/toggle actions succeed silently with no toast, so the user isn't sure it worked.*
- [ ] Error messages are **friendly, specific, and localized** — not raw/technical/wrong-language. *Common failure: a message leaks an internal field name (e.g. "the slug has already been taken") in the wrong language.*
- [ ] Confirm dialogs for destructive actions.

### J. Localization / i18n
- [ ] Switch language → **ALL** visible text switches (titles, descriptions, **button labels**, placeholders, messages). *Common failure: most text translates but button labels stay in the default language.*
- [ ] No hardcoded strings leaking the wrong language.
- [ ] Date/number formats match locale.

### K. Affordances & Interaction Cues
- [ ] Clickable things **look** clickable — **pointer cursor** on links/buttons. *Common failure: a "read more" link keeps the default arrow cursor.*
- [ ] Hover/focus states visible; disabled looks disabled.
- [ ] Buttons shown match what the user can actually do (hide/disable un-permitted actions). *Common failure: a view-only role still sees Add/Edit/Delete buttons.*

### L. Data Display (dashboards / widgets)
- [ ] Widgets show **real** data that matches the underlying records. *Common failure: a summary widget shows 0 while matching records clearly exist.*
- [ ] Totals/counts reconcile with the list they summarize.
- [ ] Charts/exports match the on-screen data.

### M. Responsive (needs real device or resize)
- [ ] Mobile (375px) and tablet (768px): no overflow, tap targets usable, menus reachable.
- [ ] Landscape/portrait.
- *(If your tool can't resize the viewport, mark **Skipped (no-tool)** — never claim it was tested.)*

---

## 5. Per-module quick pass (~15 min, run on every CRUD screen)

1. Open list → check **sort default, search-by-name, empty state**.
2. Create with valid data → **success feedback?**
3. Create with bad data → **each field validated with a friendly message?**
4. Open Edit of the record you just made → **is all data loaded back?**
5. Change + Save + reopen → **persisted?**
6. Test **Cancel/Back** → **lands where expected?**
7. Any dropdown → **only valid/active options?**
8. Any image/upload → **oversize + no-image placeholder?**
9. Switch language → **everything translated, including buttons?**
10. Glance at **DevTools Console** → any red errors while doing the above?

---

## 6. Reporting UX bugs (make them actionable)

- **Title:** short, user-facing ("Cancel on Edit navigates to the wrong menu").
- **Steps to reproduce**, **Expected vs Actual**, **screenshot** (bug state only).
- **Severity for UX:** blocking workflow > data loss/persistence > confusing/misleading > cosmetic.
- Add a **suggestion** when it's a preference, not a hard defect — style: *"Baiknya … (Jika memungkinkan)."*
- Note **env + account + browser/viewport** used.

---

## 7. Catalogue of recurring UX bug patterns

The manual pass tends to cluster into a small set of repeatable patterns —
check these first on any CRUD-heavy app:

1. **List sorting** — new data not surfaced (bottom instead of top).
2. **Save-time validation gaps** — oversized files / over-long text accepted or crashing with a raw error.
3. **Create → Edit data loss** — saved data not reloaded into the edit form.
4. **Navigation mis-wiring** — Cancel/Back goes to the wrong page.
5. **Stale reference data** — deactivated items still offered in pickers.
6. **Missing/empty-state handling** — broken image instead of a placeholder.
7. **Interaction affordances** — wrong cursor, buttons that don't match permissions.
8. **Localization gaps** — some text (esp. buttons) not translated.
9. **Dashboard/data-display mismatches** — widget shows 0 while data exists.
10. **Pagination edge cases** — delete last row on a page → empty view.

---

## 8. How the two passes fit together

- Run the **UX pass** (this doc, ~90%) and the **security pass** (~10%) as
  **separate charters** — different mindset, different account, different layer.
- Expect **little overlap** between them. Small overlap = the split is working,
  not a gap.
- Combine both lists for the real coverage picture.
