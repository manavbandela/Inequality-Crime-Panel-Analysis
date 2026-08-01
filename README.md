# Does Income Inequality Drive Crime Rates?

A global panel-data analysis (2006–2023) of the relationship between income
inequality (Gini coefficient) and intentional homicide rates, with a
secondary test of whether democratic institutions moderate that
relationship.

**Authors:** Ankita Ankita, Manav Bandela

## Overview

This project builds a country-year panel dataset from four public sources
and estimates a series of panel regression models (pooled OLS, country
fixed effects, and two-way fixed effects) to test:

- **H1:** Higher income inequality is associated with higher homicide rates.
- **H2:** The strength of the inequality–crime relationship is moderated by
  a country's level of democracy.

All models cluster standard errors at the country level to account for
serial correlation within countries over time.

## Data Sources

| Variable | Source | File |
|---|---|---|
| Intentional homicide rate (per 100,000 population) | UNODC Statistics | `data/homicide_data.xlsx` |
| Gini coefficient | World Bank Poverty & Inequality Platform (PIP) | `data/gini_Data.csv` |
| GDP per capita (constant 2015 USD) | World Bank WDI (`NY.GDP.PCAP.KD`) | `data/gdppc_data.csv` |
| Population | World Bank WDI (`SP.POP.TOTL`) | `data/pop_data.csv` |
| Democracy Index (0–10) | EIU, via World Bank DATA360 | `data/DI.csv` |

Variable metadata for the World Bank series is included in
`data/gdppc_metadata/` and `data/pop_metadata/`.

## Methodology

1. **Load & clean** each raw source, restricting to 2006–2023.
2. **Reshape** wide World Bank series to long (country–year) format.
3. **Merge** all sources into a single panel keyed on ISO3 country code and year.
4. **Interpolate** the Gini coefficient within-country (linear interpolation
   only, never extrapolation), since inequality surveys are irregular.
5. **Transform**: log-transform homicide rate and GDP per capita (both
   right-skewed); mean-center Gini and democracy before interacting them.
6. **Sample restrictions**: drop microstates (population < 500,000), drop
   country-years without an interpolated Gini value, drop rows missing GDP
   or democracy, and drop countries whose largest gap between *actual*
   Gini observations exceeds 5 years (to avoid over-relying on interpolation).
7. **Estimate** four panel models via `plm`: pooled OLS, country fixed
   effects, two-way (country + year) fixed effects, and two-way FE with a
   Gini × Democracy interaction term. A Hausman test compares the
   fixed-effects vs. random-effects specification.
8. **Robustness check**: re-estimate the main model excluding extreme
   homicide-rate outliers (> 60 per 100,000).

## Repository Structure

```
.
├── inequality_crime_analysis.Rmd     # Full analysis: data cleaning → models → figures/tables
├── data/                             # Raw input data (as downloaded from source)
│   ├── homicide_data.xlsx
│   ├── gini_Data.csv
│   ├── gini_Series - Metadata.csv
│   ├── gdppc_data.csv
│   ├── gdppc_metadata/
│   ├── pop_data.csv
│   ├── pop_metadata/
│   └── DI.csv
├── output/
│   ├── panel_clean.csv               # Final cleaned analysis panel
│   ├── inequality_crime_analysis.html # Knitted report (full output)
│   ├── figures/                      # Exported plots (histograms, scatter, panel coverage)
│   └── tables/                       # Descriptive stats & regression tables (HTML/Word)
└── paper/
    └── Research_Paper_inequality_crime.pdf / .docx   # Final write-up
```

## Requirements

R (≥ 4.2 recommended) with the following packages:

```r
install.packages(c(
  "tidyverse", "readxl", "plm", "lmtest", "sandwich",
  "modelsummary", "ggplot2", "countrycode", "zoo",
  "scales", "gt", "kableExtra"
))
```

The `.Rmd` file installs any missing packages automatically on first run.

## Reproducing the Analysis

1. Clone the repo and open it in RStudio (or set it as your working directory).
2. Ensure the `data/` folder is present at the project root (paths in the
   script are relative, e.g. `data/homicide_data.xlsx`).
3. Knit `inequality_crime_analysis.Rmd`, or run it chunk by chunk.
4. Outputs (cleaned panel, figures, regression tables) are written to the
   project root by default — see the script for exact filenames — matching
   what's stored under `output/` in this repo.

## Results

See `paper/Research_Paper_inequality_crime.pdf` for the full write-up and
discussion, and `output/tables/regression_table.html` (or `.docx`) for the
formatted regression table (Table 2: pooled OLS, country FE, two-way FE,
and two-way FE + interaction, with cluster-robust standard errors).

## License

Add a license of your choice (e.g. MIT) if you want others to be able to
reuse this code.
