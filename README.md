# 🎵 The Sound of Happiness

Does the music a country streams reflect how happy its people are?

This project explores the relationship between national happiness and popular music characteristics by combining Spotify streaming data with the World Happiness Report. Using Power BI and a Python preprocessing pipeline, we analyzed whether countries with higher happiness scores tend to stream music with higher Valence, a Spotify audio feature that represents emotional positivity.

▶ View Live Report on Power BI Public · 📄 Full Write-up · 🖼 Poster

# Overview
Music is often described as universal — but musical taste is not. This project asks whether countries with higher happiness scores also stream emotionally "brighter" music, as measured by Spotify's Valence audio feature.

The question felt personal. As a daily Spotify user, Yumi noticed that Japanese music often differs from global charts in emotional tone and structure. For Truc, moving to Seattle from Vietnam meant gravitating toward familiar Vietnamese songs — not necessarily happier ones, but culturally closer ones. Those observations became the seed of this analysis.

# Key Fidings
| # | Question | Finding |
|---|---|---|
| 1 | Do happier countries stream happier music? | No strong relationship. Culture and regional identity predict music taste more than happiness scores. |
| 2 | Did COVID-19 shift music mood? | Yes, unevenly. Acousticness rose in Canada, Taiwan, and Hong Kong. Brazil's Valence rose 15.66%; Indonesia's fell 11.97%. |
| 3 | Which regions stream the brightest music? | South America consistently — despite only moderate happiness scores. |
| 4 | Are happiness patterns stable over time? | Yes. Geographic happiness remained relatively stable from 2017–2021, unlike music Valence. |

> Bottom line: What we listen to is shaped more by where we come from
> than by how happy we are.

---

# Data Sources
| Dataset | Coverage | Source |
|---|---|---|
| Spotify Charts + Web API (sebastianplnr) | 61 countries, 2017–2021, Weekly Top 200, audio features | [GitHub](https://github.com/sebastianplnr/spotify_music_and_happiness) |
| World Happiness Report (Figure 2.1) | 2017–2021, life evaluation scores + 6 explanatory factors | [worldhappiness.report](https://worldhappiness.report) |

---
Both datasets were merged at the country_year level using a custom Python pipeline.
---

# Technical Approach

Data Pipeline (Python)

The raw Spotify dataset contained approximately 2 million rows. All preprocessing was handled in a single script:

```````python
# merge.py — core preprocessing steps

# 1. Load cleaned source file
df = pd.read_csv("5_clean_data_fixed.csv")

# 2. Remove comma formatting, convert streams to millions
df["streams"] = df["streams"].str.replace(",", "").astype(float) / 1_000_000

# 3. Aggregate into yearly, monthly, and Top 50 summaries
yearly  = df.groupby(["country", "year"]).agg({...})
monthly = df.groupby(["country", "year", "month"]).agg({...})
top50   = df.sort_values("rank").groupby(["country", "year"]).head(50)

# 4. Add region classification
yearly["region"] = yearly["country"].map(region_lookup)

# 5. Create composite key for merging
yearly["country_year"] = yearly["country"] + "_" + yearly["year"].astype(str)

# 6. Merge with World Happiness Report
merged = yearly.merge(whr_df, on="country_year", how="left")

# 7. Export to Excel for Power BI
merged.to_excel("happiness_musicyearly_merged.xlsx", index=False)

```````

# Power BI Data Model

Three core tables connected via a composite country_year key:
```````
happiness_musicyearly_merged   <->   country_year   <->   music_monthly
                                                     <->   music_top50
```````

# DAX Highlights

```````DAX
-- KPI: Happiest Country
KPI Happiest Country =
FIRSTNONBLANK(
    TOPN(1, VALUES(happiness_musicyearly_merged[country]),
    CALCULATE(AVERAGE(happiness_musicyearly_merged[happiness_score])), DESC),
    1)

-- Grouping: Valence tiers for cross-country comparison
Valence Group =
IF(happiness_musicyearly_merged[avg_valence] < 0.50, "C_Low",
IF(happiness_musicyearly_merged[avg_valence] < 0.58, "B_Medium", "A_High"))

```````

# Visuals Used

Bubble Map, Animated Scatter Plot (Play Axis), Line Charts, Bar and Column Charts, Donut Chart, Advanced Cards, Word Cloud, Narrative Visual, Chiclet Slicer, Field Parameters

# Power BI vs. Tableau

This project served as a direct comparison with prior Tableau work. Power BI's Play Axis enabled smooth in-dashboard animation, and DAX allowed more explicit KPI logic. The main tradeoff: Filled Map was unavailable in Power BI Web, requiring a Bubble Map workaround, and global theme control was limited compared to Tableau's desktop environment.

# Limitations

- Filled Map was unavailable in Power BI Web — a Bubble Map was used as a workaround
- Correlation coefficient calculation in DAX returned unreliable results and remains unresolved
- 2021 stream data appears incomplete (likely a partial year) — findings involving 2021 should be interpreted with caution
- Working in Power BI Web limited canvas size control and global theme settings

# What We'd Do Differently

- Use Power BI Desktop from the start for greater layout and formatting control
- Verify data types immediately after import to catch numeric column issues early
- Explore Python integration within Power BI for correlation analysis
- Connect to the Spotify API directly for data beyond 2021
- Investigate geographic visualization alternatives earlier in the process

# Repository Structure

```````
the-sound-of-happiness/
├── README.md
├── scripts/
│   └── merge.py              # Data cleaning and merging pipeline
├── data/
│   └── processed/            # Final Excel files used in Power BI
├── powerbi/
│   └── *.png                 # Dashboard screenshots
├── write-up/
│   └── write-up.pdf          # Full project write-up
└── poster/
    └── poster.pdf
```````

# AI Assistance Disclosure

Claude (Anthropic) was used as a support tool throughout this project — for troubleshooting Power BI and DAX issues, structuring the data pipeline, generating image collages, and editing documentation. All analytical decisions, data interpretation, and design choices were made by the project authors.

# Authors

Yumiko Kuwana and Vo Thanh Truc Huynh
Graduate students, Data Analytics in Business/ Seattle Pacific University

Feel free to reach out via LinkedIn or open an issue on this repo.

