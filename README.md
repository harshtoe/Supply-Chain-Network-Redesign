# Supply Chain Network Redesign & Cost-to-Serve Optimization

**Individual Project | Supply Chain Analytics · Network Optimization · Operations Research · Business Intelligence**

**Tools:** `MySQL 8.0` · `anyLogistix PLE` · `Python` · `PuLP / CBC` · `Excel` · `Power BI` · `DAX` · `ArcGIS Maps`

> **39.0% lower modeled transportation cost** using a four-distribution-center (DC) network versus calibrated seller-direct fulfillment: **$1.80M → $1.10M**. This is a **gross, model-based** reduction, *not* confirmed all-in savings. An indicative DC lease-cost sensitivity reduces the estimated benefit to **~10%** before additional operating costs.

**Project at a glance:** I independently developed an end-to-end analytical decision-support workflow on the public Olist Brazilian e-commerce dataset (~99,441 historical orders, 2016–2018). I engineered order- and lane-level data in SQL, diagnosed freight-cost drivers, calibrated and debugged a network model, explored facility counts, optimized a four-DC configuration, independently checked the result with Python MILP, and presented the findings in Power BI.

<!-- IMAGE SLOT 1 — RECRUITER HERO: Full Power BI dashboard screenshot. -->
<!-- Add image file at images/dashboard-overview.png, then uncomment: -->
<!-- ![Power BI executive dashboard](images/dashboard-overview.png) -->

## Business Question

**Could a regional DC network lower transportation cost-to-serve while maintaining customer coverage and accounting for delivery-service risk?**

The analyzed baseline uses **seller-direct shipping** (seller → carrier pickup → line-haul → last mile → customer), with no shared DC consolidation. The proposed network routes demand through selected regional DCs. This is a *network design study*, not a live deployment or a completed operational pilot.

## Impact & Key Results

| KPI | Result |
| --- | ---: |
| Calibrated seller-direct baseline cost | **$1,804,206.60** |
| Optimized **N=4** network cost (anyLogistix) | **$1,100,185.02** |
| **Gross modeled transportation cost reduction** | **$704,021.58 / 39.0%** |
| Blended cost per chargeable kg | **~$6.80 → ~$4.15** |
| Demand served by optimized model | **100% of 92 customer zones** |
| Independent Python MILP cross-check | **$918,737.67** (16.5% below anyLogistix) |
| Indicative savings after DC lease-only sensitivity | **~$187,000 / ~10%**, before other costs |
| Historical late deliveries | **6.66%** |
| Mean review-score gap: on-time vs. late | **2.02 points**, *correlation only* |

**Interpretation:** The 39% figure is the calibrated **modeled transportation** comparison. The lease-only scenario assumes **four 2,000 m² facilities** and estimates roughly **$517,000/year** in rent; it is an order-of-magnitude check rather than a validated total-cost-of-ownership model. Inbound seller-to-DC freight, labor, and inventory costs remain unmodeled.

## What I Built

| Workstream | My contribution | Deliverable |
| --- | --- | --- |
| **Data engineering (SQL)** | Integrated Olist orders, items, customers, sellers, product and geographic data; corrected order-item join fan-out; engineered delivery/weight features | Order facts and lane-level benchmark |
| **Cost analytics** | Compared freight against chargeable volumetric weight and distance; calibrated model coefficients against historical freight | Evidence-based transportation-cost assumptions |
| **Network design** | Aggregated sellers into three supply nodes and customers into 92 demand zones; ran Greenfield Analysis for N=1–5 | Facility-count sensitivity and hub candidates |
| **Optimization** | Built and debugged N=4 capacitated network optimization with a soft delivery penalty | Optimized facility-to-customer assignment |
| **Independent validation** | Rebuilt assignment problem in Python using PuLP/CBC and haversine distance | Cross-check against vendor-tool output |
| **Business intelligence** | Designed Power BI web dashboard with ArcGIS maps, DAX KPIs, GFA sensitivity and delay-review comparison | Executive decision-support dashboard |

## End-to-End Workflow

```mermaid
flowchart TD
    A[Olist CSV source tables] --> B[MySQL ingestion and order-level aggregation]
    B --> C[Chargeable weight, delivery timing and geographic zoning]
    C --> D[2,048 historical origin-destination lane benchmarks]
    D --> E[Freight driver analysis and cost calibration]
    E --> F[anyLogistix model checks and corrections]
    F --> G[Greenfield Analysis: N = 1 to 5]
    G --> H[Select four candidate DC locations]
    H --> I[Capacitated N=4 optimization with soft SLA penalty]
    I --> J[Independent Python PuLP/CBC MILP validation]
    J --> K[Financial sensitivity, assumptions and SLA analysis]
    K --> L[Power BI decision-support dashboard]
```

**Separate diagram files:** [`process_flowchart.svg`](process_flowchart.svg) · [`process_flowchart.mmd`](process_flowchart.mmd)

<!-- IMAGE SLOT 2 — Exported flowchart if preferred over Mermaid. -->
<!-- ![End-to-end process diagram](process_flowchart.svg) -->

## Methodology

### 1. Transform historical order data into a usable network model

- Source: **Olist Brazilian E-Commerce Dataset**, approximately **99,441** orders from **2016–2018**.
- Transformation chain: `orders + order_items → order_agg → chargeable_weight → order_fact → geo-clustering → zone_pair_ground_truth`.
- Aggregated at **order level**, preventing freight costs from being duplicated across item-level joins.
- Constructed **2,048 historical origin–destination lanes** for cost-model benchmarking.
- Represented seller origins with **three supply nodes** (São Paulo, Minas Gerais, long-tail) and demand with **92 zones** using two-digit CEP prefixes and state fallbacks. Finer city-level nodes exceeded the anyLogistix PLE 100-node limit.

### 2. Calibrate cost drivers using data, not guesswork

Observed Pearson correlation with historical freight:

| Freight predictor | Correlation |
| --- | ---: |
| **Chargeable weight** | **r = 0.657** |
| Straight-line distance | **r = 0.316** |

Chargeable weight (kg):

```text
max(actual_weight_kg, length_cm × width_cm × height_cm / 6000)
```

The calibrated model used a **0.00517403 m³/kg** mass-to-volume factor and an approximately **$2.4412/m³-km** path-cost coefficient. The corrected baseline remained **17.9% different from raw historical freight**, reflecting the difference between the chosen volumetric cost model and historical carrier pricing.

### 3. Diagnose and fix solver configuration issues

Rather than trust the first optimization outputs, I benchmarked them against historical freight and identified three issues:

1. **Vehicle capacity = 0 m³** → no orders could be fulfilled.
2. **Incorrect unit conversion of 1 kg = 1 m³** → materially inflated cost; corrected to **0.00517403 m³/kg**.
3. **Arbitrary path cost of $0.05/m³-km** → understated cost; recalibrated to **~$2.4412/m³-km**.

An Excel side-model helped distinguish volumetric costs from an embedded distance-based term in anyLogistix.

### 4. Explore facility counts and optimize N=4

- Ran **Greenfield Analysis (GFA) for N=1–5**, based on demand-weighted straight-line distance.
- Selected **N=4** considering diminishing marginal GFA improvement and practical hub availability; **this was not proven to be the total-cost economic optimum**.
- Mapped candidate centroids to reported hub areas: **Petrópolis (RJ), Santa Catarina, São Paulo, and Petrolina (PE)**. The report describes Santa Catarina at state level and does not identify a specific city.
- Solved the **N=4 capacitated network** in anyLogistix with DC monthly throughput limits, a **six-day service target** and a **$500 per day late** soft penalty. DC Initial Stock was used as a deliberate supply approximation.

<!-- IMAGE SLOT 3 — DC candidate locations / optimized network map. -->
<!-- ![Proposed distribution-center locations](images/dc-network.png) -->

<!-- IMAGE SLOT 4 — GFA N=1–5 facility count sensitivity. -->
<!-- ![Facility-count sensitivity](images/facility-sensitivity.png) -->

### 5. Validate independently with Python optimization

Reformulated the **four-DC assignment** as a capacitated **mixed-integer linear program** using **PuLP and CBC**, with the same calibrated cost parameters and **haversine** distances. The Python objective returned **$918,737.67**, compared with **$1,100,185.02** in anyLogistix.

The **16.5% difference is disclosed**, not treated as an exact match: the Python approximation does not reproduce the road-network distance treatment of the main model. This serves as a **sanity check**, not proof of identical formulations.

### 6. Translate the analysis into executive reporting

Created a Power BI **web** dashboard with ArcGIS Maps, DAX metrics, zone-level cost visualization, DC location mapping, facility-count sensitivity and order-delay versus review-score analysis. **6.66%** of historical orders were delivered late, with a **2.02-point** average review-score difference versus on-time deliveries. The relationship is **observational, not causal**.

<!-- IMAGE SLOT 5 — Cost-to-serve map. -->
<!-- ![Customer zone cost-to-serve map](images/cost-to-serve-map.png) -->

<!-- IMAGE SLOT 6 — SLA vs. review-score visual. -->
<!-- ![SLA breach and review score analysis](images/sla-review.png) -->

## What the Results Do *Not* Prove

- **Net savings are not 39%.** The gross transportation figure excludes multiple DC operating expenses; a lease-only scenario suggests ~10% before other costs.
- **Inbound seller-to-DC transportation is not yet priced.** This may further reduce realized savings.
- **N=1, 2, 3 and 5 were not fully re-optimized MILP cases.** Their sensitivity curve comes from distance-only GFA.
- **Review-score differences are not causal effects** of delayed delivery.
- **No live integration, DC rollout, inventory optimization, or disruption-resilience stress test** was completed.
- Migrating sellers to a DC-led model would also require operational and contractual changes beyond the analysis.

## Repository Structure & Source Files

The analysis source files **exist and can be added to this repository**. The folders below are suggested destinations, **not claims that the scripts/models have already been uploaded**. Replace generic names with your actual filenames and remove folders that do not apply.

```text
.
├── README.md
├── process_flowchart.mmd
├── process_flowchart.svg
├── sql/                  # MySQL data preparation and benchmark queries
├── python/               # PuLP/CBC optimization cross-check
├── anylogistix/          # Exported GFA/network model files and configuration
├── powerbi/              # Power BI export(s), DAX measures or screenshots
├── excel/                # Cost calibration side-model, if shareable
├── images/               # Dashboard screenshots and figures (add yourself)
└── docs/                 # Report / assumptions / methodology notes, if desired
```

**To reproduce:** Place the actual source files in the relevant folders and document (1) Olist CSV download/setup, (2) SQL execution order, (3) calibration queries, (4) anyLogistix model configuration and version, (5) Python requirements and run command, and (6) Power BI import steps. **Exact execution commands cannot be supplied reliably until the source files are reviewed.** Do not commit private credentials, licensed binaries you cannot redistribute, or dataset copies without checking usage terms.

## Future Work

- Expand from a transportation-only objective to **total cost of ownership (TCO)** including inbound freight, facilities, labor, inventory, and handling.
- Test disruptions, alternative demand forecasts and DC capacities.
- Investigate the highest-volume proposed **São Paulo DC** first (modeled at **134,733 kg/year**), after validating commercial feasibility.
- Incorporate live order feeds **only if** progressing from a decision-support study to a deployed operations workflow.

## Dataset & Project Context

**Dataset:** Public **Olist Brazilian E-Commerce Dataset** (historical 2016–2018 order data). **Academic context:** Production Engineering, VJTI Mumbai. **Project type:** Individual analytical project.

<!-- Add your verified original Olist dataset URL, your GitHub profile/contact, and the project's actual completion date here. -->
<!-- Add a license only if you have chosen one and have rights to distribute all included content. -->
