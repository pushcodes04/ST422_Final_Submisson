# ST422 Team 1 — Brief 8: Road Safety Analysis

Analysis of DfT STATS19 road casualty data for ST422 Statistical Consulting.  
Brief 8: identifying road safety hotspots, contributory factors, and evaluating whether recent policy changes are working.

---

## Repo Structure

```
ST422_Team1_Project/
├── Final_Workflow/              ← MAIN ANALYSIS — start here
│   ├── Data_Prep/               ← Step 1: data cleaning pipeline
│   │   ├── data_cleaning.ipynb
│   │   ├── Data/                ← place raw DfT CSVs here (see below)
│   │   │   ├── local-authority-ons-district-names.csv   ← already in repo
│   │   │   └── ons_la_population_2024.csv               ← already in repo
│   │   └── Cleaned/             ← generated on run, not committed
│   └── Data_Analysis/           ← Step 2: full analysis pipeline
│       ├── data_analysis.ipynb
│       └── Outputs/             ← generated on run, not committed
│           ├── figures/         ← fig01–fig17 PNGs
│           ├── tables/          ← tab_*.csv files
│           ├── findings.html    ← self-contained findings page
│           ├── traceability.csv ← maps every claim to an output file
│           └── run_metadata.json
├── Quality_Assurance/
│   └── QA_Data_Load.ipynb       ← 32 automated checks (31/32 expected to pass)
├── Group_Contact/               ← team coordination evidence (B6)
│   ├── Action_Log/              ← full task log with owners and evidence pointers
│   └── Meeting_Minutes/         ← minutes from all 6 team meetings
├── Management_Plan/             ← workplan, risk register, QA plan (B7)
├── requirements.txt             ← all Python dependencies
└── README.md                    ← this file
```

---

## Quick Start — Reproduce the Full Analysis

### Step 1: Install dependencies (once only)

From the **repo root**:

```bash
pip install -r requirements.txt
```

### Step 2: Download raw data

Place the following files into `Final_Workflow/Data_Prep/Data/`.  
Download from: https://www.data.gov.uk/dataset/cb7ae6f0-4be6-4935-9277-47e5ce24a11f/road-safety-data

```
dft-road-casualty-statistics-collision-1979-latest-published-year.csv
dft-road-casualty-statistics-casualty-1979-latest-published-year.csv
dft-road-casualty-statistics-vehicle-1979-latest-published-year.csv
dft-road-casualty-statistics-collision-provisional-2025.csv
dft-road-casualty-statistics-casualty-provisional-2025.csv
dft-road-casualty-statistics-vehicle-provisional-2025.csv
```

Two lookup files are already committed to the repo and do **not** need downloading:
- `local-authority-ons-district-names.csv`
- `ons_la_population_2024.csv`

### Step 3: Run the data preparation pipeline

Open Jupyter and navigate to `Final_Workflow/Data_Prep/`.  
Open `data_cleaning.ipynb`, restart the kernel, and run all cells.

Produces four cleaned CSVs in `Final_Workflow/Data_Prep/Cleaned/`:

| File | Description |
|---|---|
| `collisions_clean.csv` | One row per collision |
| `casualties_clean.csv` | One row per casualty |
| `vehicles_clean.csv` | One row per vehicle |
| `cas_full.csv` | One row per casualty with vehicle and collision context joined in — **primary analysis file** |

### Step 4: Run the analysis pipeline

Open Jupyter and navigate to `Final_Workflow/Data_Analysis/`.  
Open `data_analysis.ipynb`, restart the kernel, and run all cells.

Produces all figures, tables, traceability table, run metadata, and findings page in `Final_Workflow/Data_Analysis/Outputs/`.

A successful run ends with:
```
Run complete.
Outputs written to: .../Final_Workflow/Data_Analysis/Outputs/
```

---

## Configurable Parameters

### Data preparation (`data_cleaning.ipynb` — §0 Config cell)

| Parameter | Default | What it does |
|---|---|---|
| `YEAR_START` | `2014` | First year included from the historical file |
| `YEAR_END` | `2024` | Last confirmed published year |
| `INCLUDE_PROVISIONAL` | `True` | Whether to append the provisional year file |
| `PROVISIONAL_YEAR` | `2025` | Which provisional file to load |

No other cells need editing. All paths are relative and built automatically from `os.getcwd()`.

### Analysis (`data_analysis.ipynb` — §0 Config cell)

| Parameter | Default | What it does |
|---|---|---|
| `BASE_WINDOW_ANCHOR` | `(2015, 2017)` | Pre-COVID base comparison window |
| `RECENT_WINDOW_ANCHOR` | `(2022, 2024)` | Recent comparison window |
| `RECENT_N` | `3` | Window size if anchors set to `None` |
| `MIN_KSI_BASE` | `30` | Minimum mean KSI for LA to be included in OLS trend |
| `MIN_YEARS` | `6` | Minimum years of data required per LA for OLS |
| `LOW_R2` | `0.5` | R² threshold below which trend is flagged as non-linear |
| `TOP_N_WORSENING` | `5` | Top N worsening LAs shown |
| `TOP_N_IMPROVING` | `5` | Top N improving LAs shown |
| `HOTSPOT_N` | `3` | Number of most recent confirmed years used for hotspot analysis |
| `HOTSPOT_TOP_N` | `15` | Top N LAs in hotspot rankings |
| `MIN_COLLISIONS` | `500` | Minimum collisions for LA to appear in rate-based ranking |
| `MATERIAL_CHANGE_PCT` | `15` | % KSI change threshold for material change flag |
| `N_BOOTSTRAP` | `1000` | Bootstrap iterations for CI robustness check |
| `COVID_YEARS` | `[2020, 2021]` | Years excluded from trend fitting |

**Window anchoring:** when the cleaned data extends past an anchor's end year, both windows shift forward by the same offset to preserve the COVID gap. Set anchors to `None` for fully auto-derived windows.

---

## Analysis Sections (`data_analysis.ipynb`)

| Section | Description | Outputs |
|---|---|---|
| §0 | Config and path setup | — |
| §1 | Load data, schema assertion, window resolution, completeness filter | — |
| §2 | Data quality — missingness and IBRS adoption | Fig 1, tab_null_rates_by_year.csv, tab_ibrs_adoption.csv |
| §3 | KSI trends — seasonality, national trend, raw vs IBRS-adjusted | Figs 2–4, tab_ksi_trend_national.csv, tab_ksi_raw_vs_adjusted.csv |
| §4 | Road user severity profile | Fig 5, tab_road_user_severity.csv |
| §5 | Geographic hotspots — count, rate, choropleth, rank comparison, road user | Figs 6, 6b, 6c, 6d, tab_la_hotspots_*.csv |
| §6 | Contributory factors — urban/rural, speed/road type, day/hour, junction | Figs 7–9, tab_ksi_by_*.csv |
| §7 | LA trend analysis — OLS per LA, worsening/improving, factor breakdown | Figs 10–11, tab_la_ols_results.csv, tab_la_worsening.csv, tab_la_improving.csv |
| §8 | Robustness checks | Fig 12, tab_la_robustness_stability.csv |
| §9 | Diagnostics and modelling — OLS diagnostics, bootstrap, stepwise GLM, AIC/BIC | Figs 13–14, tab_residual_diagnostics.csv, tab_bootstrap_ci_comparison.csv |
| §10 | Policy evaluation | — |
| §11 | Material change threshold | Fig 17, tab_material_change_flagged.csv |
| §12 | Traceability and run metadata | traceability.csv, run_metadata.json |
| §13 | Findings page | findings.html |

---

## Verifying the Data Pipeline (Optional)

After running `data_cleaning.ipynb`, verify the outputs by running:

```
Quality_Assurance/QA_Data_Load.ipynb
```

This runs 32 automated checks on the four cleaned CSVs. **31/32 are expected to pass.**  
The one known fail (`collisions_clean.csv` contains 2,771 duplicate rows inherited from the raw DfT source file) does not affect `cas_full.csv`, which is the sole input to the analysis pipeline and passed all checks with 0 duplicates.

---

## Reproducibility Notes

- Always restart the kernel before a full re-run to guarantee a clean state
- All paths are relative — no editing required on any machine
- The year axis, LA list, and ranked candidates are all derived from the data at runtime; nothing is hardcoded except the configurable constants in §0
- The choropleth (Fig 6b) fetches ONS LAD boundary GeoJSON from a public ArcGIS endpoint — an internet connection is required for that figure
- `ons_la_population_2024.csv` is used for the three-way exposure robustness check; if absent from `Data_Prep/Data/`, that check is skipped gracefully
- Provisional data is tagged `provisional = True` throughout; filter with `col[~col['provisional']]` if confirmed years only are needed

---

## Dependencies

All dependencies are in `requirements.txt` at the repo root. Install once with:

```bash
pip install -r requirements.txt
```

| Package | Used in |
|---|---|
| `pandas` | data_cleaning.ipynb, data_analysis.ipynb, QA_Data_Load.ipynb |
| `numpy` | data_cleaning.ipynb, data_analysis.ipynb, QA_Data_Load.ipynb |
| `matplotlib` | data_analysis.ipynb |
| `scipy` | data_analysis.ipynb |
| `statsmodels` | data_analysis.ipynb |
| `geopandas` | data_analysis.ipynb (choropleth) |

---

## Team Working Evidence

All project management evidence is in `Group_Contact/`:

- `Action_Log/action_log.md` — every task logged with owner, deadline, status, and evidence pointer (PR/commit/issue)
- `Meeting_Minutes/` — minutes from all 6 team meetings

The management plan (workplan, risk register, QA plan, reproducibility record) is in `Management_Plan/management_plan.md`.

---

*The final reproducible analysis is in `Final_Workflow/` only.*