1. Problem instances for the go train schedule optimization problem, the solvers, and the solutions.

1.1 Problem Instance Class Explained:

| **Parameter**             | **Meaning / Definition**                        | **Effect on Ridership Curve**                         | **Typical Values**                             | **Why It’s Useful**                                           |
| ------------------------- | ----------------------------------------------- | ----------------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------- |
| **`EXP_RIDERSHIP`**       | Baseline total riders per GO line during 4–6 PM | Sets overall passenger totals for each line           | e.g. Lakeshore W = 33 239                      | Defines demand scale for each corridor                        |
| **`INTERVAL_SIZE`**       | Length of each time bucket (minutes)            | Smaller intervals → higher time resolution            | 5 min (→ 24 intervals from 4–6 PM)             | Controls time granularity of demand data                      |
| **`global_scale`**        | Multiplier for total ridership across all lines | Raises or lowers *all* demand volumes                 | 0.5 – 1.3                                      | Simulates high- or low-demand days (e.g., Sunday vs CNE)      |
| **`line_scales`**         | Per-line adjustment factors                     | Boosts or reduces demand on specific routes           | e.g. `{LW: 1.6, LE: 1.4}`                      | Models events affecting certain lines (CNE → Lakeshore spike) |
| **`spread`**              | Random noise level (% deviation around mean)    | Higher → more jagged, realistic fluctuations          | 0.08 – 0.15                                    | Adds stochastic variation to mimic real-world randomness      |
| **`shape`**               | Type of curve used for demand distribution      | `"gaussian"` = symmetric bell; `"beta"` = skewed      | mostly `"gaussian"` now, will explore beta if time allows| Chooses between symmetric or skewed arrival patterns|
| **`peak_time_min`**       | Minutes after 4 PM when demand peaks (μ)        | Moves the bell curve horizontally in time             | 60 → 5:00 PM  •  75 → 5:15 PM  •  95 → 5:35 PM | Represents *when* the rush-hour maximum occurs                |
| **`peak_width_min`**      | Approximate “width” of the peak (σ) in minutes  | Smaller → tall sharp spike; larger → flat broad curve | 20 – 50                                        | Controls how concentrated or spread-out the rush hour is      |
| **`beta_a`, `beta_b`**    | Shape parameters for Beta distribution          | Adjust skewness (early vs late rise/decline)          | e.g. 3 and 5 for right-skew                    | Used only if `shape="beta"` — for asymmetric patterns|
| **`seed`**                | Random-number seed for reproducibility          | Same seed → same scenario each run                    | integer (e.g. 101)                             | Keeps experiments repeatable                                  |
| **`get_instances()`**     | Generates all 10 profiles (Mon–Sun, CNE, etc.)  | Returns dict {name → scenario}                        | —                                              | Quickly build multiple demand cases                           |
| **`generate_scenario()`** | Creates one scenario given parameters           | Produces `{line → {time → ridership}}`                | —                                              | Core function producing data for your optimizer               |

1.2 Problem Instance Justifications:

| Category             | Timing         | Demand Shape            | Main Idea                                  |
| -------------------- | -------------- | ----------------------- | ------------------------------------------ |
| Early-week (Mon–Wed) | 5:10 – 5:15 PM | Sharp, high peaks       | Normal workday commuter surge              |
| Thursday             | 5:20 PM        | Broader, slightly later | After-work socializing delays departures   |
| Friday               | 5:00 PM        | Flat, earlier           | People leave early, flexible schedules     |
| Weekend              | 5:25 – 5:30 PM | Wide, low               | Leisure travel dominates                   |
| Christmas            | 5:00 PM        | Very broad              | Family trips, dispersed timing             |
| CNE                  | 5:35 PM        | Sharp + variable        | Event exodus centered on Lakeshore lines   |
| Valentine’s Day      | 4:10 PM        | Moderate + noisy        | Early evening outings shift travel earlier |
