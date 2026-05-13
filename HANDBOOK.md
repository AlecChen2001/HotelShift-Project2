# MSA Investment Potential Analyzer — Project Handbook

---

## Table of Contents

1. [Project Overview](#1-project-overview)
   - [1.1 Purpose & Intended Use](#11-purpose--intended-use)
   - [1.2 System Architecture](#12-system-architecture)
   - [1.3 Technology Stack](#13-technology-stack)
2. [Data Pipeline](#2-data-pipeline)
   - [2.1 Data Sources](#21-data-sources)
   - [2.2 Data File Structure (sample_data.json)](#22-data-file-structure-sample_datajson)
   - [2.3 State-Level Manual Inputs](#23-state-level-manual-inputs)
   - [2.4 Hotel Conversion Relevance Score](#24-hotel-conversion-relevance-score)
   - [2.5 Refreshing the Data](#25-refreshing-the-data)
3. [Scoring Methodology](#3-scoring-methodology)
   - [3.1 Scoring Formula Overview](#31-scoring-formula-overview)
   - [3.2 Default Index Weights](#32-default-index-weights)
   - [3.3 Economic Index](#33-economic-index)
   - [3.4 Housing Affordability Index](#34-housing-affordability-index)
   - [3.5 Supply Demand Index](#35-supply-demand-index)
   - [3.6 Pricing Power Index](#36-pricing-power-index)
   - [3.7 Valuation Index (Value Creation)](#37-valuation-index-value-creation)
   - [3.8 Regulatory Market Index](#38-regulatory-market-index)
   - [3.9 Adjusting Weights in the Interface](#39-adjusting-weights-in-the-interface)
4. [Dashboard User Guide](#4-dashboard-user-guide)
   - [4.1 Index Scoring (Default View)](#41-index-scoring-default-view)
   - [4.2 Overview Page: Scatter Plot](#42-overview-page-scatter-plot)
   - [4.3 Heatmap](#43-heatmap)
   - [4.4 Detailed Analysis](#44-detailed-analysis)
   - [4.5 Raw Data](#45-raw-data)
   - [4.6 ReZone Intelligence Generator (AI Section)](#46-rezone-intelligence-generator-ai-section)
5. [Maintenance & Update Guide](#5-maintenance--update-guide)
   - [5.1 Updating the Dataset (Annual Refresh)](#51-updating-the-dataset-annual-refresh)
   - [5.2 Adding a New MSA](#52-adding-a-new-msa)
   - [5.3 Changing Index Weights Permanently](#53-changing-index-weights-permanently)
   - [5.4 Changing Sub-Factor Weights Permanently](#54-changing-sub-factor-weights-permanently)
   - [5.5 Adding a New Investment Factor](#55-adding-a-new-investment-factor)
   - [5.6 Troubleshooting Common Issues](#56-troubleshooting-common-issues)
6. [Deployment & Hosting](#6-deployment--hosting)
   - [6.1 Local Use (No Hosting Required)](#61-local-use-no-hosting-required)
   - [6.2 Web Hosting (Sharing with Others)](#62-web-hosting-sharing-with-others)
   - [6.3 Keeping the Data Up to Date on a Hosted Version](#63-keeping-the-data-up-to-date-on-a-hosted-version)
7. [Appendix](#7-appendix)
   - [7.1 Abbreviations & Glossary](#71-abbreviations--glossary)
   - [7.2 Key External Resources](#72-key-external-resources)
   - [7.3 Version History](#73-version-history)
   - [7.4 Disclaimer](#74-disclaimer)

---

## 1. Project Overview

The MSA Investment Potential Analyzer is a browser-based decision-support tool designed to evaluate and rank U.S. Metropolitan Statistical Areas (MSAs) for hotel-to-multifamily residential conversion investment opportunities. The platform aggregates publicly available data, computes a proprietary 6-dimensional investment score, and presents interactive visualizations to assist investment professionals in identifying the most viable conversion markets across the country.

### 1.1 Purpose & Intended Use

This tool is intended for investment analysts and portfolio managers evaluating hotel acquisition targets for adaptive reuse as multifamily residential properties. It should be used as a screening and prioritization tool, not as the sole basis for investment decisions. All outputs should be validated against primary data sources before committing capital.

### 1.2 System Architecture

The application is a static front-end delivered via `index.html`, with no application server, no API layer, and no database. All scoring and interaction logic runs in the browser. At runtime, the page loads data from `data/sample_data.json` (relative to `index.html`), so both files must be hosted together in the same static site. The JSON dataset is generated and refreshed by offline Python/Jupyter data pipelines, then published for the front-end to consume.

| File | Description |
|------|-------------|
| `index.html` | Complete application — HTML structure, CSS styling, and all JavaScript logic in one file |
| `sample_data.json` | Structured dataset containing metrics for all selected MSAs (current criteria: population over 300,000 in 2024) |
| `update_sample_data.py` | Python pipeline that fetches, cleans, and integrates the latest source data to generate the structured `sample_data.json` dataset for all MSAs |
| `US_census_fixed.ipynb` *(optional)* | Notebook-based companion to `update_sample_data.py` that documents each data module with step-by-step extraction, visualization, and explanation. Designed for data scientists to debug the pipeline, inspect data quality, and understand how source metrics flow into `sample_data.json` |

### 1.3 Technology Stack

| Library / Technology | Purpose |
|----------------------|---------|
| Chart.js 4.4.0 | Bar charts, scatter plots, and radar charts for investment visualization |
| D3.js v7 + TopoJSON | Interactive U.S. choropleth heatmap |
| MathJax 3 | Rendering mathematical formulas in the Scoring section |
| jsPDF 2.5.1 + html2pdf.js | PDF export functionality in the ReZone Intelligence Generator |
| Google Fonts (Noto Sans TC / Noto Serif TC) | Typography |
| Google Gemini AI API | AI-generated market intelligence in the ReZone section (requires user-provided API key) |

---

## 2. Data Pipeline

This section documents where the data comes from, how it is processed, and what each field means in `sample_data.json`. This is the most critical section to review if the underlying data needs to be refreshed or expanded.

### 2.1 Data Sources

| Data Field Group | Data Level | Source |
|------------------|-----------|--------|
| Population, income, rent | MSA | U.S. Census Bureau — American Community Survey (ACS) 1-Year Estimates |
| Employment figures | MSA | U.S. Census Bureau ACS / Bureau of Labor Statistics (BLS) |
| Multifamily unit counts | MSA | Census Bureau Building Permits Survey / ACS housing inventory |
| Cap Rates | State | Industry reports (manually researched and entered in data sheets) |
| Operating expense ratios | State | Industry benchmarks (manually entered per state) |
| Effective tax rates | State | State and local tax data (manually entered per state) |
| Hotel Conversion Relevance | State | Manually researched and entered — see Section 2.4 |
| HERS Index Scores | State | RESNET (manually entered per state) |

### 2.2 Data File Structure (sample_data.json)

The data file has two top-level keys:

- `stats` — metadata object describing the dataset (year, total MSA count, scoring methodology)
- `msas` — array of MSA objects, one per market

Each MSA object contains the following fields:

| Field Name | Type | Description | Example |
|------------|------|-------------|---------|
| `msa_code` | Integer | OMB Metropolitan Statistical Area FIPS code | `33100` |
| `msa_name` | String | Full official MSA name including state abbreviation | `"Miami-Fort Lauderdale-West Palm Beach, FL Metro Area"` |
| `year` | Integer | Data vintage year (currently 2024) | `2024` |
| `Total_Population` | Integer | Total MSA population | `6457988` |
| `Median_Rent` | Integer | Median gross monthly rent (USD) | `2083` |
| `Median_Income` | Integer | Median household income (USD) | `80625` |
| `Median_Home_Value` | Integer | Median owner-occupied home value (USD) | `510600` |
| `Operating_Expense_Ratio` | Float | State-level OER (see Section 2.4). Range: 0–1 | `0.52` |
| `Hotel_Cap_Rate` | Float | State-level hotel capitalization rate. E.g. `0.095` = 9.5% | `0.095` |
| `Multifamily_Cap_Rate` | Float | State-level multifamily cap rate. E.g. `0.0625` = 6.25% | `0.0625` |
| `Hotel_Effective_Tax_Rate` | Float | State-level effective property tax rate for hotels | `0.019988` |
| `Multifamily_Effective_Tax_Rate` | Float | State-level effective property tax rate for multifamily | `0.019988` |
| `Employment_Rate` | Float | Employed persons / labor force. E.g. `0.954` = 95.4% | `0.95387` |
| `Employment_Growth` | Float | YoY employment growth rate. E.g. `0.049` = 4.9% | `0.048835` |
| `Pop_Growth` | Float | YoY population growth rate | `0.044441` |
| `Income_Growth` | Float | YoY median income growth rate | `0.057086` |
| `Rent_to_Income_Ratio` | Float | Monthly rent / monthly income. E.g. `0.31` = 31% | `0.310028` |
| `New_Multi_Units` | Integer | Net new multifamily units added YoY | `0` |
| `Vacancy_Rate` | Float | Vacant housing units / total housing units | `0.122524` |
| `Rent_Growth` | Float | YoY rent growth rate | `0.088297` |
| `Value_Creation` | Float | Computed value uplift from hotel-to-MF conversion (USD/unit). See formula in Section 3.7 | `65673.701053` |
| `Hotel_Conversion_Relevance` | String | Regulatory feasibility score [0–1]. See Section 2.4 | `"A"` |
| `Average_HERS_Index_Score` | Integer | State average Home Energy Rating System score | `57` |
| `Investment_Score` | Float | Final composite investment score [0–100]. See Section 3 | `67.633345` |

### 2.3 State-Level Manual Inputs

Several fields are not available from Census Bureau APIs and were manually researched and entered via data sheets. These state-level inputs are assigned uniformly to all MSAs within a state:

- **`Operating_Expense_Ratio`** — sourced from institutional real estate benchmarks per state.
- **`Hotel_Cap_Rate` and `Multifamily_Cap_Rate`** — sourced from brokerage and industry market reports.
- **`Hotel_Effective_Tax_Rate` and `Multifamily_Effective_Tax_Rate`** — derived from state tax data.
- **Hotel Conversion Relevance** — mapped score from Conversion Category; ratings are based on policy and execution feasibility for hotel-to-multifamily conversion.

To update these figures: locate the relevant state row in the source data sheets, update the value, re-run the Python pipeline, and regenerate `sample_data.json`.

### 2.4 Hotel Conversion Relevance Score

Each state was assigned a letter grade (A+ through F) reflecting its regulatory environment for hotel-to-multifamily conversions, based on factors such as zoning flexibility, adaptive reuse policies, and permitting complexity. Letter grades are then converted to a numeric [0, 1] score using the following non-equidistant mapping, which applies steeper penalties to lower tiers to reflect higher execution risk:

| State Letter Rating | Numeric Score |
|---------------------|---------------|
| A+ | 1.00 |
| A | 0.90 |
| A- | 0.84 |
| B+ | 0.78 |
| B | 0.66 |
| B- | 0.61 |
| C+ | 0.57 |
| C | 0.52 |
| C- | 0.40 |
| D+ | 0.33 |
| D | 0.25 |
| D- | 0.12 |
| F | 0.00 |

To update state ratings: revise the letter grade in the source data sheet, re-run the conversion mapping in the notebook, and regenerate `sample_data.json`. Do not change the numeric scale above unless the scoring methodology is being intentionally revised.

### 2.5 Refreshing the Data

The current dataset reflects 2024 ACS 1-Year Estimates. To update the data for a future year in `update_sample_data.py`:

1. Fetch the latest ACS 1-Year Estimate tables from the US Census for the relevant year. The Python function `fetch_recent_n_years(code: str, name: str, n_years: int = 6, max_lookback: int = 12) -> pd.DataFrame` automatically retrieves the most recent available n years from the U.S. Census API. The MSA population cutoff used by the website is currently **300,000**.

2. Refresh the state-level data sheets with any revised cap rates, OER figures, tax rates, or conversion ratings for the new year:
   - **Cap rates:** `Hotel_vs_MF_Cap_Rate_Spread_analysis.xlsx`, sheet "Cap Rate Spread Analysis", columns "Hotel Cap Rate" and "MF Cap Rate"
   - **OER:** `State OPEX Assumptions for Housing.xlsx`, column "Default OPEX %"
   - **Tax rates:** `State Property Tax Comparison_ Hotels vs. Multifamily_New.xlsx`, columns "Hotel Effective Rate" and "Multifamily Effective Rate"
   - **Conversion ratings:** `Hotel_vs_MF_Cap_Rate_Spread_analysis.xlsx`, sheet "Regulatory Environment", column "Conversion Category"
   - **HERS index:** `2014-HERS-Activity-by-State.xlsx`

3. Run the Python pipeline to merge all sources, compute derived fields from raw inputs (e.g. `Rent_to_Income_Ratio`, `Value_Creation`, etc.), and produce a new `sample_data.json`.

4. Replace the existing `sample_data.json` in the project folder with the new file. No changes to `index.html` are required unless new MSAs are added or field names change.

---

## 3. Scoring Methodology

The Investment Score is a composite index ranging from **0 to 100**. It is computed through a two-level weighting system: first, raw sub-factors are normalized and weighted within each of six indices; then the six indices are weighted against each other to produce the final score.

### 3.1 Scoring Formula Overview

- **Step 1 — Sub-factor normalization:** each raw variable (except for the HERS index score) is min-max normalized across all MSAs to a [0, 1] scale. Higher normalized values always represent more favorable conditions for investors (vacancy rate and new multifamily unit factors are inverted before normalization so that a lower raw factor = higher normalized score = more favorable condition for investors).

- **Step 2 — Index computation:** normalized sub-factors are combined using configurable intra-index weights, producing six index values each in [0, 1].

- **Step 3 — Final score:** the six index values are combined using cross-index weights, then min-max scaled to [0, 100].

The web interface allows users to adjust both sub-factor weights and cross-index weights in real time via the Investment Score section. The default weights shown below reflect the baseline configuration used to produce the `Investment_Score` values stored in `sample_data.json`.

### 3.2 Default Index Weights

| Index | Default Weight | Rationale |
|-------|---------------|-----------|
| Economic Index | 20% | Evaluates strength and growth potential of an MSA's economic fundamentals |
| Housing Affordability Index | 15% | Evaluates local housing affordability pressure relative to income |
| Supply Demand Index | 15% | Evaluates local supply-demand balance through multifamily pipeline growth and vacancy tightness |
| Pricing Power Index | 15% | Measures ability of landlords to raise rents over time |
| Valuation Index | 15% | Measures conversion-related value uplift by comparing multifamily and hotel valuation implied by the same NOI stream |
| Regulatory Market Index | 20% | Captures policy and execution feasibility for hotel-to-multifamily conversion at the state level |

### 3.3 Economic Index

Evaluates strength and growth potential of an MSA's economic fundamentals. Composed of four equally-weighted sub-factors by default:

- **Employment Rate** = Employed / Labor Force Population
- **Employment Growth** = (Employed_t − Employed_(t−1)) / Employed_(t−1)
- **Population Growth** = (Pop_t − Pop_(t−1)) / Pop_(t−1)
- **Income Growth** = (Income_t − Income_(t−1)) / Income_(t−1)

### 3.4 Housing Affordability Index

Evaluates local housing affordability pressure relative to income. Composed of one factor by default:

- **Rent-to-Income Ratio** = Median Monthly Rent / (Median Annual Income / 12)

### 3.5 Supply Demand Index

Evaluates local supply-demand balance through multifamily pipeline growth and vacancy tightness. Composed of two equally-weighted sub-factors by default:

- **New Multi Units** = Total Multi Units_t − Total Multi Units_(t−1) *(inverted: lower new supply = better for conversions)*
- **Vacancy Rate** = Vacant Units / Total Housing Units *(inverted: lower vacancy = better)*

### 3.6 Pricing Power Index

A single-factor index measuring the ability of landlords to raise rents over time:

- **Rent Growth** = (Rent_t − Rent_(t−1)) / Rent_(t−1)

### 3.7 Valuation Index (Value Creation)

Measures the per-unit economic value generated by converting a hotel to multifamily, derived from the cap rate spread between the two asset classes applied to the stabilized NOI:

```
Value Creation = Annual Rent × (1 − OER) × (1 / MF Cap Rate − 1 / Hotel Cap Rate)
```

> **Interpretation:** A positive Value Creation figure means that the same income stream is worth more when capitalized as multifamily than as a hotel. The larger the spread, the greater the conversion incentive.

### 3.8 Regulatory Market Index

A single-factor index using the Hotel Conversion Relevance score (see Section 2.4). This score captures state-level policy, zoning, and permitting feasibility on a [0, 1] scale.

### 3.9 Adjusting Weights in the Interface

Users can override default weights directly in the browser without modifying any files:

1. Open the web application and click **Investment Score** in the navigation bar.
2. For the Economic and Supply Demand indices, expand the **Sub-Factor Weights** panel to adjust how individual variables are weighted within those indices.
3. Click **Update [Index Name] Index** after adjusting sub-factor weights.
4. Use the index-level sliders to adjust the relative importance of each of the six indices. Weights auto-normalize to sum to 100%.
5. Click **Update Index Weights** after adjusting six index weights.
6. All rankings, charts, and the heatmap update in real time.

> **Note:** Weight changes are session-only and reset on page refresh. To make permanent changes, modify the default values of the weight slider elements in `index.html` (search for `weightEconomic`, `weightStability`, etc.) and update the corresponding `currentWeights` object at the top of the JavaScript section.

---

## 4. Dashboard User Guide

The application is organized into six main sections, accessible via the navigation bar at the top of the page.

> **Note:** MSAs with populations below 300,000 are excluded from this dashboard.

### 4.1 Index Scoring (Default View)

The landing view when the application opens. Contains:

- **Summary KPI cards** — total MSA count, highest scoring MSA, number of high-opportunity markets, and average investment score across all MSAs.
- **Index weight configuration panel** — sliders for both cross-index and sub-factor weights. All charts and rankings update in real time as weights change.
- **Live ranking table** — all MSAs sorted by current Investment Score, with a Details button to open the per-MSA deep-dive view.

### 4.2 Overview Page: Scatter Plot

An interactive scatter chart comparing two user-selected metrics across all MSAs. Use this view to identify clusters and outliers. Controls:

- **X Axis and Y Axis dropdowns** — select any of 10 available metrics.
- **Color By** — optionally color points by a third metric (e.g., Investment Score) to add a third dimension.
- **Show Labels toggle** — display or hide MSA name labels on the chart points.
- Hovering over a point displays the MSA name and exact metric values.

### 4.3 Heatmap

A D3.js choropleth map of the contiguous United States. Each state is shaded based on the average of its MSAs' selected metric. Controls:

- **Metric selector buttons** — switch between Investment Score, Rent Growth, Vacancy Rate, Population Growth, and Income Growth.
- **Zoom & Pan** — use mouse scroll to zoom; click and drag to pan.
- **Reset Zoom** — returns the map to its default view.
- Clicking on a state reveals a panel listing all MSAs within that state with their individual metric values.

### 4.4 Detailed Analysis

An in-depth profile for a single selected MSA. Opened by clicking the **Details** button from the Investment Score ranking table. Contains:

- **Summary metrics** — population, median rent, income, home value, and investment score.
- **Radar chart** — a six-axis spider diagram showing normalized scores across all six investment dimensions for the selected MSA.
- **Score legend** — shows exact normalized index values and how they contributed to the final score.
- **Full raw data table** — all underlying metrics for the MSA.

### 4.5 Raw Data

A sortable and searchable table showing all raw data fields and computed factor values for all selected MSAs. Features:

- Search by MSA name using the text input.
- Sort by Score, Population, Population Growth, or Median Income using the dropdown.
- **Download CSV button** — exports the complete dataset as a comma-separated file.
- Factor legend at the top of the page explains what each column represents and its directionality (↑ = higher is better, ↓ = lower is better).
- State-level input table — shows the manually-entered OER, cap rate, and tax rate for each state.

### 4.6 ReZone Intelligence Generator (AI Section)

An AI-powered research tool that generates per-city market intelligence reports using Google's Gemini API. This section requires an internet connection and a valid Gemini API key.

**Steps to use:**

1. Paste your Gemini API key from [Google AI Studio](https://aistudio.google.com) into the API key field. The key is stored only in the browser and is never sent anywhere except directly to Google's servers.
2. Select 1–15 MSAs from the ranked list using the checkboxes (or use the **Top 5 / Top 10** quick-select shortcuts).
3. Click **Generate Intelligence Report**. The tool sends one request per city to the Gemini API. Processing typically takes 15–40 seconds.
4. The generated report appears inline, containing for each city: key statistics, primary investment catalyst, building stock profile, active and pipeline conversion projects, policy timeline, financial mechanisms and incentive programs, AI intelligence feed (news, policy, deal events), and rent control status.
5. Use the **Download PDF** button to export the full report as a PDF file.

> ⚠️ The AI-generated content is produced by Gemini and may contain inaccuracies. All statistics, project names, and policy details in the AI report should be independently verified before use in investment analysis or client presentations.

---

## 5. Maintenance & Update Guide

This section is intended for whoever maintains the tool going forward. It covers the most common update scenarios in plain language.

### 5.1 Updating the Dataset (Annual Refresh)

1. Download new ACS 1-Year Estimate data from [data.census.gov](https://data.census.gov). Although multiple historical years are collected, the scoring workflow mainly uses the latest year and the latest year-over-year changes for level and growth calculations.

   The MSA population cutoff is currently **300,000**. In function `build_payload() -> dict`, this is implemented as `msa_features["Total_Population"] >= 300000`. Maintainers can change this threshold if a different market coverage is needed.

   The relevant Census table series are:

   | Field | Census Code |
   |-------|-------------|
   | `Total_Population` | `B01003_001E` |
   | `Laborforce_Population` | `DP03_0002E` |
   | `Employed` | `DP03_0004E` |
   | `Median_Household_Income` | `S1903_C03_001E` |
   | `Median_House_Value` | `B25077_001E` |
   | `Total_Housing_Units` | `B25002_001E` |
   | `House_Occupied` | `B25002_002E` |
   | `House_Vacant` | `B25002_003E` |
   | `Median_Gross_Rent` | `B25064_001E` |
   | `5_to_9_units` | `DP04_0011E` |
   | `10_to_19_units` | `DP04_0012E` |
   | `20_or_more_units` | `DP04_0013E` |

2. Update the state-level Excel inputs before running the update. The pipeline will continue to work as long as workbook structure, sheet names, and required columns remain unchanged:
   - **Cap rates:** `Hotel_vs_MF_Cap_Rate_Spread_analysis.xlsx`, sheet "Cap Rate Spread Analysis", columns "Hotel Cap Rate" and "MF Cap Rate"
   - **OER:** `State OPEX Assumptions for Housing.xlsx`, column "Default OPEX %"
   - **Tax rates:** `State Property Tax Comparison_ Hotels vs. Multifamily_New.xlsx`, columns "Hotel Effective Rate" and "Multifamily Effective Rate"
   - **Conversion ratings:** `Hotel_vs_MF_Cap_Rate_Spread_analysis.xlsx`, sheet "Regulatory Environment", column "Conversion Category"
   - **HERS index:** Most recent year's HERS index can be found at [hersindex.com](https://www.hersindex.com/hers-index/understanding-hers-index/) or by searching the most recent annual "HERS® Activity by State" source. Note: this field is retained in the Raw Data page for reference and is **not** currently used in core factor/index scoring, so skipping it will not change the composite rating logic.

3. Run the `.py` pipeline. `update_sample_data.py` merges Census data with state-level inputs, computes all derived fields, and outputs a fresh `sample_data.json`.

4. Replace the old `sample_data.json` in the project folder with the new file.

5. Open `index.html` in a local browser and verify that MSA count and scores look reasonable. Check 2–3 known markets manually against the raw data.

### 5.2 Adding a New MSA

1. Look up the MSA FIPS code from the Census Bureau's MSA reference list.
2. Add all required fields for the new MSA to the Jupyter notebook data source (Census pull + manual state inputs).
3. Re-run the notebook to regenerate `sample_data.json` with the new MSA included.
4. No changes to `index.html` are needed — the application dynamically reads all MSAs from the JSON file.

### 5.3 Changing Index Weights Permanently

The default weights are defined in two places in `index.html`. To change them:

1. Open `index.html` in a text editor.
2. Find the section labelled **Global Variables** (approximately line 2193). Locate the `currentWeights` object and update the default decimal values:

   ```javascript
   economic:   0.20,  // default 20%
   stability:  0.15,  // default 15%
   supply:     0.15,  // default 15%
   pricing:    0.15,  // default 15%
   valuation:  0.15,  // default 15%
   regulatory: 0.20   // default 20%
   ```

3. Also update the `value` attributes on the corresponding `<input type='range'>` sliders so the visual controls reflect the new defaults. Search for `id='weightEconomic'`, `id='weightStability'`, etc.

4. Save and refresh the browser. The ranking table will immediately reflect the new default weights.

### 5.4 Changing Sub-Factor Weights Permanently

Sub-factor slider defaults are set via the `value` attribute on each sub-factor range input. In `index.html`, search for the following IDs and update their `value` attributes:

| Element ID | Controls |
|------------|----------|
| `intraEmpRate` | Employment Rate weight within Economic Index |
| `intraEmpGrowth` | Employment Growth weight within Economic Index |
| `intraPopGrowth` | Population Growth weight within Economic Index |
| `intraIncGrowth` | Income Growth weight within Economic Index |
| `intraRentIncome` | Rent-to-Income Ratio weight within Stability Index |
| `intraVacancy` | Vacancy Rate weight within Stability Index |

> Sub-factor weights are auto-normalized within each index, so their exact values only matter relative to each other. Setting all four Economic sub-factors to 25 is equivalent to setting them all to any equal value.

### 5.5 Adding a New Investment Factor

Adding a new data field and incorporating it into the score requires changes in three places:

1. **`sample_data.json`** — add the new field to every MSA object in the data pipeline (notebook).
2. **`index.html`** — add the field to the Raw Data table (search for the `rawdata-controls` section and the `updateRawData()` function). Add the field to the factor legend.
3. **`index.html` scoring logic** — add a new sub-factor slider to the appropriate index panel, handle the new field in the `computeScore()` function (search for `'computeScore'`), and ensure the field is included in min-max normalization.

> ⚠️ This is a moderately complex change that requires familiarity with JavaScript. Test thoroughly on a local copy before deploying to the client.

### 5.6 Troubleshooting Common Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Page loads but no data appears / blank table | `sample_data.json` is missing or in the wrong folder | Ensure `sample_data.json` is in the same directory as `index.html`. Open browser developer tools (F12) → Console to confirm the error message. |
| Map shows blank / no colors on heatmap | Browser blocked external resource (TopoJSON CDN) | Check internet connection. The map requires the `topojson-client` library from jsDelivr CDN. If offline, host the library locally. |
| ReZone report generates an error | Invalid Gemini API key OR too many MSAs selected at once | Verify the API key in Google AI Studio. Reduce selection to 5 or fewer cities if the error mentions truncation. |
| Weights don't sum to 100% | Sub-factor sliders adjusted but Update button not clicked | After changing sub-factor sliders, click the **Update [Index] Index** button before expecting score changes. |
| Scores look wrong after updating data | Field names in new `sample_data.json` don't match expected names | Compare field names in the new JSON against the table in Section 2.2. The JavaScript references exact field names — any mismatch will silently produce 0 values. |
| Charts don't render correctly on mobile | The application is optimized for desktop (1600px max-width) | The tool is designed for desktop use. Mobile view may have limited chart readability — this is expected behavior. |

---

## 6. Deployment & Hosting

Because the application is a single HTML file with an external JSON data file, deployment is straightforward and requires no server-side infrastructure.

### 6.1 Local Use (No Hosting Required)

1. Place `index.html` and `sample_data.json` in the same folder on any computer.
2. Open `index.html` in a modern web browser (Chrome, Edge, or Firefox recommended).
3. No internet connection is required for core functionality. The ReZone AI section and the Google Fonts / CDN libraries require internet access.

> **CORS Note:** Some browsers block local file system requests for JSON files due to CORS security policy. If the application loads but shows no data, run it through a simple local server. In a terminal, navigate to the project folder and run:
> ```bash
> python -m http.server 8000
> ```
> Then open `http://localhost:8000` in your browser.

### 6.2 Web Hosting (Sharing with Others)

To share the tool with a wider audience, host both files on any static web host:

- **GitHub Pages** — free, version-controlled. Upload both files to a repository and enable Pages in repository settings.
- **Netlify / Vercel** — drag-and-drop deployment. Free tier is sufficient for this application.
- **AWS S3 Static Website** — enterprise-grade option with access controls.
- **Any shared hosting service** — upload both files via FTP. No server-side languages needed.

After uploading, navigate to the hosted URL in a browser to confirm the application loads correctly.

### 6.3 Keeping the Data Up to Date on a Hosted Version

When the dataset is refreshed (see Section 5.1), simply replace `sample_data.json` on the hosting platform with the new version. The HTML file does not need to change. Users will see updated data immediately on their next page load.

---

## 7. Appendix

### 7.1 Abbreviations & Glossary

| Term | Definition |
|------|------------|
| MSA | Metropolitan Statistical Area — a geographic unit defined by the U.S. Office of Management and Budget (OMB) centered on an urban core of at least 50,000 people, including surrounding economically integrated counties. |
| ACS | American Community Survey — the U.S. Census Bureau's annual survey providing detailed demographic, social, economic, and housing statistics at the MSA level. |
| OER | Operating Expenditure Ratio — the proportion of a property's gross operating income consumed by operating expenses (excluding debt service). |
| Cap Rate | Capitalization Rate — the ratio of a property's net operating income (NOI) to its market value. Used to compare asset classes and geographies. |
| MF Cap Rate | Multifamily Capitalization Rate — cap rate specific to residential multifamily properties. |
| NOI | Net Operating Income — gross rental income minus operating expenses, before debt service and taxes. |
| HERS Index | Home Energy Rating System Index — a standard scoring system for residential building energy efficiency. Lower scores indicate more efficient buildings. |
| YoY | Year-over-Year — the percentage change between the current year's value and the prior year's value. |
| Hotel-to-MF Conversion | Adaptive reuse of a hotel building as a multifamily residential property, the core investment thesis this tool is designed to evaluate. |
| Value Creation | The per-unit financial uplift generated by capitalizing a given NOI stream at the multifamily cap rate rather than the hotel cap rate. |
| ReZone Intelligence | The AI-powered market intelligence feature within the application, powered by Google Gemini API. |

### 7.2 Key External Resources

- [U.S. Census Bureau Data Portal](https://data.census.gov)
- [Census Bureau MSA Delineation Files](https://www.census.gov/programs-surveys/metro-micro/about/delineation-files.html)
- [Google AI Studio (Gemini API Keys)](https://aistudio.google.com)
- [Chart.js Documentation](https://www.chartjs.org/docs/latest)
- [D3.js Documentation](https://d3js.org)

### 7.3 Version History

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2024 | Initial release. 169 MSAs, 6-dimensional scoring system, interactive heatmap, scatter plot, detailed view, raw data export, ReZone AI generator. |

### 7.4 Disclaimer

This tool is designed for informational and analytical purposes only. Investment scores and rankings are generated from publicly available data using a proprietary methodology and do not constitute financial, legal, or investment advice. All data should be independently verified before use in any investment decision. Cap rates, operating costs, and regulatory conditions are subject to rapid change and should be treated as estimates only.

AI-generated content in the ReZone Intelligence section is produced by Google Gemini and may contain factual errors, outdated information, or omissions. All AI output must be verified against primary sources prior to use in client deliverables.
