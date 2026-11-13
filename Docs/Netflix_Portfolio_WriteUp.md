# Netflix Content Analysis — Portfolio Write-Up

## 1. Resume bullet (3 lines)

**Netflix Content Analysis Dashboard** — Python/pandas + Power BI
- Cleaned and modelled 8,807 Netflix titles across three tables (titles, genres, countries) into a star schema with two DAX measures and a custom dark theme.
- Built a 3-page interactive report — KPI cards, year and month trends, year × month heatmap, country/genre top-10s, audience mix, runtime distribution and director ranking.
- Found that film is 69.6% of the catalogue, additions peaked at 2,016 titles in 2019, and adult content accounts for 45.5% of all titles.

---

## 2. LinkedIn / portfolio blurb

A streaming catalogue is easy to chart and hard to explain. For this project I took 8,807 Netflix titles, cleaned them in Python/pandas — splitting duration into runtime and seasons, deriving year/month added and mapping ratings to audience segments — then modelled them in Power BI as a small star schema with bridge tables for the multi-value genre and country fields.

The result is a three-page dark-themed report: a KPI and trend overview, a deep dive into when titles are added, who they're for, how long they run and which ratings dominate, and an insights page documenting method and caveats.

What the data says: 69.6% of the catalogue is film; additions peaked at 2,016 titles in 2019 after a 177% jump in 2017; July is the busiest month; the US supplies 3,690 country tags ahead of India's 1,046; and 45.5% of titles are aimed at adults, with films averaging 99.6 minutes.

Every chart is annotated with its own limitations — partial 2021 data, 831 titles with no country, and genre tags that sum above the title count. #PowerBI #Python #DataAnalytics #DataVisualization

---

## 3. Project summary (one paragraph)

**Problem.** Netflix's public catalogue lists 8,807 titles added between 2008 and September 2021, but the raw export is messy: multi-value genre and country fields, duration packed into a single text column, missing directors and countries, and a final year that stops in September. **Approach.** I cleaned the data in Python/pandas — repairing malformed quotes, splitting duration into `movie_minutes` and `tv_seasons`, deriving `year_added`, `month_added`, `decade` and `years_to_netflix`, mapping ratings to five audience segments, and keeping the first-listed country and genre as primary values — then loaded the result into Power BI as a fact table with one-to-many bridge tables for genre and country. The report uses two DAX measures (Movie % and average movie runtime), built-in aggregations elsewhere, a custom Netflix-inspired dark theme, and a fixed 12-pixel grid across three pages: an overview (KPI cards, additions per year, country and genre rankings), a deep dive (monthly seasonality, year × month heatmap, audience mix, rating treemap, runtime histogram, top directors) and an insights/method page. **Findings.** Film accounts for 69.6% of titles (6,131 vs 2,676) and catalogue growth peaked at 2,016 additions in 2019 after a 177% jump in 2017; July (827 titles) and December (813) are the strongest months; the US leads with 3,690 country tags, followed by India (1,046) and the UK (806); International Movies is the most common genre tag (2,752 titles); 45.5% of the catalogue targets adults; and films average 99.6 minutes with 74% running between 60 and 120 minutes. Caveats are documented in the report itself: 2021 is partial, 831 titles have no country, and 6,787 titles carry multiple genres, so tag counts exceed the catalogue size.

---

## 4. Talking points if you're asked about it (30 seconds each)

- **"Why a star schema instead of one wide table?"** Genre and country are one-to-many; keeping them separate means a title is counted once in KPI cards while still appearing under every genre it belongs to. It also makes the double-counting explicit and explainable instead of hidden.
- **"Why does the country chart show more than 8,807 titles in total?"** 1,315 titles are co-productions with more than one country. The chart counts title-tag pairs; the KPI cards count titles. Both are correct, and the footnote says so.
- **"Why does the country slicer use primary_country?"** With single-direction relationships, filtering the country dimension can't push back to the fact table. Using the fact-table field keeps the slicer genuinely global without creating ambiguous filter paths.
- **"What would you do next?"** Add release-year vs addition-year lag analysis (the `years_to_netflix` field is already there), a co-production network view, and a refresh script so the report can be re-run on newer data.
