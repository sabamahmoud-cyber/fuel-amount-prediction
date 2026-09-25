# NTD Transit Fuel — Exploratory Data Analysis (EDA)

**Part 2: Exploratory Data Analysis**

## 1. Overview

This notebook (`fuel_eda.ipynb`) picks up after the fuel data cleaning
notebook (`fuel data cleaning.ipynb`) and explores fuel consumption
patterns across the 2023 National Transit Database (NTD) using the
cleaned, GGE-normalized dataset. Because every fuel type has already been
converted to a common **Gasoline-Gallon-Equivalent (GGE)** basis, totals
can be summed and compared directly across fuel types, states, and
transit modes without unit-mismatch issues.

## 2. Input Data

| | |
|---|---|
| **File loaded** | `cleaned_dataset.csv.xlsx` |
| **Shape** | 2,942 rows × 34 columns |
| **Grain** | one row per agency + mode + fuel type (long/tidy format) |
| **Columns used in this notebook** | `agency`, `state`, `mode_name`, `Fuel Type`, `Amount_Original`, `GGE` |

This is the same tidy table produced at the end of the cleaning notebook
(referred to there as `df_gge_fuel`) — every row is a reported fuel amount
for one agency/mode/fuel-type combination, with `GGE` already
unit-converted and ready to aggregate.

## 3. Notebook Structure

| Section | Purpose |
|---|---|
| 1. Setup | Imports `pandas`, `numpy`, `matplotlib`, `seaborn` |
| 2. Load Cleaned Data | Reads `cleaned_dataset.csv.xlsx` into `df` |
| EDA 1 — Fuel Amount by Type | Total GGE consumption by fuel type |
| EDA 2 — Top States | Top 15 states by total GGE consumption |
| EDA 3 — Consumption by Mode | Total GGE consumption by transit mode |
| EDA 4 — Fuel Mix by Mode | Fuel-type breakdown within each transit mode |

## 4. EDA 1 — Fuel Amount by Type

**What it does:** groups `df` by `Fuel Type`, sums `GGE`, sorts
descending, and plots a horizontal bar chart.

**Result (total GGE by fuel type):**

| Fuel Type | Total GGE | Share of total |
|---|---:|---:|
| Diesel | 480,994,200 | 48.9% |
| Electric Propulsion | 181,829,200 | 18.5% |
| CNG | 181,123,600 | 18.4% |
| Gasoline | 90,208,560 | 9.2% |
| Biodiesel | 35,965,380 | 3.7% |
| LPG | 6,341,363 | 0.6% |
| Other Fuel | 5,258,123 | 0.5% |
| Electric Battery | 2,141,713 | 0.2% |
| Hydrogen | 328,216 | <0.1% |

<img width="889" height="490" alt="image" src="https://github.com/user-attachments/assets/60aaa3fc-48b4-4bd0-9ee1-e11e14ff60c2" />

**Takeaway:** Diesel accounts for nearly half of all reported fuel
consumption (GGE-basis) across U.S. transit agencies. Electric
Propulsion (third-rail/overhead-wire systems) and CNG are essentially
tied for a distant second, together making up another ~37%.




## 5. EDA 2 — Top States

**What it does:** groups by `state`, sums `GGE`, sorts descending, and
takes the top 15 (`TOP_N_STATES = 15`).

**Result (top 15 states by total GGE):**

| Rank | State | Total GGE |
|---:|---|---:|
| 1 | NY | 179,582,700 |
| 2 | CA | 156,344,500 |
| 3 | IL | 70,040,040 |
| 4 | NJ | 65,742,500 |
| 5 | TX | 56,171,990 |
| 6 | FL | 47,626,200 |
| 7 | WA | 47,592,490 |
| 8 | PA | 40,721,150 |
| 9 | MA | 40,616,320 |
| 10 | DC | 30,252,330 |
| 11 | MD | 22,830,120 |
| 12 | AZ | 19,665,260 |
| 13 | OH | 17,325,570 |
| 14 | VA | 15,126,750 |
| 15 | GA | 14,683,010 |


<img width="889" height="590" alt="image" src="https://github.com/user-attachments/assets/33724920-af27-4cf0-ac62-d07ff47e2e7e" />



**Takeaway:** New York and California are far ahead of every other
state, together accounting for roughly a third of all reported fuel
consumption — consistent with them hosting the largest, highest-ridership
transit agencies (MTA, LA Metro, etc.).

## 6. EDA 3 — Consumption by Mode

**What it does:** groups by `mode_name`, sums `GGE`, sorts descending.

**Result (total GGE by transit mode, full list):**

| Mode | Total GGE |
|---|---:|
| Bus | 513,399,800 |
| Commuter Rail | 159,689,700 |
| Heavy Rail | 101,607,900 |
| Demand Response | 87,636,210 |
| Ferryboat | 51,712,320 |
| Light Rail | 26,353,400 |
| Commuter Bus | 23,881,610 |
| Vanpool | 9,138,508 |
| Bus Rapid Transit | 3,837,449 |
| Hybrid Rail | 1,987,849 |
| Streetcar Rail | 1,517,319 |
| Trolleybus | 1,364,336 |
| Alaska Railroad | 1,090,314 |
| Monorail/Automated Guideway | 518,580 |
| Publico | 352,041 |
| Cable Car | 81,209 |
| Inclined Plane | 11,050 |
| Aerial Tramway | 10,836 |


<img width="889" height="590" alt="image" src="https://github.com/user-attachments/assets/37dcd749-644a-491c-ad72-48b7983f4cfd" />


**Takeaway:** Bus service alone accounts for over half of all reported
fuel/energy consumption — unsurprising given how much of U.S. transit
service (and mileage) runs on buses. Rail modes (Commuter + Heavy +
Light) together make up roughly 29%.

## 7. EDA 4 — Fuel Mix by Mode

**What it does:** builds a pivot table of `GGE` summed by `mode_name` ×
`Fuel Type`, reorders rows to match the mode ranking from EDA 3, then
plots a normalized (share-of-total) stacked horizontal bar chart so each
mode's fuel mix is comparable regardless of its overall size.


<img width="985" height="690" alt="image" src="https://github.com/user-attachments/assets/34c96c20-88c7-4be0-8fd1-60238308ea02" />


**Selected findings from the pivot table:**

| Mode | Dominant fuel type(s) |
|---|---|
| Bus | Diesel (~304M) and CNG (~173M) — together ~93% of Bus's GGE |
| Commuter Rail | Diesel (~101M) and Electric Propulsion (~50M) |
| Heavy Rail | 100% Electric Propulsion (~102M) — no other fuel type reported |
| Demand Response | Gasoline (~73M) is dominant, not diesel — the opposite pattern from Bus |
| Ferryboat | Diesel (~50M) is essentially the only fuel type used |

**Takeaway:** the overall fuel mix (Diesel-heavy) masks real differences
between modes — Heavy Rail is fully electrified, Demand Response runs
mostly on gasoline rather than diesel, and CNG usage is concentrated
almost entirely in Bus service.

## 8. How to Run

**Requirements**
```
pandas
numpy
matplotlib
seaborn
openpyxl   # needed by pandas to read .xlsx files
```

**Steps**
1. GGE fuel eda.ipynb 
  
2. Run all cells top to bottom. Each EDA section depends on `df`







## 9. Next Steps

Going to modeling 
   (`GGE` as the target, agency/mode features as predictors).

