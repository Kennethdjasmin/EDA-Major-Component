# README — KRA EDA Claude Code subprompts

Four standalone prompts decomposed from `opus_prompt_eda_notebook.md`, each for a fresh cloud Claude Code session against `EDA_Major_Component.ipynb`.

| # | File | What it does | Edits notebook? | xlsx sheet |
|---|------|--------------|-----------------|------------|
| 1 | `task1_rfm_migration.md` | Year-banded RFM migration (additive — original RFM untouched) | Yes (adds Section 10B cells) | `10_RFM_Migration` |
| 2 | `task2_verify_rfm_scatter.md` | Verify slides 33/34 vs notebook | **No** (report + `verification_task2_rfm.md`) | — |
| 3 | `task3_flagC.md` | Verify slide 25 (report) + UPCOMING/ON-TRACK heatmaps + `opp_C_full` + sheet | Yes (Parts B/C only) | `7C_Flag_C_Full` (+ patch `7C_Fleet_Overdue`) |
| 4 | `task4_island_ranking.md` | Export island ranking shift | Yes (adds export cell) | `3D_Island_Ranking` |

## Recommended run order

Run **in the same persistent cloud project**, one prompt per session, in this order so the in-place notebook edits accumulate:

1. **Task 1** → 3. **Task 3** → 4. **Task 4** → 2. **Task 2** (last — it is read-only and just audits).

Why this order: Tasks 1, 3, 4 each append one `add_sheet(...)` call to cell 205; running them sequentially in the same notebook means each sees the previous edits, and the idempotency guard (skip if the sheet name already appears in cell 205's source) prevents duplicates. After each task, the agent re-runs cell 205, so the workbook stays consistent.

> If your cloud sessions do **not** share a directory: after each task, **download the edited `EDA_Major_Component.ipynb`** and upload that (not the original) as the input to the next task. Otherwise edits from earlier tasks will be missing.

## Upload manifest

The notebook's loader (cell 6) reads files by **exact bare filename from the working-directory root**. Several expected names use **spaces/dashes, not underscores** — rename your uploads to match, or the load fails.

| Needed by | Expected filename (verbatim) |
|-----------|------------------------------|
| Tasks 1,3,4 | `Intern_SalesKRA_2017_202603.xlsx` |
| Task 3 | `Intern_Data_WH_202603.xlsx` |
| Task 3 | `Intern_Hist Comp Repl for RUL 202602.xlsx`  *(spaces)* |
| Task 3 | `Intern_Data_Populasi_202603.xlsx` |
| Task 4 | `Intern_Data_Cab_Site.xlsx` |
| Task 2 | *(none — uses stored notebook outputs + the .pptx)* |

Notes:
- Cell 6 loads ~13 files in one block, including `Intern_Data_WH_Historical_2023.xlsx` (not in your shared set) and several commodity files. A single missing/misnamed file makes that cell error. **Every build prompt therefore includes a preflight fallback**: if cell 6 fails on a file the task does not need, the agent loads only the required dataframes directly and continues — so you only strictly need the per-task files above.
- Simplest path if you have them all: upload the full data set with the exact cell-6 names, and the whole loader runs clean.

## Python deps

`pandas`, `numpy`, `matplotlib`, `seaborn`, `openpyxl` (all already used by the notebook), plus `python-pptx` for Tasks 2 & 3's slide reads (`pip install python-pptx`). **Prophet / statsmodels are not needed** — every prompt runs only the minimal prerequisite cell range (typically 0–53, plus a small section lead-in), never the Section-2/15 forecasting cells.

## Guarantees baked into each prompt

- **Verification = report, not embed** (Task 2 entirely; Task 3 Part A) — findings go to your chat + a `verification_*.md` file, never into the notebook.
- **Additive safety** — Task 1 never alters cells 173–177 or reassigns `rfm`/`seg_summary`; Task 4 never reassigns `yearly_mix`; Task 3 B/C never overwrite `opp_C`/`df_fleet_latest`.
- **Anti-hallucination** — no asserted number unless computed in-session or read from a stored output / the pptx; "I don't know" otherwise.
- **Exact urgency labels** for Flag C (`'OVERDUE'`, `'DUE SOON (0-500h)'`, `'UPCOMING (500-2000h)'`, `'OK'`) so filters don't silently return empty.
- **xlsx idempotency + anchor** — new sheets inserted before the `# Data dictionary` line in cell 205, skipped if already present.
