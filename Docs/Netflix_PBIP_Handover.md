# Netflix Dashboard — PBIP build handover

**Project:** `PBIPPRO/projectPBI-TANVIR.pbip`
**Built:** 3 pages, 28 visuals, 1 custom theme, 2 measures, 2 calculated columns, 2 relationships
**Backup of the pre-build state:** `output/pbip-backup-before-completion.zip`
**Generator (re-runnable):** `output/build_dashboard.py`

---

## 1. Open it

1. Close Power BI Desktop completely (it may still hold the old folder path).
2. **File → Open** → `PBIPPRO\projectPBI-TANVIR.pbip`.
3. If a yellow bar says the data source can't be found, see §5.
4. **Home → Refresh** once. Expected: completes with no errors. The three KPI numbers must read **8,807 / 6,131 / 2,676**.

> If Desktop refuses to open with a validation error, jump to §6 — the error text names the file, and removing that one visual folder gets you back in.

---

## 2. What was built (visual map)

Canvas 1280 × 720 on all pages. 16 px page margin, 16 px gutters, 180 px slicer rail, body from x=212 to x=1264.

### Page 1 — "01 Overview" (14 visuals)

| Visual | Type | X | Y | W | H |
|---|---|---|---|---|---|
| Header banner | textbox | 16 | 12 | 1248 | 64 |
| Total titles (red accent card) | card | 16 | 88 | 236.8 | 92 |
| Movies | card | 268.8 | 88 | 236.8 | 92 |
| TV shows | card | 521.6 | 88 | 236.8 | 92 |
| Movies share % | card (measure) | 774.4 | 88 | 236.8 | 92 |
| Avg movie runtime | card (measure) | 1027.2 | 88 | 236.8 | 92 |
| Type / Year / Audience / Country slicers | slicer | 16 | 196 / 326 / 456 / 586 | 180 | 118 |
| Titles added per year | lineChart | 212 | 196 | 700 | 238 |
| Movies vs TV | donutChart | 928 | 196 | 336 | 238 |
| Top 10 countries | clusteredBarChart | 212 | 450 | 700 | 254 |
| Top 10 genres | clusteredBarChart | 928 | 450 | 336 | 254 |

### Page 2 — "02 Deep Dive" (9 visuals)

| Visual | Type | X | Y | W | H |
|---|---|---|---|---|---|
| Header banner | textbox | 16 | 12 | 1248 | 64 |
| Same four slicers | slicer | 16 | 196 / 326 / 456 / 586 | 180 | 118 |
| Titles added by month | clusteredColumnChart | 212 | 196 | 700 | 238 |
| Audience mix by type (100%) | hundredPercentStackedBarChart | 928 | 196 | 336 | 238 |
| Year × month heatmap | pivotTable (matrix) | 212 | 450 | 700 | 254 |
| Ratings by title count | treemap | 928 | 450 | 336 | 254 |

### Page 3 — "03 Insights and Method" (5 visuals)

| Visual | Type | X | Y | W | H |
|---|---|---|---|---|---|
| Header banner | textbox | 16 | 12 | 1248 | 64 |
| Key Insights (7 lines) | textbox | 16 | 88 | 608 | 336 |
| Runtime distribution | clusteredColumnChart | 16 | 440 | 608 | 264 |
| Top 10 directors | tableEx | 640 | 88 | 624 | 300 |
| Method / caveats | textbox | 640 | 404 | 624 | 300 |

### Field bindings that matter

- KPI cards: **Count of show_id** (not `Min(show_id)`), with a `type` categorical filter on the Movies and TV cards.
- Movie % card: measure `Movie %`. Runtime card: measure `Avg Movie Runtime`.
- Countries bar: `netflix_countries[Countries]` + TopN 10 + exclude "Unknown".
- Genres bar: `netflix_genres[genre]` + TopN 10.
- Heatmap: Rows `year_added`, Columns `Month Name`, Values Count of `show_id`.
- Directors table: `director_raw` + Count of `show_id`, TopN 10, exclude "Unknown".
- Time visuals all filter out blank `year_added` / `month_added` so the 10 undated titles don't create a blank category.

---

## 3. Model changes (all inside Power Query / TMDL — no CSV was touched)

| Change | Why it was needed |
|---|---|
| `PromoteHeaders` added to **netflix_genres** and **netflix_countries** | Their M never promoted headers, so the literal header row (`show_id`,`genre` / `show_id`,`country`) was being loaded as a fake title-genre / title-country row. Genre rows are now the correct 19,323, country rows 10,843. |
| `Column1`/`Column2` → `show_id`/`genre` in netflix_genres | Readable field list, and the relationship can be written explicitly. |
| `Countries` data category changed `County` → `Country` | Map visuals / tooltips can now geocode it correctly. |
| `""` → `null` for `date_added`, `year_added`, `month_added`, `movie_minutes`, `tv_seasons`, `release_year`, `rating` | The CSVs store blanks as empty strings. `Int64.Type` / `type date` on an empty string errors on refresh — this is why the model could only ever load from cache before. |
| `date_added` → Date, `release_year` / `month_added` / `movie_minutes` / `tv_seasons` → Whole number | `AVERAGE(movie_minutes)` and the runtime histogram are impossible while it is text. `movie_minutes` was also set to *sum*, which would have produced nonsense. |
| All numeric columns set to **Do not summarize** | Stops accidental `SUM(year_added)` = 17 million. |
| Two calculated columns: `Month Name` = `FORMAT(DATE(2000,[month_added],1),"MMM")`, sorted by `month_added`; `Runtime Band` = "00-59" / "60-90" / "90-120" / "120+" | Correct Jan→Dec order in the matrix, and a real 4-bin histogram without adding a bin group. |
| Two relationships added to `model.tmdl` | They were **not** in the saved model: `netflix_genres[show_id] → netflix_cleaned[show_id]` and `netflix_countries[show_id] → netflix_cleaned[show_id]`, one-to-many, single direction. Without them no slicer can filter the genre/country visuals. |
| Two measures | `Movie %` = `DIVIDE(CALCULATE(COUNTROWS(netflix_cleaned), netflix_cleaned[type]="Movie"), COUNTROWS(netflix_cleaned))`, `Avg Movie Runtime` = `AVERAGE(netflix_cleaned[movie_minutes])`. |

**Verified expectations after refresh:** Movie % = 69.6 %, Avg Movie Runtime = 99.6 (formats: `0.0%`, `0.0`).

---

## 4. Theme

- Theme file: `projectPBI-TANVIR.Report/StaticResources/RegisteredResources/Netflix Dark.json`
- Registered in `report.json` as `customTheme` (`RegisteredResources`), base theme CY24SU10 kept underneath.
- Palette: `#E50914` red first, then greys/white. Page `#0B0B0B`, cards `#141414`, borders `#262626` radius 8, Segoe UI text classes.
- A standalone copy for importing into other files: `output/Netflix_Dark_Theme.json` (same file).

Because it is registered, it is **already applied** — you do not need View → Themes → Browse.

---

## 5. Where the data comes from

The three queries already read the CSVs that ship inside this project — no `C:\Users\Jasnit\Downloads` path remains:

```
C:\Users\artic\downloads\projbi\PBIPPRO\netflix_cleaned.csv
C:\Users\artic\downloads\projbi\PBIPPRO\netflix_genres.csv
C:\Users\artic\downloads\projbi\PBIPPRO\netflix_countries.csv
```

- First open: Desktop asks permission for these file paths → **Connect**.
- **If you move the `PBIPPRO` folder** (or submit it from a different machine), fix it in **Home → Transform data → Data source settings → Change Source** for each of the three queries, then **Home → Refresh**. Do it through the UI rather than editing M by hand, so Power BI re-validates the path.
- A stale local cache caused the earlier `DataModelLoadFailed` / "Datatype conversion failed" error. It was deleted (`.pbi\cache.abf`), so the model now rebuilds from the queries on open.

---

## 6. If Power BI reports a validation error on open

Every page and visual is its own JSON file, so the failure is isolated and the error text names the file/visual.

1. Note the visual name in the error (20 hex characters).
2. Close Desktop. Delete that one folder: `projectPBI-TANVIR.Report\definition\pages\<page>\visuals\<name>\`.
3. Reopen. Everything else loads; tell me the visual type and I will regenerate it correctly.
4. Worst case, restore `output/pbip-backup-before-completion.zip` over `PBIPPRO/` to get back to the pre-build state.

Most likely candidates for a schema complaint, in order: the three `textbox` visuals (their paragraph JSON is the least standardised part of PBIR), and the `hundredPercentStackedBarChart` type name.

---

## 7. Already wired in the files (nothing to click)

- **Slicer sync.** All four slicers exist on pages 1 and 2 with `syncGroup` set to `Sync type`, `Sync year_added`, `Sync audience`, `Sync primary_country` (`fieldChanges` and `filterChanges` both true). View → Sync slicers will show Sync + Visible ticked for both pages, so one control drives the whole report.
- **Interactions.** In page 1's `visualInteractions`, the Top 10 countries bar and the Top 10 genres bar are `NoFilter` against all 12 other data visuals. On page 3 the directors table is `NoFilter` against the runtime histogram. They read bridge tables (one row per country/genre/director tag), so as filter sources they would double-count — now they no longer filter anything.

## 8. Left for you manually (2 jobs, both need Power BI Desktop)

1. **Open, refresh, look at it.** **File → Open** → `PBIPPRO\projectPBI-TANVIR.pbip` → **Connect** if asked about the CSV paths → **Home → Refresh**. The Total titles card must read **8,807**. One visual pass is still needed for text fitting in the two text boxes, which I cannot render here.
2. **Export for the portfolio:** **File → Export → Export to PDF** (all 3 pages), plus **View → Screenshot** or a snipping tool at 100 % zoom for LinkedIn. To publish instead: **Home → Publish → My workspace**, then **File → Embed report** for a shareable link.

---

## 9. QA results

| Check | Result |
|---|---|
| JSON validity, all 28 visuals + 3 pages + report.json + theme | Pass (parsed) |
| Every field referenced by a visual exists in the TMDL | Pass (automated check) |
| No leftover references to `Column1` / `Column2` | Pass (0 found) |
| All visuals inside 1280×720 | Pass |
| No overlapping visuals on any page | Pass (automated rectangle check) |
| Page order / active page | 01 → 02 → 03, page 1 active |
| Theme registered and name matches file | Pass |
| Relationships exist in model.tmdl | Pass (2, both one-to-many into netflix_cleaned) |
| Header rows removed from bridge tables | Pass (`PromoteHeaders` in all three queries) |
| Blank strings converted before typing | Pass |
| KPI expectations vs CSV | 8,807 / 6,131 / 2,676 / 69.6 % / 99.6 min |
| Slicers synced | Pass (4 groups × 2 pages, each group binds one field) |
| Bridge-table visuals set to None | Pass (24 + 1 interaction pairs, all `NoFilter`) |
| `report.json` theme package registered once | Pass (1 `RegisteredResources` + base `SharedResources`) |

**Not verifiable from here:** actual rendering. I cannot run Power BI Desktop, so colours, font fitting and text overflow in the text boxes need one visual pass on your machine. Everything structural was checked programmatically.

---

## 10. Numbers used in the text boxes (all from the CSVs, not estimated)

- Titles 8,807; movies 6,131 (69.62 %); TV 2,676 (30.38 %).
- Additions by year peak 2019 = 2,016 (1,424 movies + 592 TV); 2020 = 1,879 (−6.8 %); 2021 = 1,498 to 25 Sep.
- Months: Jul 827, Dec 813, Sep 770, Oct 760, Apr 764, Mar 742, Jan 738, Jun 728, Nov 705, May 632, Feb 563.
- Countries (tags): US 3,690, India 1,046, UK 806, Canada 445, France 393, Japan 318, Spain 232, South Korea 231, Germany 226, Mexico 169; Unknown 831; total tags 10,843 across 1,315 multi-country titles.
- Genres: International Movies 2,752, Dramas 2,427, Comedies 1,674, International TV Shows 1,351, Documentaries 869; 6,787 multi-genre titles.
- Audience: Adults 4,009 (45.5 %), Teens 2,650 (30.1 %), Older Kids 1,150 (13.1 %), Kids 908 (10.3 %), Unrated 90 (1.0 %); Kids = 17.4 % of TV vs 7.2 % of movies.
- Runtime: n = 6,131, mean 99.6, median 98, 74 % in 60–120, 30 % under 90.
- Unknowns: director 2,634, cast 825, primary_country 833; 10 titles without a date, 7 without a rating.
