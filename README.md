# fuel-amount-prediction
Public Transit Fuel Consumption Analysis and Prediction

# NTD Transit Fuel & Energy Consumption : Data Cleaning & GGE Conversion

**Part 1: Introduction, Dataset Description, Cleaning & Reshaping**

## 1. Project Overview

U.S. public transit agencies report annual fuel and energy consumption to
the Federal Transit Administration (FTA) as part of the **National Transit
Database (NTD)**. This project uses the **2023 NTD release** to prepare a
clean, analysis-ready dataset that will later support a regression model
predicting an agency's fuel/energy consumption based on service type, mode
of transport, and other operating details.

This first part of the project covers **data loading, cleaning, reshaping,
and unit conversion** — it does not yet build the regression model itself.

## 2. Repository Structure

```
NTD_Fuel_Project/
├── README.md
└── notebooks/
    └── fuel data cleaning.ipynb
```

> **Note:** The raw data file itself (`8ehq-7his.csv.xlsx`) can be downloaded from: https://data.transportation.gov/Public-Transit/2022-2024-NTD-Annual-Data-Fuel-and-Energy/8ehq-7his/about_data

## 3. Dataset Description

- **Source:** FTA National Transit Database (NTD), 2023 reporting year
- **Raw shape:** 1,237 rows × 65 columns
- **Grain:** one row per **transit agency + mode of service** combination

Raw columns fall into four groups:

| Group | Examples |
|---|---|
| Agency / admin identifiers | `agency`, `city`, `state`, `ntd_id`, `organization_type`, `report_year`, `uace_code`, `primary_uza_population`, `mode_name`, `typeofservicecd` |
| Fuel-amount columns (9 fuel types) | `diesel_gal`, `gasoline_gal`, `liquefied_petroleum_gas_gal`, `compressed_natural_gas_gal`, `bio_diesel_gal`, `hydrogen_kg_`, `other_fuel_gal_gal_equivalent`, `electric_propulsion_kwh`, `electric_battery_kwh` |
| Plain-named fuel-indicator columns | e.g. `diesel`, `gasoline`, `hydrogen` — a second, redundant set of fuel columns carried over from the raw NTD export |
| Fuel-efficiency columns (9 fuel types) | `diesel_mpg`, `gasoline_mpg`, `liquefied_petroleum_gas_mpg`, `compressed_natural_gas_mpg`, `hydrogen_mpkg_`, `other_fuel_mpg`, `electric_propulsion_mi_kwh`, `electric_battery_mi_kwh` |



## 4. Cleaning & Reshaping Pipeline

The notebook processes the raw file through the following stages:

### Step 1 — Drop fully-empty columns
24 columns (mostly `*_questionable` flags and duplicate `*_1` columns) are
100% null and are dropped.
`65 → 41 columns`

### Step 2 — Drop identifier / redundant columns
| Column | Reason dropped |
|---|---|
| `ntd_id` | Redundant — `agency` already identifies the agency |
| `reporter_type` | Administrative category, unrelated to fuel/energy use |
| `uza_name` | Redundant with `uace_code` |
| `modecd` | Redundant with `mode_name` |

`41 → 37 columns`

### Step 3 — Melt fuel-amount columns
The 9 fuel-amount columns are melted into a long/tidy table
(`df_fuel_amount`) with a human-readable `Fuel Type` label and a derived
`Unit of Fuel` (`gal`, `kg`, or `kwh`). Rows with no reported amount are
dropped, and duplicates are removed.
**Result:** 2,942 rows.

Fuel type row counts after this step:

| Fuel Type | Rows reported |
|---|---|
| Electric Battery | 1,237 |
| Gasoline | 707 |
| Diesel | 558 |
| CNG | 209 |
| Electric Propulsion | 90 |
| Biodiesel | 73 |
| LPG | 54 |
| Hydrogen | 9 |
| Other Fuel | 5 |

### Step 4 — Melt efficiency columns
The 8 efficiency columns are melted separately into `df_efficiency`,
mapped to the same `Fuel Type` labels, with an `Efficiency Unit`
(`mpg`, `mpkg`, or `kwh`) derived. Rows with no reported efficiency are
dropped. Note: **Biodiesel has no matching efficiency column** in the raw
data — this is expected, not a bug.
**Result:** 1,656 rows.

### Step 5 — Left-join efficiency onto fuel amounts
`df_fuel_amount` and `df_efficiency` are joined on the shared agency/mode/
`Fuel Type` keys, keeping **every** fuel-amount row. Efficiency is
attached where reported and left as `NaN` where it wasn't (`NaN` means "no
efficiency reported," not "drop this row").
**Result (`df_fuel`):** 2,942 rows × 33 columns, with 1,608 rows carrying
an attached efficiency value and 1,334 without one.

### Row/Column Count Through Every Stage

| Stage | Rows | Columns |
|---|---:|---:|
| Raw file | 1,237 | 65 |
| After dropping empty & identifier columns | 1,237 | 37 |
| Fuel-amount table (melted) | 2,942 | 31 |
| Efficiency table (melted) | 1,656 | 32 |
| Final tidy table (fuel + efficiency joined) | 2,942 | 33 |

## 5. Gasoline-Gallon-Equivalent (GGE) Conversion

`Amount Used` mixes incompatible units across fuel types (gallons,
kilograms, kWh), so totals can't be compared directly until everything is
put on a common basis. The notebook converts each fuel type on the cleaned
**wide** table to its Gasoline-Gallon-Equivalent (GGE) value:

| Fuel type column | GGE factor |
|---|---:|
| `diesel_gal` | 1.12 |
| `gasoline_gal` | 1.00 |
| `liquefied_petroleum_gas_gal` | 0.74 |
| `compressed_natural_gas_gal` | 1.00 |
| `bio_diesel_gal` | 1.05 |
| `hydrogen_kg_` | 1.00 |
| `other_fuel_gal_gal_equivalent` | 1.00 |
| `electric_propulsion_kwh` | 0.030 |
| `electric_battery_kwh` | 0.030 |

Rules applied during conversion:
- Values are coerced to numeric; non-numeric entries become `NaN`.
- **Negative reported amounts are treated as missing** (not physically
  meaningful) rather than converted.
- A `Total_GGE` column sums all converted fuel columns per row (skipping
  `NaN`s).

**Result (`df_gge`):** 1,237 rows × 47 columns.

The GGE columns are then melted into a long **tidy GGE table**
(`df_gge_fuel`, 2,942 rows) so every fuel type is on the same basis and
directly comparable across agencies, states, and modes for EDA.

## 6. Key Output Tables

| Variable | Description | Shape |
|---|---|---|
| `df` | Cleaned wide table after Steps 1–2 | 1,237 × 37 |
| `df_fuel_amount` | Tidy fuel-amount table | 2,942 × 31 |
| `df_efficiency` | Tidy efficiency table | 1,656 × 32 |
| `df_fuel` | Fuel amounts + left-joined efficiency | 2,942 × 33 |
| `df_gge` | Wide table with GGE-converted fuel columns + `Total_GGE` | 1,237 × 47 |
| `df_gge_fuel` | Tidy, GGE-normalized table ready for EDA | 2,942 rows |

## 7. How to Run

**Requirements**
```
numpy
pandas
matplotlib
seaborn
scikit-learn
```

**Steps**
1. fuel data cleaning.ipynb
2. Run all cells top to bottom — each stage depends on the output of the
   one before it (raw load → drop columns → melt fuel → melt efficiency →
   join → GGE conversion → tidy GGE table).

## 8. Known Data Quirks

- **Duplicate fuel columns:** the raw export contains both a suffixed
  amount column (e.g. `diesel_gal`) and a plain-named column (e.g.
  `diesel`). The plain-named columns are *not* part of `FUEL_COLUMNS` and
  are carried through as extra context rather than melted.
- **`electric_battery_kwh` is fully populated (1,237/1,237 non-null)**
  but with mostly zero values, unlike the other fuel columns which are
  sparse worth checking before treating it as "usage reported" during
  EDA.
- **Biodiesel has no efficiency (`mpg`) column** in the raw data, so it
  will always show `NaN` efficiency after the join.
- **`compressed_natural_gas_mpg_1`** survives as a duplicate-looking
  column in the final tidy GGE table (`df_gge_fuel`) — worth confirming
  whether it should have been dropped alongside the other `_1`/
  `_questionable` columns in Step 1.

## 9. Next Steps

This notebook only covers **Part 1** (cleaning, reshaping, unit
conversion). Suggested follow-on work:
1. **EDA** on `df_gge_fuel` 
3. **Regression modeling** `GGE` regression model against agency/mode features.
