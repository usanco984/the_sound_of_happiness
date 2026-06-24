# 🎵 The Sound of Happiness
Does the music people stream most reflect the happiness levels of the countries where they live?

A data visualization project combining Spotify chart data (2017–2021) with the World Happiness Report to explore whether national happiness shapes musical taste — built in Power BI with a Python preprocessing pipeline.

# Short Description
This project analyzes Spotify streaming data from 61 countries (2017–2021) alongside the World Happiness Report to test whether happier countries stream emotionally "brighter" music. Built with Power BI (DAX, Field Parameters, Add-on visuals) and a custom Python data pipeline.

# Project Overview
Music is often described as universal, but musical taste is not. This project asks: do countries with higher happiness scores also stream songs with higher musical Valence, a Spotify audio feature measuring emotional positivity?

This question felt personal to us. As a daily Spotify user, Yumi noticed that Japanese music often differs from global charts in emotional brightness and song structure. For Truc, moving to Seattle from Vietnam meant gravitating toward familiar Vietnamese songs, not necessarily happier ones, but culturally closer ones. Those observations became the seed of this analysis, which sits at the intersection of data analytics and everyday human experience.

# Key Takeaways
- Music preference appears to reflect culture more than happiness.
- COVID-19 shifted music mood unevenly. Acousticness rose in several countries, while Valence diverged by region.
- South America shows the highest music Valence despite only moderate happiness scores.
- Northern Europe and Oceania dominate happiness rankings, but not music Valence.
- Geographic happiness patterns stayed relatively stable from 2017 to 2021, unlike music mood.

# Key Visual Results
DashboardMusic Valence vs. Happiness (Animated)Global Map

Music Trends (The Sound of a Pandemic)High Valence does not equal High HappinessShow



# Business Problem
Music streaming platforms like Spotify invest heavily in understanding listener behavior across global markets. If there is a measurable relationship between a country's happiness level and the audio characteristics of its popular music, this could inform playlist curation and recommendation systems, regional marketing and content placement strategy, and how record labels decide which markets to target for specific genres or artists.

We chose this topic because it connects data analytics with culturally relevant, globally understandable human experience, engaging for analysts and general audiences alike.

# Dataset
SourceCoverageLinkSpotify Charts and Web API (collected by sebastianplnr, 2021)61 countries, 2017-2020, weekly Top 200, audio features (Valence, Energy, Danceability, Acousticness, Tempo)GitHub repoWorld Happiness Report (Figure 2.1)2017-2020, life evaluation scores and 6 explanatory factorsworldhappiness.report

Both datasets were merged at the country_year level using a Python preprocessing pipeline.

# Analysis Approach
Data Pipeline (Python)
The raw Spotify dataset contained approximately 2 million rows and required cleaning before it could be used in Power BI. All preprocessing was consolidated into a single script:

# merge.py -- core preprocessing steps

# 1. Load cleaned source file
df = pd.read_csv("5_clean_data_fixed.csv")

# 2. Remove comma formatting, convert streams to millions
df["streams"] = df["streams"].str.replace(",", "").astype(float) / 1_000_000

# 3. Aggregate into yearly, monthly, and Top 50 summaries
yearly = df.groupby(["country", "year"]).agg({...})
monthly = df.groupby(["country", "year", "month"]).agg({...})
top50 = df.sort_values("rank").groupby(["country", "year"]).head(50)

# 4. Add region classification
yearly["region"] = yearly["country"].map(region_lookup)

# 5. Create composite key for merging
yearly["country_year"] = yearly["country"] + "_" + yearly["year"].astype(str)

# 6. Merge with World Happiness Report
merged = yearly.merge(whr_df, on="country_year", how="left")

# 7. Export to Excel for Power BI
merged.to_excel("happiness_musicyearly_merged.xlsx", index=False)

Run with: python3 merge.py

Full script: /scripts/merge.py

Power BI Data Modeling

Three core tables connected via a composite country_year key:

happiness_musicyearly_merged   <->   country_year   <->   music_monthly
                                                     <->   music_top50

DAX Measures (KPI Cards)

daxKPI Happiest Country =
FIRSTNONBLANK(
   TOPN(1, VALUES(happiness_musicyearly_merged[country]),
   CALCULATE(AVERAGE(happiness_musicyearly_merged[happiness_score])), DESC),
   1)

KPI Top Valence Country =
FIRSTNONBLANK(
    TOPN(1, VALUES(happiness_musicyearly_merged[country]),
        CALCULATE(AVERAGE(happiness_musicyearly_merged[avg_valence])), DESC),
    1)

DAX Calculated Columns (Grouping)

daxValence Group =
IF(happiness_musicyearly_merged[avg_valence] < 0.50, "C_Low",
IF(happiness_musicyearly_merged[avg_valence] < 0.58, "B_Medium", "A_High"))

Happiness Group =
IF(happiness_musicyearly_merged[happiness_score] < 5.5, "Low",
IF(happiness_musicyearly_merged[happiness_score] < 7, "Medium", "High"))

# Visuals Used
Bubble Map, Animated Scatter Plot (Play Axis), Line Charts, Bar and Column Charts, Donut Chart, Advanced Cards, Word Cloud, Narrative Visual, Chiclet Slicer, Field Parameters.

Full methodology: /write-up/write-up.pdf

# Key Results

#QuestionFinding1Do happier countries listen to happier music?No strong relationship observed. Music preference appears to reflect culture and regional taste more than happiness.2Did music mood shift around COVID-19?Yes, unevenly. Acousticness rose in Canada, Taiwan, and Hong Kong. Energy declined globally. Valence diverged by country (Brazil +15.66%, Indonesia -11.97%).3Which regions show the highest-Valence music?South America consistently, despite only moderate happiness scores.4Are happier nations associated with different audio traits?Only a weak association. Northern Europe and Oceania rank highest in happiness, not Valence.5Are geographic happiness patterns stable over time?Yes, relatively stable from 2017 to 2021, unlike music Valence.


# Business Insights
Music mood does not scale linearly with national happiness. Platforms should be cautious about assuming that a happier market means a preference for brighter playlists.
Cultural and regional identity appears to be a stronger predictor of music taste than macro-level wellbeing indicators. Recommendation systems may benefit more from cultural and regional clustering than happiness-based segmentation.
COVID-19's effect on listening behavior was not uniform. A one-size-fits-all global content strategy during major world events may miss meaningful regional variation.
Audio feature shifts, such as rising Acousticness, could serve as an early signal of changing listener mood in specific markets, which may be useful for content and marketing teams.


# Power BI vs. Tableau

This project also served as a comparison point with prior Tableau projects. Yumi's previous project analyzed workforce stability, while Truc's focused on student depression data.

AreaTableauPower BIGeographic visualizationChoropleth maps built in minutes via drag-and-dropFilled Map deprecated in Web version; required Bubble Map workaroundCalculationsLOD and RANK expressions powerful but complex in interactive contextsDAX more explicit; faster iteration; correlation coefficient calculation unresolvedInteractivity and AnimationPages feature animates within worksheets, harder to sync across a dashboardPlay Axis integrates animation directly into the report pageParametersRequires manual setup via parameters and calculated fieldsField Parameters natively support dynamic metric switchingAdd-on visualsNo native Word Cloud; limited marketplaceRich AppSource library including Word Cloud, Narrative, and Advanced CardDesign polishMore flexible color customization out of the boxNo global theme control in Web version; per-visual adjustment required

Full comparison: Section 6 of the Write-up

# Project Strengths
- Successfully merged and reduced a approximately 2 million row raw dataset into a clean, analyzable model without losing meaningful trends
- Cohesive Spotify-inspired green and black design theme across all report pages
- Animated Play Axis scatter plot effectively communicated how the happiness-valence relationship evolved year over year
- DAX-driven grouping columns (Valence Group, Happiness Group) made 61-country comparisons interpretable at a glance
- Slicers enabled multi-page, multi-dimension filtering by country, year, and region for deeper exploration

# Limitations
- Filled Map was unavailable in Power BI Web, requiring a Bubble Map workaround that did not fully replicate the original choropleth vision
- Correlation coefficient calculation in DAX returned unreliable results and remained unresolved within the project scope
- Radar Chart was abandoned because it required a long-format data structure incompatible with the existing tables
- 2021 stream data appears incomplete, likely representing a partial year. Findings involving 2021 should be interpreted with caution.
- Working exclusively in the Power BI Web version limited canvas size control and global theme settings

# Next Steps
- Use Power BI Desktop instead of the Web version for greater layout and formatting flexibility
- Verify data types immediately after import to catch numeric column issues early
- Explore Python integration within Power BI for the correlation analysis
- Investigate Filled Map and geographic visualization alternatives earlier in the project lifecycle
- Connect to the Spotify API directly for more recent data beyond 2021 and explore live data connections
- Experiment with Radar Charts and other advanced visuals using a properly restructured dataset

# AI Assistance Disclosure
This project was built collaboratively in Power BI, with Claude (Anthropic) used as a support tool throughout the process. This included troubleshooting Power BI and DAX issues, refining DAX formulas, structuring the data pipeline logic, generating image collages for the write-up and poster, and editing and organizing this README and the accompanying write-up document. All analytical decisions, data interpretation, and final design choices were made by the project authors.


# Repository Structure
the-sound-of-happiness/
├── README.md
├── /scripts/
│   └── merge.py              # Data cleaning and merging pipeline
├── /data/
│   └── processed/            # Final Excel files used in Power BI
├── /powerbi/
│   └── *.png                 # Dashboard screenshots
├── /write-up/
│   └── write-up.pdf          # Full project write-up
└── /poster/
    └── poster.png

# Authors
Yumiko Kuwana and Truc
Graduate students, Data Analytics in Business

#Contact
Feel free to reach out via LinkedIn or open an issue on this repo.
