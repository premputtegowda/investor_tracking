---
name: refresh-investor-report
description: Regenerate the investor tracking dashboard from a new Cash Flow Portal Excel export. Produces index.html (per-sponsor progress bars), changes.html (new investors, partial payments, newly funded since the prior snapshot), and totals_by_sponsor.csv. Use when the user asks to refresh, rebuild, regenerate, or publish the dashboard, or when a new Cash_Flow_Portal_Investments_MM_DD_YYYY.xlsx file appears in the project root.
---

# Refresh investor report

Run the `investor_tracking_echarts.ipynb` notebook to regenerate `index.html`, `changes.html`, and `totals_by_sponsor.csv` from the latest Cash Flow Portal export, then optionally commit and push so GitHub Pages picks it up.

## Inputs

- **Required**: a fresh `Cash_Flow_Portal_Investments_MM_DD_YYYY.xlsx` file in the project root (`/Users/premputtegowda/Documents/CFP/`). The notebook auto-picks the most-recently-dated file.
- **Optional**: a `PREV_DATE_OVERRIDE` value if the user wants the `changes.html` "since" comparison to point at a specific prior date instead of the second-most-recent file.

## Steps

### 1. Identify the latest export
Run `ls -t Cash_Flow_Portal_Investments_*.xlsx | head -3` and confirm with the user which file is "current". If the newest file is not what they expected, stop and ask.

### 2. Set the comparison date (if requested)
The notebook has a `PREV_DATE_OVERRIDE` constant near the top of cell `imports-cell` (around line ~80). Default is a date in `MM_DD_YYYY` format (e.g. `'05_27_2026'`) or `None` for auto.

- If the user gave a comparison date: edit the constant in the notebook to match (use `NotebookEdit` on cell `imports-cell`, or `Edit` if surgical).
- Verify the matching `Cash_Flow_Portal_Investments_<DATE>.xlsx` exists. If not, warn the user — the notebook will print `WARN: override file ... not found — falling back to auto` and use the second-most-recent file instead.

### 3. Run the notebook
Execute all cells in `investor_tracking_echarts.ipynb` from top to bottom. Use whichever execution method is available (Jupyter kernel, `papermill`, or have the user run it themselves and paste back output).

Cells, in order, produce:
1. `imports-cell` — picks `SRC` (current) and `SRC_PREV` (comparison) xlsx files.
2. `load-cell` — loads + cleans the data, builds `df`, `df_bar`, `summary`, `sponsor_order`.
3. `new-cell` — computes three DataFrames: `new_investors`, `partial_payments`, `newly_funded`. Investors who are both brand-new AND made a payment between the dates go into the payment section with an `is_new=True` flag.
4. `changes-cell` — writes `changes.html` (three tables; NEW badge for `is_new` rows).
5. `chart-cell` — writes `index.html` (per-sponsor target progress bars + investor pizza tracker).
6. `totals-cell` — writes `totals_by_sponsor.csv`.

### 4. Verify outputs
- `index.html` exists and reports `Saved index.html (N investors across M sponsor sections)`.
- `changes.html` exists and reports `Saved changes.html (new=..., partial=..., newly funded=...)`.
- `totals_by_sponsor.csv` exists and reports `Saved totals_by_sponsor.csv (13 sponsors + Total row)`.
- Optionally open `index.html` in a browser for a quick visual check.

### 5. Commit and push (with user confirmation)
**Always confirm with the user before committing or pushing**, since this publishes to GitHub Pages.

```bash
git add index.html changes.html totals_by_sponsor.csv investor_tracking_echarts.ipynb
git commit -m "Refresh dashboard"
git push origin main
```

Match the existing commit style — recent commits in this repo use short messages like `"Refresh dashboard"` (see `git log --oneline -5`). After push, the GitHub Pages site at https://premputtegowda.github.io/investor_tracking/ rebuilds in ~1–2 minutes. A hard browser refresh (`Cmd + Shift + R`) is usually needed to clear cache.

To verify deploy: `gh run list --limit 3` shows the pages build status.

## Notes

- Excel filenames use `MM_DD_YYYY` (e.g. `05_15_2026.xlsx`). One file in this repo uses `05_19_2025` — that's an off-by-one-year typo, not a real 2025 file.
- Per-sponsor targets are hardcoded in the `TARGETS` dict inside `chart-cell` of the notebook. To change a target, edit the dict and re-run.
- The dashboard hides fully-funded investors from per-sponsor rows but keeps them in sponsor-level totals.
- `partial_payments` is detected by comparing `Funded amount` between the two xlsx files: if it went up but is still less than `Invested amount`, the investor went into the partial bucket.
- The skill does **not** push the notebook's own changes unless the user explicitly requests it (the notebook itself only changes when its constants or cells are edited).
