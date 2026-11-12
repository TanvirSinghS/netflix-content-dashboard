# Netflix Content Dashboard — Power BI Portfolio Project

An interactive Power BI report on the Netflix titles catalogue (8,807 titles): what the library is made of,
when content was added, where it comes from, and how long it runs.

## What's inside

| Folder | Contents |
|---|---|
| `Dashboard/` | The Power BI project (PBIP): `projectPBI-TANVIR.pbip` + `projectPBI-TANVIR.Report` + `projectPBI-TANVIR.SemanticModel`. Open the `.pbip` file. |
| `Data/` | The three cleaned CSVs the model reads. `netflix_cleaned.csv` is one row per title (23 columns); `netflix_countries.csv` and `netflix_genres.csv` are bridge tables with one row per country / genre tag. |
| `Notebook/` | `netflix-data-portfolio-sql-pbi.ipynb` — the SQL/analysis notebook used to clean and profile the data. |
| `Docs/` | `Netflix_Dashboard_Guide.md` (audit, Power Query fixes, every visual's fields and exact X/Y/W/H, DAX with checks, insights, QA), `Netflix_PBIP_Handover.md` (build state and what is left to do), `Netflix_Portfolio_WriteUp.md` (resume/LinkedIn copy), `Netflix_Insights_TextBox.txt` (the report's insight text). |
| `Theme/` | `Netflix_Dark_Theme.json` — the report theme, also registered inside the project. |
| `Assets/` | `before-desktop-screenshot.jpeg` — the starting hand-built page, for a before/after. |

## Open the dashboard

1. Install **Power BI Desktop** (free) with the Power BI Project (PBIP) preview enabled — it is on by default in current builds.
2. Open `Dashboard/projectPBI-TANVIR.pbip`.
3. Desktop asks permission for the CSV file paths → **Connect**.
4. **Home → Refresh**. The *Total titles* card should read **8,807** (Movie 6,131 / TV 2,676 → **69.6 %** movies, average movie runtime **99.6 min**).

## If the data doesn't load on your machine

The queries hold an absolute path that points at this folder on the author's machine. The fix is one dialog:

**Home → Transform data → Data source settings → Change Source** → set each of the three queries
(`netflix_cleaned`, `netflix_genres`, `netflix_countries`) to the matching file in `Data/`, then **Home → Refresh**.

## What the report contains

- **3 pages / 28 visuals:** `01 Overview`, `02 Deep Dive`, `03 Insights and Method`.
- **4 slicers** (type, year added, audience, primary country), synced across pages so one control drives the whole report.
- **DAX measures:** `% Movie of Titles`, `Avg Movie Runtime (min)`.
- **Country / genre / director visuals set to "None" as filter sources** so they don't double-count the bridge tables.
- **Dark theme** (`#0B0B0B` page, `#E50914` accent), 1280×720 canvas.

## Key numbers (all from the data, not estimated)

| Metric | Value |
|---|---|
| Titles | 8,807 |
| Movies / TV shows | 6,131 (69.6 %) / 2,676 (30.4 %) |
| Peak year of additions | 2019 — 2,016 titles |
| Largest country (tags) | United States — 3,690 |
| Largest genre | International Movies — 2,752 |
| Audience | Adults 45.5 %, Teens 30.1 %, Kids 10.3 %, Older Kids 13.1 %, Unrated 1.0 % |
| Movie runtime | mean 99.6 min, median 98 min, 74 % between 60 and 120 min |
