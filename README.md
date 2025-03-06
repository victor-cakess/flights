# Data Glow Up: flight data analysis

This repository documents a data analysis project centered around flight data. The dataset was sourced from the **Data Glow Up** monthly challenge and spans multiple years of flight information. The primary goals of this project were to:

1. **Ingest and clean raw CSV files** (using Polars for speed).
2. **Store and query** the data efficiently in a DuckDB database (faster OLAP performance than PostgreSQL for this use case).
3. **Perform detailed analyses** on flight volumes, routes, delays, and network centralities.
4. **Visualize** findings using Python libraries (Seaborn, Matplotlib, GeoPandas, NetworkX, etc.).

Below is a high-level overview of the entire workflow, from data ingestion and cleaning to advanced analyses and forecasting.

---

## Table of contents

1. [Project motivation](#project-motivation)  
2. [Tech stack](#tech-stack)  
3. [Data source](#data-source)  
4. [Data ingestion & cleaning](#data-ingestion--cleaning)  
5. [Database setup & merging](#database-setup--merging)  
6. [Exploratory analyses](#exploratory-analyses)  
   - [Flight volumes over time](#flight-volumes-over-time)  
   - [CSV conversion and standardization](#csv-conversion-and-standardization)  
   - [Routes and network analysis](#routes-and-network-analysis)  
   - [Delay analyses](#delay-analyses)  
   - [Seasonality and trends](#seasonality-and-trends)  
7. [Advanced analytics](#advanced-analytics)  
   - [Centrality measures](#centrality-measures)  
   - [Linear regression forecast](#linear-regression-forecast)  
8. [Visualizations](#visualizations)  

---

## Project Motivation

**Data Glow Up** hosts monthly challenges that encourage participants to practice data wrangling, cleaning, and analysis. This project tackles one of those challenges focused on flight data spanning multiple years. The core motivations were:

- **Speed and Efficiency:** Use Polars for rapid CSV ingestion and DuckDB for fast OLAP queries.
- **Comprehensive Analysis:** Investigate flight routes, delays, and volumes from multiple angles.
- **Practical Insights:** Uncover real-world patterns in flight operations, highlight potential optimization areas, and forecast near-future trends.

---

## Tech Stack

1. **[Polars](https://www.pola.rs/):**  
   - Chosen for its speed and memory efficiency when handling CSV files.
   - Provided straightforward methods to detect file encodings, delimiters, and convert large datasets.

2. **[DuckDB](https://duckdb.org/):**  
   - A fast OLAP database engine optimized for analytical queries on local files.
   - Offers superior performance over PostgreSQL for read-heavy analytics in this scenario.

3. **Python Libraries:**
   - **Seaborn / Matplotlib:** Data visualization.
   - **NetworkX:** Network/graph analysis for flight routes.
   - **GeoPandas / geodatasets:** Spatial operations and plotting of flight routes on a world map.
   - **Scikit-learn (LinearRegression):** Simple forecasting model for flight volume growth.

4. **Other Tools:**
   - **Regex / Re:** For filename standardization and text cleanup.
   - **Chardet:** Detecting file encodings where necessary.

---

## Data Source

- **Provider:** Data Glow Up (monthly challenge).
- **Format:** Multiple CSV files representing flight records, including departure/arrival times, airport codes, airline information, etc.
- **Time Span:** Covers flights from the early 2000s to around 2024 (with some partial data for 2025).
- **Airports data:** Contains information about airports location, available in https://github.com/jpatokal/openflights/blob/master/data/airports.dat

---

## Data ingestion & cleaning

1. **CSV Standardization:**
   - **File renaming:** Used regex to enforce a `VRA_YYYY_MM.csv` format.
   - **Encoding & delimiter detection:** 
     - `chardet` to detect file encodings, defaulting to UTF-8 when unknown.
     - `csv.Sniffer` (or custom logic) to determine whether the delimiter was `,`, `;`, or `\t`.
   - **Polars conversion:** Loaded each CSV, converted all columns to string, or forced semicolon delimiters for consistency.

2. **Cleaning steps:**
   - Removed empty strings and replaced them with NULL values where appropriate.
   - Standardized columns like `ano` (year) and `mes` (month) into integer types.
   - Merged or dropped unnecessary columns (`referencia`, etc.) to streamline the schema.

3. **Output:**
   - A collection of neatly formatted CSV files in a consistent, analysis-ready structure.

---

## Database setup & merging

1. **Creating the DuckDB Database:**
   - Connected to `flight_monolith.duckdb` and imported each standardized CSV into its own table using `read_csv_auto`.
   - Applied transformations (`all_varchar=True` or forced conversions) to ensure consistent data types.

2. **Consolidation:**
   - Unioned all flight tables into a single `flights` table for easier querying and analysis.
   - Deduplicated rows by selecting `DISTINCT *` where needed, removing partial or exact duplicates.

3. **Final schema:**
   - Columns like `empresa_aerea`, `numero_voo`, `aeroporto_origem`, `aeroporto_destino`, `ano`, `mes`, etc., stored as strings or integers depending on usage.
   - Additional columns for status, delays, or numeric conversions (e.g., `numero_assentos` -> INT).

---

## Exploratory analyses

### Flight volumes over time

- **Query:** Counted total flights per year, plotted them as line charts or scatter plots to observe overall trends.
- **Findings:** 
  - Identified peaks in certain years, dips around major disruptions (e.g., 2020 pandemic effects).
  - Some years displayed outlier values, leading to further analysis and outlier removal steps.

### Routes and network analysis

- **Route extraction:** 
  - Mapped each origin-destination pair to flight counts and average delays.
  - Used GeoPandas to plot routes on a world map, showing line thickness or color based on flight frequency.

- **Visual insights:**
  - Identified major corridors (high-frequency routes) vs. smaller, less-traveled routes.
  - Showed how certain airports dominate as hubs, with high traffic and more connections.

### Delay analyses

- **Hourly & seasonal delays:** 
  - Explored how delay rates vary by hour of day or by month, factoring in year or grouped year ranges.
  - Noted patterns like early morning vs. late evening spikes, or seasonal fluctuations in delay percentages.

- **Causes & patterns:**
  - Found that delays can compound over the day (afternoon/evening spikes).
  - Certain months show consistently higher or lower delay rates, hinting at weather or holiday influences.

### Seasonality and trends

- **Monthly seasonality:** 
  - Compared flight volumes or delay rates across months (Jan–Dec), using line charts with multiple years superimposed.
  - Observed recurring dips or peaks in certain months, possibly tied to travel demand or operational changes.

---

## Advanced analytics

### Centrality measures

- **NetworkX approach:** 
  - Constructed a directed graph (`DiGraph`) where each airport is a node and each route is a directed edge.
  - **Degree Centrality:** Highlights how many connections (in + out) an airport has.
  - **Betweenness Centrality:** Identifies critical airports on the shortest paths between others.
  - **Closeness Centrality:** Shows how quickly an airport can reach all others.

- **Findings:** 
  - Major airports (e.g., SBGR, SBKP, SBGL) consistently scored highest across multiple measures.
  - Key hubs are crucial for connectivity and operational flow, indicating potential choke points if disruptions occur.

### Linear regression forecast

- **Outlier removal (IQR):** 
  - Filtered out extreme yearly flight counts that skewed the regression.
- **Fitting a model:** 
  - Trained a simple linear model using `sklearn.linear_model.LinearRegression` on the cleaned data.
- **Predictions:** 
  - Forecasted the next 5 years of total flight volumes.
  - Visualized future estimates with a dashed line and separate markers for clarity.

---

## Visualizations

Throughout the project, various plots were created using **Matplotlib** and **Seaborn**:

1. **Line charts:** Yearly flight counts, monthly delay rates, etc.
2. **Bar charts:** Top hub airports, busiest routes, or flight distributions.
3. **Network graphs (NetworkX):** Subgraphs of key airports by centrality, route frequency, or average delays.
4. **Geo Maps (GeoPandas):** Route overlays on a world map, line widths proportional to flight frequency.