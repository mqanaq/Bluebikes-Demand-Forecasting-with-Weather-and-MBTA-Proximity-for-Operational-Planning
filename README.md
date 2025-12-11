# Bluebikes Demand Forecasting with Weather and MBTA Proximity for Operational Planning

## Team Members

- Mohamad Gong  
- Marcus Shi  
- Tianqi Sun  
- Tzu-Jen Chen  
- Arshdeep Oberoi  
- Xiaoqing Ye  

---

## Overview

This repository contains a demand-forecasting project for Boston’s Bluebikes system.  
The goal is to help Bluebikes operators, planners, and city partners anticipate where and when dock and bike shortages are most likely to occur, so they can better allocate rebalancing trucks, expand station capacity, and coordinate with MBTA service.

Using multi-year trip data, daily weather indicators, and MBTA rapid-transit proximity, the project:

- Profiles system- and station-level usage patterns over time, by user type and geography.
- Quantifies how temperature, precipitation, and weather hazards affect daily station demand.
- Trains a **BigQuery ML** model to forecast near-term station demand under different weather scenarios.
- Examines how Bluebikes stations near MBTA rapid-transit stops support first- and last-mile connectivity.

All code for cleaning, feature engineering, analysis, and modeling is contained in a single Jupyter notebook.

---

## Data Sources

**Bluebikes Trip Data**  
Trip-level records of rides in Boston’s public bike-share system, including start/end time, origin and destination station, and user type.  
- Coverage: roughly May 2018 – September 2025  
- Format: monthly CSVs (zip archives)  
- Public source: Bluebikes / Hubway historical trip data (S3 index)

**Bluebikes Station List**  
Official list of active Bluebikes docking stations, with IDs, names, coordinates, seasonal status, and dock counts.  
Used to aggregate trip flows, align station IDs, and distinguish permanent vs. seasonal sites.

**MBTA Rapid-Transit Stations**  
Point locations of MBTA rapid-transit stations (subway and light-rail).  
Used to compute proximity metrics and define first-/last-mile stations near transit.

**Daily Weather Indicators**  
Daily climate series for the Boston region assembled from National Weather Service / NOAA products.  
Used to derive features such as:
- Mean temperature and temperature buckets (below freezing, cold, mild, warm, hot)  
- Precipitation and hazard flags (e.g., heavy rain, snow, storms)  
- Simple “hazard vs. no hazard” indicator

> **Note:** Exact data acquisition and table schemas are documented in the notebook’s data-model and SQL sections.

---

## Methodology

### 1. Data Model and Architecture

The project is organized around a simple analytics data model hosted in **Google BigQuery**:

- **Fact table – daily station demand**
  - One record per (`service_date`, `station_id`)
  - Fields for total trips, member trips, casual trips, and usage near MBTA stations

- **Dimension tables**
  - Station dimension with station attributes (name, location, MBTA proximity, seasonal status, dock count)
  - Weather dimension with daily temperature, precipitation, hazard flags, and derived buckets

The notebook documents the entity-relationship diagram and shows how raw tables roll up into the fact table and model inputs.

### 2. Importing and Cleaning

- Bulk-loads multi-year Bluebikes trip CSVs into BigQuery using Cloud Storage.
- Standardizes column names and user-type fields (old vs. new schema).
- Filters out obviously invalid trips (e.g., test records or 0-second “re-docks”).
- Harmonizes station IDs between legacy and current station lists.
- Joins trips to station metadata and classifies stations by proximity to MBTA rapid-transit stops.
- Aggregates trips to daily station-level demand metrics.

Weather and station tables are similarly cleaned and joined into the daily fact table.

### 3. Feature Engineering

- Builds daily temperature buckets and precipitation/hazard flags.
- Creates lagged and rolling features (e.g., moving averages) to capture temporal structure in demand.
- Encodes calendar features such as:
  - Day of week  
  - Month / season  
  - Holidays and weekends

### 4. Modeling with BigQuery ML

- Trains a **BigQuery ML** regression model to predict next-day (or near-term) station demand.
- Uses station-level and weather-related features to estimate how many trips are expected under different climate conditions.
- Evaluates model performance with standard error metrics and hold-out validation sets.
- Produces forecast outputs that can feed a dashboard or planning workflow.

### 5. Analysis and Visualization

- Explores system growth, seasonal patterns, and differences between member and casual riders.
- Compares demand across stations and neighborhoods, highlighting those:
  - Closest to MBTA rapid-transit stations  
  - With the highest first-/last-mile usage  
- Examines how demand responds to:
  - Temperature buckets  
  - Rain/snow and severe weather days  
- Summarizes where rebalancing and dock-expansion pressure is most acute.

---

## Key Findings (Summary)

Some of the headline insights from the notebook:

- **Strong seasonality and growth** – Daily trips have increased over time, with demand peaking in late spring through early fall while remaining meaningful in winter.
- **Members vs. casual riders** – Members form a stable weekday backbone of demand, especially for commute-like trips; casual riders drive surges on warm, sunny days and weekends.
- **Weather sensitivity** – Mild and warm days support significantly higher daily trips, whereas extreme heat, heavy rain, and snow depress demand and shift usage toward members.
- **Transit adjacency** – A large share of trips start or end at stations close to MBTA rapid-transit stops, underscoring Bluebikes’ role as a first- and last-mile connector.

For full tables, charts, and maps, see the notebook’s “Analysis and Findings” section.

---

## Using This Repository

1. **Clone the repository**

   ```bash
   git clone https://github.com/<your-username>/Bluebikes-Demand-Forecasting-Weather-MBTA.git
   cd Bluebikes-Demand-Forecasting-Weather-MBTA
