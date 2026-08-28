# Solar Energy Potential Analysis Across Alberta

A SAIT Data Analytics capstone project analyzing solar energy potential across four Alberta cities, combining 20 years of historical weather data with real SAIT campus solar generation data, machine learning, physics-based PV simulation, and a Power BI dashboard.

**Team:** Abiola Abioye, Samuel Aduaka, Prutha Chokshi — SAIT Data Analytics Capstone, 2026

**Client framing:** Alberta Utilities Commission (AUC)

## Problem

Alberta has strong but geographically and seasonally variable solar resources, while raw hourly engineering-weather records aren't directly usable by non-technical decision-makers. This project converts station-level irradiance and weather data into city comparisons, predictive evidence, installation-level yield estimates, financial sensitivity ranges, and long-term trend information for the AUC, municipalities, solar developers, and Alberta households/businesses.

## Analytical Questions

1. How do GHI and related solar-resource measures differ across Calgary, Edmonton, Lethbridge, and Medicine Hat?
2. Can hourly GHI be predicted from geography, meteorology, and cyclical time information with out-of-sample R² ≥ 0.85?
3. What fixed panel tilt maximizes annual plane-of-array irradiance, and what annual energy/payback follows for 10 kWp and 100 kWp systems?
4. Do annual GHI records show statistically significant trends from 1998–2017?
5. Do historical Calgary solar patterns remain consistent with the real SAIT campus generation profile, given available data coverage?

## Data Sources

| Dataset | Coverage | Rows | Role |
|---|---|---|---|
| CWEEDS — 4 Alberta stations | 1998-01-01 to 2017-12-31 | 701,280 combined | EDA, ML, tilt/POA simulation, trend testing |
| SAIT campus solar | Complete usable hours: 2019-03-08 to 2023-09-19 | 1,048,575 raw → 13,423 complete hours | Campus EDA + climatological consistency check |

## Data Cleaning & Quality Assurance

- Verified exactly 175,320 hourly records per CWEEDS station with zero duplicate timestamps and zero missing hours
- Physical-range audit found zero negative GHI/DNI/DHI values and zero invalid sunshine, sky cover, wind direction, or snow-cover values
- SAIT's raw 1,048,575-row file (irregular 1–60 minute intervals) was aggregated to hourly, with an hour classified "complete" only when it contained all 60 underlying minute observations — 13,423 such hours retained
- 71,293 negative Carport readings and 27 negative Rooftop readings were clipped to zero for PV-generation analysis, with correction counts retained for transparency

## Feature Engineering & Machine Learning

**Target:** hourly Global Horizontal Irradiance (`ghi_wh_m2`)

**Features:** latitude/longitude/elevation, dry-bulb temperature, dew point, wind speed, station pressure, cyclical hour-of-day (`hour_sin`/`hour_cos`) and day-of-year (`doy_sin`/`doy_cos`) encodings, and one-hot city indicators. Same-hour DNI/DHI were deliberately excluded to avoid the model becoming a trivial reconstruction of the target.

**Split:** chronological (not random) to prevent temporal leakage — train 1998–2011 (70%), validation 2012–2014 (15%), test 2015–2017 (15%).

**Model:** Random Forest (150 trees, max depth 25, min leaf size 3, sqrt feature sampling), selected on validation RMSE before a single evaluation on the untouched test set.

| Model | R² | MAE | RMSE |
|---|---|---|---|
| Linear Regression (validation) | 0.6953 | 100.72 | 129.07 |
| Random Forest (validation) | 0.9006 | 38.03 | 73.70 |
| **Random Forest (test)** | **0.9169** | **36.59** | **69.54** |

All four cities individually exceeded R² = 0.90 on test data. Top feature importances: `hour_cos` (0.50), `dry_bulb_temp_c` (0.22), `doy_cos` (0.09).

## PV Optimization, Yield & Payback

Used the `pvlib` library to simulate plane-of-array irradiance for fixed south-facing panels from 20°–60° tilt, with an isotropic sky model and ground albedo of 0.20.

| City | Optimized Tilt | Annual POA (kWh/m²) | Base-Case Payback |
|---|---|---|---|
| Calgary | 42° | 1,624.91 | 13.74 years |
| Edmonton | 42° | 1,537.00 | 14.52 years |
| Lethbridge | 40° | 1,687.64 | 13.23 years |
| Medicine Hat | 39° | 1,659.30 | 13.45 years |

Payback sensitivity modeled across best-case ($2.00/Wp, $0.16/kWh), base-case ($2.50/Wp, $0.14/kWh), and worst-case ($3.00/Wp, $0.12/kWh) scenarios.

## Long-Term Trend Testing

Linear regression and Mann-Kendall trend tests were run on annual mean GHI per city. Only Edmonton showed a statistically significant increasing trend (+2.95 Wh/m² per decade, linear p = 0.023, Mann-Kendall p = 0.030); the other three cities showed positive but non-significant slopes.

## SAIT Real-World Validation

The original plan called for timestamp-aligned validation between Calgary weather predictions and measured SAIT generation. This wasn't possible — the CWEEDS data ends in 2017, while complete SAIT solar coverage begins in 2019, leaving no overlapping period. Instead, a climatological consistency check was used: historical Calgary GHI and SAIT PV generation were each aggregated by month × hour, normalized to comparable scales, and compared by Pearson correlation — returning **r = 0.9686**, supporting strong similarity in the seasonal/diurnal profile (a consistency check, not a timestamp-level predictive validation).

## Power BI Dashboard

A four-page report built on nine pre-aggregated analytical tables exported from the Python pipeline:

1. **City Comparisons** — GHI cards, year slicer, city comparison bar chart, yearly GHI line chart
2. **Seasonal Heatmap** — month-by-city matrix and monthly GHI line chart
3. **Optimization & Payback** — PV-yield table, payback scenario matrix, best/base/worst payback chart
4. **Trends & SAIT** — trend slope table and normalized historical GHI vs. SAIT PV profile chart

## Files

| File | Purpose |
|---|---|
| `01_data_cleaning.ipynb` | CWEEDS & SAIT data cleaning and quality assurance |
| `02_eda.ipynb` | Exploratory data analysis |
| `03_feature_engineering.ipynb` | Cyclical time features, city encoding |
| `04_modeling.ipynb` | Random Forest training, tuning, and evaluation |
| `05_sait_validation.ipynb` | Climatological consistency check vs. SAIT data |
| `06_tilt_optimization.ipynb` | `pvlib` PV simulation, tilt optimization, payback |
| `07_powerbi_exports.ipynb` | Final aggregated exports for Power BI |
| `Capstone_Project.pbix` | Power BI report file |
| `Powerbi_solar_dashboard 1.csv` | Dashboard source export |
| `Solar_Capstone_Presentation.pptx` | Final presentation deck |
| `Deliverable 4 Analysis.docx` / `.pdf` | Full written analysis and verification report |
| `Deliverable 5 Project Presentation.pdf` | Final presentation deliverable |

## Tech

Python (pandas, scikit-learn, `pvlib`), Random Forest regression, chronological time-series validation, Mann-Kendall trend testing, Power BI.
