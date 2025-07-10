# ✈️ Airline Route Analysis - README

## Overview

This project analyzes U.S. domestic flight data to identify the top 5 round-trip airline routes worth investing in. The selection is based on profitability, operational efficiency, and reliability.

## Data Sources

We use three datasets from Q1 2019:

* **Flights**: Flight details such as date, origin, destination, distance, flight number, air time, and occupancy rate
* **Tickets**: Sample ticket data including fare and passenger counts
* **Airports**: Airport codes and locations (city, country, coordinates)

## Goal

Help an airline evaluate potential domestic routes by identifying high-performing round-trip markets.

---

## Step 1: Data Cleaning

We first cleaned the data to ensure accuracy and compatibility:

* Fixed incorrect data types (e.g., converted strings to numeric types)
* Removed non-numeric characters (e.g., "\$", commas, spelled-out numbers)
* Standardized missing values using `pd.NA`

Example fix using regular expressions:

```python
str_series = series.astype(str)
non_numeric = str_series.str.findall(r'[^0-9.-]')
```

---

## Step 2: Filter Relevant Records

* Removed canceled flights
* Kept only flights between medium and large airports
* Reduced data size and excluded outliers caused by irrelevant records

---

## Step 3: Outlier & Quality Check

Using IQR and scatter plots, we identified:

* Negative or unrealistic values (e.g., 49-hour delays, \$38,000 fares)
* Correlation between flight time and distance (r = 0.97), used to validate outlier detection

---

## Step 4: Route\_ID Generation

We standardized round trips using a unified `Route_ID`, combining outbound and return flights (e.g., JFK-ORD).

Assumptions:

* Round-trip directions are symmetric in demand and revenue
* All flights are direct
* Airports match across datasets

---

## Step 5: Handling Missing Values

Key columns: `AIR_TIME`, `DISTANCE`, `ITIN_FARE`, `PASSENGERS`

Approach:

* Grouped by `Route_ID`
* Imputed missing values using group median (if sufficient data)
* Dropped routes with excessive missing data

Special cases:

* Filled missing arrival delays based on departure delays or set to 0
* Dropped `FL_DATE` and `OCCUPANCY_RATE` if coverage was too low

---

## Step 6: Merging Datasets

We joined the datasets on `(Route_ID, Year, Quarter)` after verifying:

* \~99% route overlap
* Time alignment (Q1 2019)
* Airline coverage in both Flights and Tickets datasets

Aggregation level:

```python
{
  'DISTANCE': 'first',
  'AIR_TIME': 'mean',
  'DEP_DELAY': 'mean',
  'OCCUPANCY_RATE': 'mean',
  'FL_DATE': 'count'  # Number of flights per route
}
```

Join Strategy:

* Flights + Airports (origin/destination): Left Join
* Tickets + Airports: Left Join
* Flights + Tickets: Inner Join on Route\_ID, Year, Quarter

---

## Analysis Tasks

### 1. Busiest Route

* Based on total number of round-trip flights

### 2. Most Profitable Route

* Initial approach: total profit
* Refined approach: average profit per flight per quarter

  * Revenue = (average fare / 2) × passengers

### 3. Multi-Factor Evaluation

Metrics considered:

* Profit and profit margin
* Flight frequency (demand)
* Occupancy rate (efficiency)
* Delay rates (reliability and cost)

### 4. Investment Recommendation

* Assumed 1 aircraft per route (\$90M each → \$450M total)
* Did not adjust for seasonality (future work)
* Ensured route symmetry (equal outbound/inbound flights)

---

## Future Improvements

* Factor in seasonality and time trends
* Refine route symmetry validation
* Incorporate NPV/ROI and discounting in investment planning

---

## Suggested File Structure

```bash
project_root/
├── data/
│   ├── flights.csv
│   ├── tickets.csv
│   └── airports.csv
├── notebooks/
│   ├── 01_data_cleaning.ipynb
│   ├── 02_outlier_detection.ipynb
│   └── 03_route_analysis.ipynb
├── output/
│   ├── cleaned_data.csv
│   └── route_summary.csv
└── README.md

