---
name: qa-bug-reporting
description: Record confirmed defects from any QA test directly in the shared project List Feedback, including safe concurrent updates.
license: MIT
---

Read `../../AGENTS.md` and the [List Feedback templates](../../references/qa-standards.md#list-feedback-templates). This skill applies to every QA specialist and Sena. The agent that finds a bug writes its own report entry; it does not hand the writing to `qa-manual`.

1. Confirm the defect and its scope. Recheck an automation failure before filing it; distinguish a product defect from flaky tests, broken fixtures, or environment failures. For API-only or nonfunctional findings, describe the endpoint or measured condition accurately and do not claim a user-facing UI failure without UI evidence. Keep unconfirmed observations in the test notes, not as `Open` bugs.
2. Find the project's existing List Feedback and follow its template, language, IDs, and user edits. If no report or explicit template choice exists, ask once: Kabayan or Standard. Use `qa-docs/List_Feedback.md` for a new local report; use a spreadsheet only when the user requested it. Sena should pass the chosen template to parallel specialists.
3. Prepare the finding with reproducible steps, expected and observed behavior, environment/build, role or account alias, source test ID or API route/measurement, severity (`Critical / High / Medium / Low`), and available evidence. New defects start as `Open`. Never include passwords or tokens. Apply `../humanizer/SKILL.md` to prose only; preserve exact technical values and evidence.
4. Before changing the shared report, ensure the project `qa-docs/` folder exists and acquire an exclusive lock by creating `qa-docs/.list-feedback.lock` with an atomic directory-create operation (`mkdir`). If it already exists, retry for up to 30 seconds; never overwrite the report or remove a lock owned by another running agent. If the lock cannot be obtained, keep the finding in the run notes and report the blocked write. Release your lock with `rmdir` after saving, including on errors.
5. While holding the lock, read the **latest** report or sheet, check for the same underlying defect, and allocate the next ID from the current contents. Add new evidence to an existing defect instead of filing a duplicate. Append or update only the affected entry, preserve unrelated rows and comments, refresh summary counts, then read back the saved entry before releasing the lock. For Sheets, use targeted append/update calls while the shared project lock is held. A local lock coordinates only agents using the same project folder; if they do not share that folder, do not write concurrently to the same report.

Tell Sena and the user which defect ID was written, its severity, report path, and any finding that could not yet be filed. Sena reviews cross-agent coverage and consistency; it is not required to transcribe the specialists' bugs.
