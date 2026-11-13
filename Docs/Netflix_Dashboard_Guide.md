# Netflix Content Analysis — Complete Power BI Build & Design Guide

**Project:** `project-tani.pbix` · **Data:** `netflix_cleaned.csv` (8,807 titles), `netflix_genres.csv` (19,323 rows), `netflix_countries.csv` (10,843 rows)
**Canvas:** 1280 × 720 px (16:9) · **Pages:** 3 · **Theme:** `output/Netflix_Dark_Theme.json`

---

> **Status update:** the dashboard has since been built directly in the PBIP project (`PBIPPRO/projectPBI-TANVIR.pbip`) — 3 pages, 28 visuals, theme applied, model corrected, slicers synced across pages and bridge-table visuals set to None. Only opening/refreshing and exporting the PDF are left, and both need Power BI Desktop. See `output/Netflix_PBIP_Handover.md` §7–8. The manual steps below remain valid as a from-scratch reference or if you rebuild in a plain .pbix.

## 0. What I checked before writing this

| Check | Result |
|---|---|
| Saved report | 1 page named `Page 1`, 1280×720, **7 visuals**, no slicers, default blue theme |
| KPI cards in saved file | All three bound to `Min(netflix_cleaned[show_id])` — wrong aggregation |
| Country bar in saved file | Uses `netflix_countries.Column2` — not the `country` column |
| Row counts | 8,807 unique `show_id` in `netflix_cleaned`; 1 row per title confirmed |
| Bridge coverage | All 8,807 titles exist in both bridge tables (no orphan rows, no blanks in relationships) |
| Unknowns | 2,634 unknown director · 825 unknown cast · 831 unknown country · 0 unknown audience |
| Missing dates | 10 titles with blank `date_added` / `year_added` / `month_added` |
| Date range | 2008-01-01 → 2021-09-25 (2021 is partial) |
| Multi-tag titles | 6,787 titles carry >1 genre, 1,315 carry >1 country |

Your saved `.pbix` is behind what you described (it has no slicers, no month chart, no genre bar, no treemap, no histogram, no directors table). Everything below builds the finished report from the data up, so it covers both.

**How to use this guide:** work top to bottom. Section 3 (theme) and Section 4 (two measures) should be done first, then build page 1, duplicate it for page 2, and add page 3.

---

## 1. AUDIT — what's missing, wrong or misleading

### 1.1 Wrong / misleading (fix these first)

| # | Problem | Why it matters | Fix |
|---|---|---|---|
| A1 | All three KPI cards are bound to **`Min(show_id)`** in the saved file | `show_id` is a text ID (`s1`, `s2`…), so Min gives a meaningless string. A card must use a count. | Section 5.3 — rebind every card to **Count of `show_id`**, with a type filter. |
| A2 | Country bar uses **`netflix_countries.Column2`** | If `Column2` is a duplicate/index column, every bar shows the same total — exactly the "country counts repeating the total" symptom you fixed earlier. | Section 2.1 — rename/repair the column in Power Query, then rebuild the bar on `netflix_countries[country]`. |
| A3 | `"Unknown"` is treated as a country (831 titles) and appears as the **3rd biggest bar** | Reads as if Netflix has an "Unknown" country. Misleading headline. | Exclude `Unknown` from the country bar (page 1) and state the 831 in the footnote. |
| A4 | Genre and country counts add up far beyond 8,807 | Genre rows = 19,323; country rows = 10,843. Anyone adding bars will notice the totals don't reconcile. | Keep the bars, but the visual titles say "titles tagged", and the footnote explains it. Never use these bars for a % of catalogue. |
| A5 | 2021 line falls off a cliff | The data stops on **25 September 2021** (1,498 titles vs 1,879 in 2020). Without a label it looks like a collapse in output. | Chart title carries "(2021 partial)" + footnote. Never quote 2021 as a full year. |
| A6 | Default blue theme, white page | The single biggest reason it looks like "default Power BI". | Section 3 — import the dark theme; page background #0B0B0B. |
| A7 | Layout is off-grid: cards start at **x=0**, card heights are 156.85 / 166.30 / 149.29, chart tops are 95 vs 118, bottom chart widths are 550 vs 548 | This is what makes a dashboard feel amateur, even with good charts. | Sections 5–6 give exact X/Y/W/H on a 12 px gutter grid. |
| A8 | Auto titles ("Count of show_id by type", "Count of show_id by Countries") | Auto titles describe the query, not the insight. | Every visual gets a written title (given below). |
| A9 | `Total values in Dataset` wording on card 1 | Not a business label. | Rename to `TOTAL TITLES`. |
| A10 | 7 titles have a blank rating; 90 titles are `Unrated` | A treemap with an unnamed slice looks broken. | Filter the treemap to `rating is not blank`; mention Unrated in the footnote. |

### 1.2 Missing visuals (from your list and the saved file)

- Year × month heatmap matrix — **missing** (Section 6.3)
- Movie % card — **missing** (Section 4)
- Average movie runtime card — in the saved file only as an unbuilt idea; built in Section 5.3
- Titles added by month (column chart) — **missing** (Section 6.1)
- Top 10 genres bar — **missing** (Section 5.6)
- Movie runtime histogram — **missing** (Section 6.5)
- Rating treemap — **missing** (Section 6.4)
- Top 10 directors table — **missing** (Section 6.6)
- Slicer panel (type, year, audience, country) — **missing** (Section 5.4)
- Header banner, footer/source note, page 2, insights page — **missing** (Sections 5.1–5.2, 6, 8)

### 1.3 Model issues to confirm before designing

1. In **Model view**, both relationships must be: `netflix_genres` **Many** → `netflix_cleaned` **One**, and `netflix_countries` **Many** → `netflix_cleaned` **One**, cross-filter **Single**.
   If the arrow points the other way, the bridge tables become filters that can't be filtered — this is the usual cause of "counts repeat the same total".
2. With single-direction filtering, **a country slicer built on `netflix_countries[country]` will not filter your other visuals.** That is not a bug — filters only flow *from* `netflix_cleaned` *to* the bridge tables. Fix: the slicer uses `netflix_cleaned[primary_country]` (Section 5.4).
3. `show_id` uniqueness confirmed — plain **Count** is correct; do **not** use "Count (Distinct)" here, it hides errors.

---

## 2. Power Query and data-type fixes (do not edit the CSVs)

### 2.1 Fix the `Column2` problem in `netflix_countries`

1. **Home → Transform data → Transform data** (opens Power Query Editor).
2. In **Queries** (left pane), click `netflix_countries`.
3. Look at the columns: you must see exactly two — `show_id` and `country`.
4. If the second column is called `Column2`, right-click its header → **Rename** → type `country`.
5. In **Applied Steps** (right pane), check for steps named `Removed Columns`, `Duplicated Column`, `Added Index` — delete any step that renamed or duplicated the country column by clicking the **X** next to the step name.
6. Confirm types in the header icons: `show_id` = **Text** (ABC), `country` = **Text**.
7. Repeat step 3–6 for `netflix_genres` (must be `show_id`, `genre`).
8. **Home → Close & Apply**.

### 2.2 Confirm data types in `netflix_cleaned`

Click `netflix_cleaned`, then check the icon left of each column header. Highlight a column → **Home → Data Type** dropdown if it's wrong.

| Column | Type |
|---|---|
| `show_id`, `type`, `title`, `director_raw`, `cast_raw`, `countries_raw`, `rating`, `duration`, `genres_raw`, `description`, `decade`, `audience`, `primary_country`, `primary_genre` | Text |
| `date_added` | Date |
| `release_year`, `year_added`, `month_added`, `years_to_netflix`, `movie_minutes`, `tv_seasons` | Whole Number |

Then **Home → Close & Apply**. Do **not** filter rows, do **not** replace `"Unknown"`, do **not** add date columns here — all of that is handled in DAX/visuals so the raw data stays honest.

> Expected: 8,807 rows for `netflix_cleaned`. `movie_minutes` is blank for all 2,676 TV shows and `tv_seasons` blank for all 6,131 movies — that is correct, not an error.

---

## 3. THEME + PAGE SETUP (the big visual win — 6 minutes)

### 3.1 Import the dark theme

1. **View → Themes → Browse for themes**.
2. Select `output/Netflix_Dark_Theme.json`.
3. Click **Open**. The page turns near-black, visuals get #141414 cards, rounded borders, red accents.
4. Open any visual → **Format your visual** → if it still shows old colours, click **Reset to default** at the bottom of the Format pane.
5. The theme hides the small header icons to keep screenshots clean. To get the **…** menu back on one visual: **Format → General → Header icons → On**.

What the theme does automatically: page background #0B0B0B, card background #141414, border #2A2A2A with 10 px radius, white titles, grey axis labels, subtle gridlines, hidden visual header icons, red-first data colour sequence, donut labels as % of total, 8–11 pt font scale.

> If Power BI refuses the file it will say "invalid theme file". Tell me the exact error text — but the JSON is valid and uses only standard theme properties, so this should not happen.

### 3.2 Canvas and page

1. Click empty canvas → **Format page** (paint-roller icon in the Format pane) → **Canvas settings**.
2. **Type:** Custom · **Width:** 1280 · **Height:** 720. (If it already shows 16:9 / 1280×720, leave it.)
3. **Canvas background** → **Color** `#0B0B0B`, **Transparency** 0%.
4. **Wallpaper** → Off.
5. Rename the page tab: double-click `Page 1` → type `01 Overview`.

### 3.3 Turn on the alignment helpers

1. **View** tab → tick **Gridlines** and **Snap to grid**.
2. **View → Page view → Fit to page** so you can see the whole canvas while positioning.
3. Use **Format → Align** (with several visuals selected) to check edges.

### 3.4 The design system (use these everywhere)

| Token | Value | Used for |
|---|---|---|
| Page | `#0B0B0B` | canvas |
| Card | `#141414` | visual + shape backgrounds |
| Rail | `#1A1A1A` | slicer panel |
| Border | `#2A2A2A` | 1 px borders |
| Gridline | `#242424` | chart gridlines |
| Accent | `#E50914` | Netflix red: Movie %, highlights, one bar per chart |
| Text primary | `#F5F5F1` / `#FFFFFF` | titles, values |
| Text secondary | `#B3B3B3` | labels, legend |
| Text muted | `#8A8A8A` | axis values |
| Font | Segoe UI / Segoe UI Semibold | everywhere (safe on university PCs) |
| Sizes | 24 title · 12 visual title · 9 labels · 8 axes | consistent scale |
| Gutter | 12 px between visuals, 16 px page margin | grid |

Colour rule: **red is a spotlight, not a palette.** One red element per visual (the Movie slice, the highlighted column, the top country bar); everything else grey.

---

## 4. DAX — exactly two measures

### 4.1 Measure 1: Movie % (needed for the Movie % card)

1. **Modeling → New measure**.
2. In the formula bar, paste (all on one line is fine):

```DAX
Movie % = DIVIDE ( CALCULATE ( COUNTROWS ( netflix_cleaned ), netflix_cleaned[type] = "Movie" ), COUNTROWS ( netflix_cleaned ) )
```

3. Press **Enter**, then in **Measure tools**: **Format** → **Percentage**, **Decimal places** → `1`.
4. **Check:** with no slicers, a card using it must read **69.6%**. Manual cross-check: 6,131 ÷ 8,807 = 0.6962.

### 4.2 Measure 2: Average movie runtime (needed for the runtime card)

1. **Modeling → New measure**.
2. Paste:

```DAX
Avg Movie Runtime = AVERAGE ( netflix_cleaned[movie_minutes] )
```

3. **Measure tools → Format → Custom** and type: `0.0 "min"`
4. **Check:** with no slicers the card must read **99.6 min**. The CSV average is 99.6 (min 3, max 312).

Why these two and no more: everything else (counts, top-N, heatmap values) is a **built-in aggregation** on a unique ID — a measure would add nothing. Keep the model light.

> Expected behaviour: if the Type slicer is set to **TV Show**, `Movie %` shows 0.0% and `Avg Movie Runtime` goes blank. Both are correct — there are no movies selected. Mention it in your demo, don't "fix" it.

---

## 5. PAGE 1 — "01 Overview"

### 5.1 Header banner (5 objects)

1. **Insert → Shapes → Rectangle**. With it selected: **Format → Shape → Fill** `#141414`, **Line → Off**.
2. **Format → General → Properties → Position/Size**: X `0`, Y `0`, W `1280`, H `72`.
3. **Arrange → Send to Back**.
4. **Insert → Shapes → Rectangle** again: Fill `#E50914`, Line Off, X `0`, Y `69`, W `1280`, H `3`, Send to Back. (This 3 px red line is the whole "brand" trick.)
5. **Insert → Text box** → type `NETFLIX CONTENT ANALYSIS`. Select the text: **Home → Font** Segoe UI Semibold, **Size 24**, colour `#FFFFFF`. Position X `24`, Y `12`, W `900`, H `34`. Turn **Format → General → Effects → Background → Off**.
6. Second text box: `Titles added 2008 – 25 Sep 2021 · 8,807 titles · Movies, TV shows, countries, genres · Python/pandas + Power BI`. Size 9, colour `#B3B3B3`. X `24`, Y `44`, W `900`, H `18`.
7. **View → Selection**: rename the three shapes to `bg_header`, `accent_line`, `txt_title` and click the **lock** icon on all four so you can't drag them by accident.

### 5.2 Slicer rail

1. **Insert → Shapes → Rectangle**: Fill `#1A1A1A`, Line `#2A2A2A` 1 px, **X 16, Y 88, W 180, H 616**, Send to Back.
2. Lock it. All four slicers sit inside it.

### 5.3 KPI card row (5 cards, 12 px gutters)

Build one, format it once, then copy-paste it four times (Power BI keeps all formatting) and change fields.

**Card 1 — TOTAL TITLES**
1. In **Visualizations**, click **Card**.
2. Drag `netflix_cleaned[show_id]` into **Fields** — it appears as `Count of show_id`.
3. **Right-click** `Count of show_id` in the Fields well → **Rename for this visual** → `TOTAL TITLES`.
4. **Format → General → Properties**: X `208`, Y `88`, W `201.6`, H `92`.
5. **Format → Visual → Callout value**: Font **Segoe UI Semibold**, Size **32**, Colour `#FFFFFF`; **Label**: Size `9`, Colour `#B3B3B3`.
6. Title off (the label acts as the title).

**Card 2 — MOVIES** (copy card 1: Ctrl+C, Ctrl+V)
- X `421.6`, Y `88`, W `201.6`, H `92`.
- Rename the field label `MOVIES`.
- **Filters pane → Filters on this visual** → drag `netflix_cleaned[type]` → select only **Movie**.

**Card 3 — TV SHOWS**
- X `635.2`, Y `88`, same size. Type filter = **TV Show**.

**Card 4 — MOVIE %**
- X `848.8`, Y `88`, same size.
- Fields: drag the **`Movie %`** measure. Rename for this visual: `MOVIE % OF CATALOGUE`.
- Callout colour `#E50914` (this is the accent card), size 30 so it doesn't clip.

**Card 5 — AVG MOVIE RUNTIME**
- X `1062.4`, Y `88`, same size.
- Fields: drag **`Avg Movie Runtime`**, rename `AVG MOVIE RUNTIME` (or `AVG RUNTIME` at size 8).

### 5.4 Slicers (left rail, top to bottom)

Create each with **Visualizations → Slicer**, then set position.

| Slicer | Field | Style | X | Y | W | H |
|---|---|---|---|---|---|---|
| `TYPE` | `netflix_cleaned[type]` | **Tile** | 24 | 100 | 164 | 76 |
| `YEAR ADDED` | `netflix_cleaned[year_added]` | **Between** (slider) | 24 | 188 | 164 | 104 |
| `AUDIENCE` | `netflix_cleaned[audience]` | **Vertical list** | 24 | 300 | 164 | 132 |
| `COUNTRY` | `netflix_cleaned[primary_country]` | **Dropdown** + search | 24 | 440 | 164 | 148 |

Exact settings per slicer:
1. Add the slicer, drag the field in.
2. **Format → Slicer settings → Options → Style** → choose the style from the table.
3. **Format → General → Title** → On, text as in the table, Size `9`, colour `#B3B3B3`.
4. **TYPE:** **Format → Slicer settings → Selection → Single select** → Off (multi-select by default).
5. **YEAR ADDED:** Style **Between** → **Format → Slicer settings → Slider** → On, **Show value labels** → On. Leave the range unset (= all years). *Do not* lock the max at 2021 in a way that hides the partial-year caveat.
6. **COUNTRY:** **Format → Slicer settings → Options → Search** → On; **Selection → Multi-select with Ctrl** → On. (123 countries — search is essential.)
7. Slicer text is 9 pt via the theme; if any slicer shows a white box, **Format → General → Background** → `#141414`, **Border** → `#2A2A2A`, **Rounded corners** `8`.
8. Add a small text box in the rail: X `24`, Y `596`, W `164`, H `100`, size 8, colour `#8A8A8A`:
   `Ctrl+click to multi-select.` / `Right-click → Clear all to reset.` / `Country uses the primary (first-listed) country.`

### 5.5 Line chart — titles added per year

1. **Visualizations → Line chart**.
2. **X-axis:** `netflix_cleaned[year_added]` · **Y-axis:** `Count of show_id` · **Legend:** `netflix_cleaned[type]`.
3. **Format → General → Properties**: X `208`, Y `192`, W `696`, H `244`.
4. **Format → General → Title** → text: `Titles added per year, by type (2021 partial)`, Size `11`, colour `#FFFFFF`, Align left.
5. **Format → Visual → X axis**: Type **Categorical**, font `8`, colour `#B3B3B3`, Title off.
6. **Format → Visual → Y axis**: Range **Min 0**, font `8`, colour `#8A8A8A`, Title off; **Gridlines** On, colour `#242424`, 1 px.
7. **Format → Visual → Lines → Markers** On, size 4; **Stroke width** 3.
8. **Format → Visual → Colors**: Movie `#E50914`, TV Show `#B3B3B3` (deliberate contrast — the film line is the story).
9. **Data labels:** Off (11 years × 2 series would be noise). Tooltips carry the numbers.
10. **"…" → Sort axis → year_added → Sort ascending**.

### 5.6 Donut — Movies vs TV shows

1. **Visualizations → Donut chart**.
2. **Legend:** `netflix_cleaned[type]` · **Values:** `Count of show_id`.
3. Position X `916`, Y `192`, W `348`, H `244`.
4. Title: `Movies vs TV shows` · Size `11`.
5. **Format → Visual → Slices → Inner radius** `62%`; **Colors**: Movie `#E50914`, TV Show `#B3B3B3`.
6. **Detail labels → Label style** `Percent of total`, font `9`, colour `#FFFFFF`; **Label position** Inside.
7. **Legend** Top centre, font `8`, colour `#B3B3B3`.
8. **Check:** 69.62% / 30.38% (6,131 / 2,676).

### 5.7 Top 10 countries bar

1. **Visualizations → Clustered bar chart**.
2. **Y-axis:** `netflix_countries[country]` — **not** `Column2`. **X-axis:** `Count of show_id` from `netflix_countries`.
3. Position X `208`, Y `448`, W `696`, H `244`.
4. **Filters pane → Filters on this visual**:
   - `country` → **Filter type: Basic** → tick everything except **Unknown** (use the search box in the filter list), or choose **Advanced** → `is not` → `Unknown`.
   - `country` → **Filter type: Top N** → **Show items: Top 10** → **By value** → drag `Count of show_id` into the By value well → **Apply filter**.
5. **"…" → Sort axis → country → Sort descending** (largest at the top of a bar chart).
6. **Format → Visual → Bars → Color** `#B3B3B3`; then **click the top bar once** (selects that data point) → **Format → Bars → Color** → `#E50914`. Only the leader is red.
7. **Data labels** On, font `8`, colour `#F5F5F1`, position Outside end; **X axis** Off; **Y axis** font `8`, Title off.
8. Title: `Top 10 countries by titles` · Size `11`.

> Expected: United States 3,690 · India 1,046 · United Kingdom 806 · Canada 445 · France 393 · Japan 318 · Spain 232 · South Korea 231 · Germany 226 · Mexico 169.
> The whole country table has 10,843 rows for 8,807 titles, because 1,315 titles carry more than one country. Each bar therefore counts title–country tags, which is why the footnote explains it and why these bars must never be added up into a catalogue share.

### 5.8 Top 10 genres bar

1. **Clustered bar chart**. **Y-axis:** `netflix_genres[genre]` · **X-axis:** `Count of show_id` (from `netflix_genres`).
2. Position X `916`, Y `448`, W `348`, H `244`.
3. Filter: `genre` → Top N 10 → by `Count of show_id`.
4. Sort descending; bars `#6E6E6E`, top bar `#E50914`; data labels on (font 8), X axis off, Y axis font 8.
5. Title: `Top 10 genres by titles`.
6. Expected leaders: International Movies 2,752 · Dramas 2,427 · Comedies 1,674 · International TV Shows 1,351 · Documentaries 869 · Action & Adventure 859 · TV Dramas 763 · Independent Movies 756 · Children & Family Movies 641 · Romantic Movies 616.

### 5.9 Footnote (both pages)

1. **Insert → Text box**, X `208`, Y `700`, W `1056`, H `16`, font `8`, colour `#8A8A8A`:
   `2021 is partial (data ends 25 Sep 2021) · 831 titles have no listed country; 2,634 no listed director · Country and genre charts count one title once per tag, so bars can sum above 8,807 · Source: Netflix titles dataset, cleaned in Python/pandas`

### 5.10 Page 1 checklist before moving on

- [ ] All 5 cards read 8,807 / 6,131 / 2,676 / 69.6% / 99.6 min
- [ ] No card still uses `Min(show_id)`
- [ ] Country bar shows US ≈ 3.7K and no "Unknown" bar
- [ ] All 4 slicers change the KPI row and both charts
- [ ] Nothing is flush against the canvas edge; left column all 208; right column all ends at 1264

---

## 6. PAGE 2 — "02 Deep Dive"

**Fastest route:** right-click the `01 Overview` tab → **Duplicate Page** → rename `02 Deep Dive`. Keep the header, rail, slicers and footnote; delete only the 5 cards, line chart, donut and the two bars. Then the slicers are already in the right place.

Then:
1. Edit the header: title `CATALOGUE DEEP DIVE`, subtitle `When titles were added, who they're for, and how long movies run`.
2. **View → Sync slicers** → tick **Sync** and **Visible** for `TYPE`, `YEAR ADDED`, `AUDIENCE`, `COUNTRY` on **both** pages.

### 6.1 Column chart — titles added by month

1. **Visualizations → Clustered column chart**.
2. **X-axis:** `netflix_cleaned[Month Name]` · **Y-axis:** `Count of show_id`.
3. Position X `208`, Y `88`, W `696`, H `200`.
4. Title: `Titles added by month (all years)`.
5. **Format → Visual → Columns → Color** `#B3B3B3`; select the **July** column (single click) → colour `#E50914`.
6. **Data labels** On, font `8`, colour `#F5F5F1` — 12 columns, so labels help.
7. **X axis** font `8`, Title off; **Y axis** font `8`, Title off, gridlines `#242424`.
8. **Check:** Jul 827 · Dec 813 · Sep 770 · Oct 760 · Apr 764 · Mar 742 · Jan 738 · Jun 728 · Nov 705 · May 632.
9. **Check:** Aug 755 · Feb 563 complete the twelve months.
10. `Month Name` must sort Jan→Dec: **Data pane → `Month Name` → Column tools → Sort by column → `month_added`** (you already did this — verify it's still set).

### 6.2 100% stacked bar — audience mix by type

1. **Visualizations → 100% Stacked Bar Chart**.
2. **Y-axis:** `netflix_cleaned[type]` · **X-axis:** `Count of show_id` · **Legend:** `netflix_cleaned[audience]`.
3. Position X `916`, Y `88`, W `348`, H `200`.
4. Title: `Audience mix by type (% of titles)`.
5. **Data labels** On → **Values → Percent of total** On, **Value** Off; font `8`, colour `#FFFFFF`, position Inside centre.
6. Legend Top centre, font `8`.
7. **Colors** in this order: Adults `#E50914`, Teens `#F5F5F1`, Older Kids `#9E9E9E`, Kids `#5C5C5C`, Unrated `#2E2E2E` — a red-to-grey ramp, readable in greyscale too.
8. **Check:** Adults 45.5% of all titles; TV Shows are visibly more kid-heavy (17.4% Kids) than Movies (7.2% Kids).

### 6.3 Year × month heatmap matrix

1. **Visualizations → Matrix**.
2. **Rows:** `netflix_cleaned[year_added]` · **Columns:** `netflix_cleaned[Month Name]` · **Values:** `Count of show_id`.
3. Position X `208`, Y `300`, W `696`, H `200`.
4. Title: `Titles added: year × month heatmap`.
5. **Format → Visual → Row subtotals** → Off · **Column subtotals** → Off (this removes the "Total" row/column that would swamp the colour scale).
6. **Format → Visual → Cell elements → Background color** → click the **fx** icon:
   - **Format style:** Gradient · **What field should we base this on?** `Count of show_id`
   - **Minimum:** colour `#1B1B1B`, value 0 (or Lowest)
   - **Maximum:** colour `#E50914`, value **260** (a fixed maximum stops 2019/2020 from washing out every other year; July 2021 peaks at 257)
   - **Apply** → **OK**.
7. **Font** for values: size `8`, colour `#FFFFFF`; **column headers** `8`, colour `#B3B3B3`; **row headers** `8`, colour `#E5E5E5`.
8. **Format → Visual → Grid**: horizontal and vertical both Off (cells should read as a heatmap, not a spreadsheet).
9. **Check the scale:** 2019-11 = 255 and 2020-01 = 205 are the hot cells; 2008–2013 are almost black (1–24 titles/year). The 2021 row stops hot at September — that's the partial-year story in one line.

### 6.4 Rating treemap

1. **Visualizations → Treemap**.
2. **Category:** `netflix_cleaned[rating]` · **Values:** `Count of show_id`.
3. Position X `916`, Y `300`, W `348`, H `200`.
4. **Filters on this visual:** `rating` → **is not blank** (7 titles have no rating).
5. Title: `Titles by rating`.
6. **Format → Visual → Data labels** On, font `9`, colour `#FFFFFF`.
7. Colour: keep theme default so red lands on the biggest group. Expected: TV-MA 3,207 · TV-14 2,160 · TV-PG 863 · R 799 · PG-13 490 · TV-Y7 334 · TV-Y 307 · PG 287 · TV-G 220 · NR 80 · G 41 · TV-Y7-FV 6 · NC-17 3 · UR 3.
8. The 3 tiny tiles (NC-17, UR, TV-Y7-FV) will show no label — that's fine, tooltips cover them. Turn labels off if you prefer a cleaner look.

### 6.5 Movie runtime histogram (needs one bin group — 30 seconds)

**Create the bins:**
1. **Data pane** → expand `netflix_cleaned` → right-click the **`movie_minutes`** field → **New group**.
2. **Group type:** Bin · **Bin size:** `10` · **Name:** `Runtime (min, 10-min bins)` → **OK**.
   (New column appears in the model; the CSV is untouched.)

**Build it:**
3. **Visualizations → Clustered column chart**.
4. **X-axis:** `Runtime (min, 10-min bins)` · **Y-axis:** `Count of show_id`.
5. Position X `208`, Y `512`, W `524`, H `180`.
6. **Filters on this visual:** `type` **is** `Movie` (so the blank TV minutes never appear).
7. Title: `Movie runtime distribution (10-min bins)`.
8. **Format → Visual → Columns → Color** `#B3B3B3`; **X axis** Title → On, text `Runtime (minutes)`, font `8`; **Y axis** Off; **data labels** Off (shape matters, not exact counts).
9. **Check:** the mass sits at 90–110 minutes; average 99.6, median 98; 74% of films run 60–120 min.

### 6.6 Top 10 directors table

1. **Visualizations → Table**.
2. **Columns:** `netflix_cleaned[director_raw]`, then `Count of show_id`.
3. Position X `744`, Y `512`, W `520`, H `180`.
4. **Filters on this visual:** `director_raw` → **does not contain** `Unknown`; then `director_raw` → **Top N 10** → by `Count of show_id`.
5. Title: `Top 10 directors by titles`.
6. **"…" → Sort by → Count of show_id → Sort descending**.
7. Rename the value column: double-click its header → `Titles`.
8. Format: header font `8`, colour `#B3B3B3`; values `8`, colour `#F5F5F1`; gridlines horizontal `#242424`, vertical off; **Totals** Off.
9. Expected leaders: Rajiv Chilaka 19 · Suhas Kadav 16 · Marcus Raboy 16 · Jay Karas 14 · Cathy Garcia-Molina 13 · Youssef Chahine 12 · Martin Scorsese 12 · Jay Chapman 12 · Steven Spielberg 11 · Don Michael Paul 10.
10. Note in your demo: rows with co-directors ("Robert Cullen, José Luis Ucha") appear as one combined string; 2,634 titles have no director recorded and are excluded here.

### 6.7 Page 2 footnote
Reuse the page-1 footnote text; change the first phrase to `2021 is partial (data ends 25 Sep 2021) · Heatmap scale capped at 260 titles · ...`

---

## 7. INTERACTIVITY

### 7.1 What the slicers do

| Slicer | Field | Affects | Notes |
|---|---|---|---|
| TYPE | `netflix_cleaned[type]` | every visual on pages 1–2 | Movie % → 0% and runtime → blank when only TV is selected (correct) |
| YEAR ADDED | `netflix_cleaned[year_added]` | every visual | Using 2021 alone gives ~1,498 titles: partial year |
| AUDIENCE | `netflix_cleaned[audience]` | every visual | 90 Unrated titles make that slice tiny — expected |
| COUNTRY | `netflix_cleaned[primary_country]` | every visual | Primary = first-listed country; a US/India co-production counts once, under US |

**Why the country slicer uses `primary_country`, not `netflix_countries[country]`:** the relationship is single-direction (`cleaned → countries`). A slicer on the country table cannot push a filter back up to `netflix_cleaned`, so it would look like it does nothing. `primary_country` lives on the fact table, filters everything, and avoids double counting. The bridge table is still used — for the chart, where multi-country titles are the point.

### 7.2 Edit interactions — exact settings

Select a visual, then **Format tab → Edit interactions**. Little icons appear on every other visual: **Filter** (funnel), **Highlight** (bar chart), **None** (circle-slash).

| Source visual (click it first) | Target visuals | Setting | Why |
|---|---|---|---|
| Country bar | all others | **None** | Single-direction model: its selection can't reach `netflix_cleaned`; leaving it on "Filter" makes the dashboard look broken. Use the COUNTRY slicer instead. |
| Genre bar | all others | **None** | Same reason (`cleaned → genres` is one-way) |
| Directors table | all others | **None** | Clicking a name shouldn't redefine the whole page |
| Donut | all others | **Filter** (default) | Clicking "Movie" is a great demo — all cards and charts collapse to films |
| Line chart | all others | **Filter** (default) | Click a year → everything follows |
| Month column | all others | **Filter** (default) | Clicking July shows the July catalogue |
| Heatmap matrix | all others | **Filter** (default) | Click a cell = one month of one year |
| Treemap | all others | **Filter** (default) | Click TV-MA → the whole page becomes mature titles |
| Cards | all others | **None** | A KPI card is an output, not a control |
| Runtime histogram | all others | **None** | A bin click would filter by an exact 10-minute range — confusing |

### 7.3 Test pass (2 minutes, do it after every layout change)

1. Click the donut's Movie slice → cards should read 6,131 / 6,131 / 0 / 100.0% / 99.6 min. Click empty canvas area to clear.
2. Click 2019 on the line → cards drop to 2,016 / 1,424 / 592; the heatmap's 2019 row stays visible and the other rows grey out.
3. Choose **TV Show** in TYPE → ramp bar disappears/greys, runtime card blanks. Clear.
4. Pick **India** in COUNTRY → country bar does **not** change (it's a bridge-table visual) but the cards, line, month, heatmap, treemap do. That asymmetry is the documented trade-off, not a bug.
5. Set YEAR ADDED min 2020 → the month chart shows 2020+2021 combined and Sep–Dec is visibly thin (partial 2021).
6. Go to page 2 → confirm the slicers carried their selections (**sync** from 6.0).

---

## 8. KEY INSIGHTS

Build page 3: right-click a page tab → **Duplicate Page** → rename `03 Insights & Method` → delete all visuals and slicers, keep header + red line. Then:

1. **Insert → Shapes → Rectangle**: X `16`, Y `88`, W `624`, H `600`, Fill `#141414`, Line `#2A2A2A`, radius 12. Send to Back.
2. **Insert → Text box**: X `40`, Y `104`, W `576`, H `568`. Paste the six bullets from `output/Netflix_Insights_TextBox.txt`. Format: Segoe UI, size `11`, colour `#E5E5E5`, line spacing 1.4, bullet colour default.
3. Second shape: X `656`, Y `88`, W `608`, H `600`, same fill/border. Second text box: X `680`, Y `104`, W `560`, H `568` with the method notes (bottom of this section).
4. Header: title `KEY INSIGHTS & METHOD`, subtitle `Full-catalogue figures — these text panels are static and do not follow the slicers`.

### Where each number comes from (read it off your own visuals, don't type my numbers blindly)

| Insight | Read it from | Value you should see |
|---|---|---|
| Movies vs TV split | Donut + cards MOVIES / TV SHOWS / MOVIE % | 6,131 vs 2,676 · 69.6% / 30.4% |
| Growth and peak year | Line chart — highest point, and its tooltip | 2019 peak = 2,016 (1,424 movies + 592 TV); 2020 = 1,879 |
| 2021 is partial | Line chart's last point + footnote | 1,498 titles, last date 25 Sep 2021 |
| Strongest months | Month column chart | Jul 827, Dec 813, Sep 770; 2021's own peak = Jul 257 |
| Top countries | Country bar (excludes Unknown) | US 3,690 · India 1,046 · UK 806 |
| Unknown country count | Footnote / filter card | 831 titles |
| Top genres | Genre bar | International Movies 2,752, Dramas 2,427, Comedies 1,674 |
| Audience mix | 100% stacked bar + audience slicer | Adults 45.5%, Teens 30.1%, Older Kids 13.1%, Kids 10.3%, Unrated 1.0% |
| Runtime | AVG MOVIE RUNTIME card + histogram | 99.6 min average, 98 median, 74% between 60–120 |

### Method panel copy (right column)

```
SOURCE
Netflix titles dataset (Kaggle), 8,807 unique titles, added 2008 – 25 Sep 2021. Three tables: one row per title, one row per genre tag, one row per country tag.

CLEANING (Python/pandas)
Malformed quotes in the type column fixed; year_added, month_added, years_to_netflix and decade derived from date_added and release_year; duration split into movie_minutes and tv_seasons; audience mapped from rating; primary_country and primary_genre = first listed value; missing director / cast / country filled with "Unknown".

CAVEATS
· 10 titles have no date_added and are excluded from all time visuals (still counted in totals).
· 2,634 titles have no director, 825 no cast, 831 no country.
· Genre table = 19,323 rows and country table = 10,843 rows: titles with several tags appear several times, so these bars must never be summed into a catalogue share.
· 2021 is partial (ends 25 Sep), so year totals for 2021 and month totals for Sep–Dec understate reality.
· 7 titles have a blank rating and are excluded from the treemap.

TOOLS
Python/pandas (cleaning, aggregation) → Power BI Desktop (star-schema model, two DAX measures, 3-page dark theme report).
```

---

## 9. FINAL QA CHECKLIST

**Model & data**
1. `netflix_cleaned` = 8,807 rows; both bridge tables relate on `show_id`, **Many to one**, single direction.
2. No KPI card uses `Min(show_id)`; every count reads `Count of show_id`.
3. `date_added` = Date; `year_added`, `month_added`, `release_year`, `movie_minutes`, `tv_seasons` = Whole Number (check the summation icon: Σ, not text).
4. `Month Name` sorted by `month_added` (Column tools → Sort by column).
5. Country bar uses `netflix_countries[country]`; `Column2` is gone.
6. `Runtime (min, 10-min bins)` group exists and is only used in the histogram.

**Content**
7. Both measures return 69.6% and 99.6 min with no filters.
8. Every visual has a written title; no "Count of show_id by…" titles remain.
9. 2021 caveat appears in at least 2 places (chart title + footnote).
10. Unknown handling: excluded from country bar and directors table; counted in the footnote.
11. Treemap has no blank slice.
12. No visual claims a % built from the genre or country bridge tables.

**Design**
13. Canvas 1280×720; page background `#0B0B0B`; all cards `#141414` with `#2A2A2A` borders.
14. Left column x=208, right column ends at 1264, gutter 12 px — every visual matches the tables above.
15. One red element per visual maximum; nothing uses the old blue.
16. Fonts only Segoe UI / Segoe UI Semibold; no visual below 8 pt.
17. Header and rail are locked in the Selection pane; nothing overflows the canvas.

**Interactivity**
18. All four slicers change every visual on pages 1–2; slicers are synced between pages.
19. Interactions match the table in 7.2 (country/genre bars and directors table = None).
20. Clicking the donut, line, month, heatmap and treemap filters the page; clicking empty space clears.

**Delivery**
21. **Save:** **File → Save as** → `Netflix_Content_Dashboard.pbix`. The imported theme is stored *inside* the .pbix, so the file is self-contained (keep the JSON in `output/` for re-use).
22. **Export PDF:** **File → Export → Export to PDF** → choose **All pages** → **Export**. Check the dark background survived.
23. **Screenshots:** **View → Page view → Fit to page**, collapse the Filters and Visualizations panes, then `Win + Shift + S`. Take one shot per page at 100% zoom.
24. **Publish (optional):** **Home → Publish** → choose a workspace. University tenants often need Pro and block publishing — if it fails, the PDF + screenshots are enough for a portfolio; say so rather than fighting it.
25. Keep the notebook and CSVs untouched (they're your audit trail); everything new lives in `output/`.

---

## 10. Summary — what I did, what I changed, what's left for you

**Built for you (files in `output/`)**
- `Netflix_Dark_Theme.json` — importable dark theme: black canvas, #141414 cards with 10 px rounded borders, red-first palette, Segoe UI scale, subtle gridlines, %-of-total donut labels, styled slicers/table/matrix.
- `Netflix_Dashboard_Guide.md` — this document: audit, Power Query fixes, both measures with checks, every visual's fields and exact X/Y/W/H, interactions, insights, QA, publishing.
- `Netflix_Insights_TextBox.txt` — the six bullets, ready to paste into the text box.
- `Netflix_Portfolio_WriteUp.md` — resume bullet, LinkedIn blurb, project paragraph.

**Changed or improved on my own (and why)**
- **Split into 3 pages** instead of one crowded page: overview (KPIs + trends + catalogue mix), deep dive (timing, audience, runtime, ratings, directors), insights & method. The 9 requested visuals plus header, rail and footnotes simply don't fit legibly in 1280×720.
- **Country slicer moved to `primary_country`** — the only version that actually filters the page, given your single-direction relationships.
- **Country bar excludes "Unknown"** and keeps multi-country counting; labelling explains both.
- **Heatmap colour scale capped at 260** so 2019/2020 don't black out every other year.
- **Movie runtime via a 10-minute bin group** rather than a new Power Query column — same result, no data change.
- **Header/footnote/rail added** — the fastest credibility gain for a portfolio screenshot.
- **Interactions set to None for the two bridge-table charts and the directors table** so the report doesn't appear broken.

**Left for you (Power BI Desktop only — I can't edit the .pbix from here)**
1. Fix `Column2` in Power Query (Section 2.1), then **Close & Apply**.
2. Import the theme and set canvas 1280×720 (Section 3).
3. Create the two measures (Section 4).
4. Rebind the three KPI cards, then build/reposition the visuals using the position tables (Sections 5–6).
5. Set the interactions (7.2), sync slicers, run the test pass (7.3).
6. Paste the insights and method text (Section 8), fill the footnote.
7. Save, export the PDF, take the screenshots.
