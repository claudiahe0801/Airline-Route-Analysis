# ✈️ Airline Route Analysis - README

## 📦 What We Have

We are working with raw data from **three datasets**:

* **Flights**: Q1 2019 flight data — includes date, origin, destination, distance, flight number, air time, occupancy rate
* **Tickets**: Q1 2019 ticket data — includes itinerary details and fare information
* **Airports**: Airport metadata — includes airport code, city, country, coordinates

## 🎯 Objective

To identify and recommend the **top 5 round-trip airline routes** to invest in, based on:

* Profitability
* Operational efficiency
* Reliability

We work under the scenario of helping an airline company evaluate opportunities to enter the US domestic market.

---

## 🧹 Step 1: Data Cleaning

### ✅ Data Type Validation

We started with identifying and converting incorrect data types. For instance:

| Column     | Current Type | Expected Type | Issue                      | Action                                 |
| ---------- | ------------ | ------------- | -------------------------- | -------------------------------------- |
| AIR\_TIME  | object       | float64       | Non-numeric/missing values | Remove non-numeric characters, convert |
| DISTANCE   | object       | float64       | Same                       | Same                                   |
| FL\_DATE   | object       | datetime64    | Needs format conversion    | Use `errors='coerce'` to clean         |
| ITIN\_FARE | object       | float64       | Contains "\$", ",", etc.   | Remove symbols, convert to float       |

Used regex to identify non-numeric characters:

```python
str_series = series.astype(str)
non_numeric = str_series.str.findall(r'[^0-9.-]')
```

Cleaned issues include:

* `$`, `***`, `nan`, spelled-out numbers ("Twenty")
* Converted missing values to `pd.NA`

---

## 🔍 Step 2: Filter Business-Relevant Data

* Remove canceled flights
* Keep only relevant airports (medium/large)

Benefits:

* Reduce size for faster processing
* Prevent misleading outliers

---

## 📊 Step 3: Outlier Detection & Quality Check

* Detected:

  * Negative flight times/distances
  * Outrageous fares (\$38,400), 681 passengers per ticket
  * Unrealistic delays (49 hours)
* Used IQR method and scatter plots (e.g. air\_time vs distance, correlation = 0.97)

---

## 🛫 Step 4: Route\_ID Generation

### Why?

To standardize round-trip routes: JFK → ORD and ORD → JFK = `JFK-ORD`

### Assumptions:

* Profit and demand are symmetric for round-trip directions
* No missing legs in route pair
* Direct flights only

Checked that airport codes in `Flights` and `Tickets` exist in `Airports` dataset before proceeding.

---

## 🧩 Step 5: Handling Missing Values

### Key Columns:

* AIR\_TIME, DISTANCE, ITIN\_FARE, PASSENGERS

### Strategy:

* Group by `Route_ID`
* If group has valid values, fill missing with median
* If mostly missing → drop the route

### Assumptions:

* Data per route is normally/symmetrically distributed
* Median is preferred over mean (handles skewed data, avoids outlier influence)

### Special Handling:

* If both `ARR_DELAY` and `DEP_DELAY` missing, set `ARR_DELAY = 0`
* Delete `FL_DATE` and `OCCUPANCY_RATE` due to low coverage or better usage elsewhere

---

## 🔗 Step 6: Table Joining

### Feasibility Check:

* **Route Overlap**: \~99% overlap between Flights and Tickets
* **Time Alignment**: Both datasets = Q1 2019
* **Airline Coverage**: 20/26 (Flights), 20/21 (Tickets)

### Aggregation Level:

* Granularity = Year, Quarter, Route\_ID

### Aggregation Methods:

```python
{
  'DISTANCE': 'first',
  'AIR_TIME': 'mean',
  'DEP_DELAY': 'mean',
  'OCCUPANCY_RATE': 'mean',
  'FL_DATE': 'count'  # total flights = route activity
}
```

### Joining Strategy:

* Flights + Airports (origin/dest) = Left Join
* Tickets + Airports = Left Join
* Final: Flights + Tickets = Inner Join on (Route\_ID, Year, Quarter)

---

## 📈 Key Tasks

### Task 1: Busiest Round-Trip Route

* Based on total number of round-trips per route

### Task 2: Most Profitable Round-Trip Route

* Initial Metric: Total Profit → favors high-demand routes
* Final Metric: **Avg Profit per Flight per Route per Quarter**

  * Balanced supply/demand view
  * Revenue = (avg\_fare / 2) \* passengers

### Task 3: Multi-Factor Route Evaluation

* Considerations:

  * Total Profit & Margin
  * Flight Frequency (Market Size)
  * Occupancy Rate (Service & Competency)
  * Delay Control (Risk & Customer Impact)

### Task 4: Investment Recommendation

* Assumption: One plane per route → \$90M × 5 = \$450M
* Assumes equal seasonal profit; future steps could add seasonality and discount rate factors
* Must validate round-trip balance (e.g., JFK → ORD = ORD → JFK)

---

## 📝 Future Improvements

* Consider seasonality and dynamic demand
* Refine route balancing
* Introduce NPV/ROI calculations for aircraft investment

---

## 📂 File Structure (suggested)

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
```

---

Feel free to open issues or contribute to further route evaluation logic!
