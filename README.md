# AI Programming Foundations — Project 1: Data Workflow

## Description
A Pandas/Matplotlib data workflow notebook analysing UK road traffic collision
severity — how it relates to time (year, hour, day of week, month) and speed
environment (posted speed limit). Covers ingestion, cleaning, EDA, and
visualization; no model training (out of scope for this project).

## What was built
- `data_workflow.ipynb` — the full workflow: full-width ingestion (all 44
  collision-table columns, so dtypes/summary stats reflect the whole
  dataset) narrowed only afterwards to the columns this analysis and its
  visualizations actually need, two documented cleaning functions, a
  parameterised EDA function, and five interpreted visualizations, in
  clearly headed sections (Setup, Data Ingestion, Data Cleaning, EDA,
  Visualizations, Summary & Interpretation).

## Dataset
**STATS19 Road Safety Collision Data** — the UK Department for Transport's
official road traffic collision database, "last 5 years" pre-curated extract
(collision years 2021-2025).
- Official source: [gov.uk Road Safety Data](https://www.gov.uk/government/statistics/road-safety-data)
- [data.gov.uk series](https://www.data.gov.uk/dataset/cb7ae6f0-4be6-4935-9277-47e5ce24a11f/road-accidents-safety-data)

Raw data files are not committed to this repository (the collision CSV is
~94MB). To reproduce, place the following at the paths the notebook expects:
- `data/raw/dft-road-casualty-statistics-collision-last-5-years.csv`
- `data/dimensions/dft-road-casualty-statistics-road-safety-open-dataset-data-guide-2025.xlsx`
  (already included in this repo)

## How to Run
```bash
git clone git@github.com:LittleRabbitFooFoo/ai-programming-foundations-project.git
cd ai-programming-foundations-project
python3 -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook data_workflow.ipynb
```
Run all cells top to bottom. No network access or download step is required —
the notebook reads the local data files listed above.

## Reflections

- **Bias awareness:** two cleaning-stage risks stand out. First, if the `-1`
  missing/unknown sentinel had been dropped (row-wise) instead of converted
  to `NaN`, an entire stratum could be silently discarded — biasing results
  toward whatever remained. Second, imputing a "typical" value in place of a
  real unknown would mask genuine missingness as if it were ordinary data,
  producing misleadingly confident conclusions. Both risks are why this
  notebook nulls sentinels explicitly rather than dropping or imputing them.
- **ML workflow changes:** the notebook's full-width ingestion step
  programmatically checks every field's entry in the DfT data guide and
  finds 23 of the collision table's 44 columns are genuine coded categorical
  fields (e.g. `weather_conditions`, `road_surface_conditions`,
  `junction_detail`, `light_conditions`, `road_type`, `urban_or_rural_area`,
  plus higher-cardinality ones like `police_force` and
  `local_authority_district`) — these would need one-hot (or similar)
  encoding. Continuous/count/identifier fields (`number_of_vehicles`,
  `number_of_casualties`, `speed_limit`, `first_road_number`,
  `second_road_number`, the lat/long/easting/northing location fields) would
  need scaling instead. Doing the check against the full schema, rather than
  guessing from memory, also turned up a real gotcha: two fields the guide
  names (`enhanced_collision_severity`,
  `did_police_officer_attend_scene_of_collision`) don't match the actual CSV
  column names (`enhanced_severity_collision`,
  `did_police_officer_attend_scene_of_accident`) — a reminder to verify
  field names empirically before trusting documentation.
- **Neural network prep:** the same encoding/scaling requirements apply, plus
  attention to class imbalance — Fatal collisions are a small fraction (~1.5%)
  of the data, which would need addressing (e.g. class weighting or
  resampling) before training a classifier on severity.
- **Agentic automation potential:** the DfT data guide (dimension/codebook
  tables) is small enough — a few thousand tokens for the collision table —
  to be injected directly into an agent's context as machine-readable
  field/value documentation, rather than requiring a retrieval system, making
  it a plausible candidate for agent-assisted querying of this dataset.
