# WeatherNext 2 data access report

**Evidence date:** 7 August 2026 (Australia/Brisbane)  
**Scope:** Windward Milestone 1 only. No pipeline or application code was started.  
**Decision:** **HOLD before Milestone 2.** WeatherNext 2 data is available, but the current public data products do not expose the cyclone-native outputs required by DeepMind's reference tracker, and the legal review in [`LICENSING.md`](LICENSING.md) identifies launch-blocking terms for this life-safety use case.

If those blockers are cleared, **BigQuery/Analytics Hub is the best candidate for one centrally scheduled ingest per forecast cycle**. Earth Engine is useful for geospatial exploration, GCS Zarr for bulk historical work, and Vertex AI for a separately budgeted experiment. None should be queried directly by 10,000 clients.

## What the investigation changed

| Brief assumption | Verified position on 7 August 2026 | Consequence for Windward |
|---|---|---|
| Hundreds to 1,000 ensemble members | The public experimental dataset produced by an operational model version releases **64 members**. Vertex defaults to 64: four model seeds with 16 samples each. No official source establishes a 1,000-member option. | Raw probability increments are 1/64 = **1.5625 percentage points**. Product copy and calibration plans must not imply finer native ensemble granularity. |
| Hourly public forecasts | BigQuery, Earth Engine, and the documented Zarr dataset are **6-hourly** from +6 to +360 hours. Vertex offers a selected hourly variable subset. | An hourly arrival window cannot be computed honestly from the public feed without an explicitly validated interpolation method; Vertex is not yet a production answer. |
| Archive roughly from 2020 | The public precomputed archive is **2022-present**. Vertex can generate historical forecasts only from **January 2024-present**. | The calibration sample is shorter than assumed and may be thin for Australian cyclone events. |
| Forecast-cycle data contains cyclone tracks | No reviewed BigQuery schema, Earth Engine catalog, Zarr schema, Vertex schema, or developer guide advertises cyclone IDs, positions, tracks, wind radii, or cyclone-existence fields. | DeepMind's tracker cannot be applied directly to the advertised public fields. This is a hard data-contract blocker, not an implementation detail. |
| Four near-real-time cycles daily | Confirmed at 00, 06, 12, and 18 UTC, but BigQuery/Earth Engine release is scheduled about **7 hours 30 minutes after initialization**. | Windward's source freshness is materially later than the nominal model initialization time. |
| New managed-data release in August 2026 | Google's release notes place WN2 v1 on BigQuery/Earth Engine and a Vertex soft launch on **3 October 2025**. July 2026 deprecated older Graph/Gen feeds; 6 August 2026 added WN2 to the open-source repository. | Treat managed-feed and repository releases as separate, independently versioned dependencies. |

The published forecast specifications and dissemination schedule are documented in the [WeatherNext model guide](https://developers.google.com/weathernext/guides/models), [Earth Engine catalog](https://developers.google.com/earth-engine/datasets/catalog/projects_gcp-public-data-weathernext_assets_weathernext_2_0_0), and [forecast-access guide](https://developers.google.com/weathernext/guides/access-forecast). Google labels the feed experimental, says the operational model version may not match research-reported accuracy, and requires a notice that it is not validated or approved for real-world use. Those qualifications are material for Windward.

## Evidence labels and workload definition

- **Confirmed** means visible in an official Google page, catalog, downloadable official notebook, or the pinned upstream repository.
- **Published schema** means Google's starter notebook documents the shape, but an authenticated live table inspection has not yet been possible.
- **Estimate** means an explicit calculation from documented dimensions and list prices; it is not an invoice prediction.
- **Unknown** means the public material does not answer the question. It is kept as a blocker rather than filled with an assumed value.

The cost workload used throughout is:

- 10,000 users × 4 refreshes/day × 30 days = **1,200,000 application refreshes/month**.
- Four WeatherNext forecast cycles/day × 30 days = **120 source ingests/month**.
- Recommended topology: query the WeatherNext source once per cycle, precompute location products centrally, and serve app refreshes from a database/object cache/CDN. Application-store, CDN, observability, and end-user egress costs are outside this source-access estimate.
- All prices below are USD list-price snapshots and exclude tax, negotiated discounts, failures/retries, and support.

The brief's requirement to retain every run is not free and is not included in query/compute figures below. The official Zarr notebook displays roughly 1 TB logical size for a full run, implying roughly **120 TB of new logical data/month** if every full field is copied. Retaining only the two global 10 m wind components adds 29.704 GiB/run, or **3.481 TiB/month** (41.772 TiB/year) before compression and metadata. The minimum replay/calibration field set, compressed bytes, storage class, region, lifecycle, and whether an independently durable copy is contractually required all remain unresolved; a storage quote before those decisions would be false precision.

## Access-path decision matrix

| Path | Actual availability | Data shape | Cadence and latency | Historical depth | Quota/cost finding | Windward verdict |
|---|---|---|---|---|---|---|
| `developers.google.com/weathernext` | Public documentation, not a data API | Guides, catalogs, starter notebooks, terms, and access forms | Documentation only | N/A | No runtime quota or data charge | Authority/index only |
| BigQuery via Analytics Hub | Active, but requires a [WeatherNext Data Request](https://developers.google.com/weathernext/guides/access-forecast) and linked-dataset subscription | Nested geospatial tables for full 64-member output and ensemble mean | Four runs/day; all members scheduled about 7h30 after init | 2022-present | On-demand query bytes billed to subscriber; worked estimate below | **Preferred candidate**, conditional on access, schema inspection, dry runs, and legal clearance |
| Earth Engine | Active gated full and mean ImageCollections | One image per init/lead/member; 0.25° raster bands | Same four runs/day and publication schedule as BigQuery | 2022-present | EECU-based and workload-dependent; paid commercial backend required | Exploration, validation, and maps; not the primary tabular production path |
| Google Cloud Storage | Gated scheduled real-time and historical Zarr feed is documented; object access was not authenticated in this review | Chunked multidimensional arrays, full ensemble and mean | Four runs/day; scheduled about 6h50 after init, with no SLA | 2022-present plus current scheduled runs | Read-operation and transfer costs; chunking is poor for isolated point reads | Bulk candidate after object audit; not the point-serving layer |
| Vertex AI Model Garden | WeatherNext 2 only; restricted Early Access Program and project allowlist | Custom Training jobs write configured Zarr forecasts to the customer's bucket | Init every 6h; job queue/start/runtime latency unpublished | Can generate January 2024-present | GPU custom-training cost and quota dominate; no defensible total without benchmark | Separate experiment only; **not production-cleared** |
| Pinned GitHub repository | Public code and weights bucket; no operational feed client | WN2 model, configs, checkpoint loader, and cyclone tracker | User-operated inference only | Depends on separately sourced initial/training data | Self-host cost not estimated because v1 excludes it | Reference implementation only; see [`REPOSITORY_REPORT.md`](REPOSITORY_REPORT.md) |

The access form is reviewed weekly and Google says approval typically takes 5–7 business days. Approval was not available during this investigation, so live authenticated table metadata and billing dry runs remain unresolved.

The developer portal is documentation and discovery, not a callable forecast API. Its [release notes](https://developers.google.com/weathernext/release-notes) list WeatherNext 2 v1.0 on BigQuery and Earth Engine from 3 October 2025, while the July 2026 access documentation deprecates the older WeatherNext Graph and Gen feeds. No separate August 2026 managed-dataset release is documented; the 6 August change is the open-source repository update covered in `REPOSITORY_REPORT.md`.

## The common public experimental forecast contract is 64-member, 0.25°, and 6-hourly

The full experimental collection, produced by an operational model version, is global on a 0.25° latitude/longitude grid (721 × 1,440 = 1,038,240 cells), initialized at 00, 06, 12, and 18 UTC. Each run contains 64 ensemble members and 60 lead steps, +6 through +360 hours. The nominal pixel size in Earth Engine is 27,830 m.

All 64 members are released together to BigQuery and Earth Engine on this approximate UTC schedule:

| Initialization | Scheduled publication | Nominal latency |
|---:|---:|---:|
| 00:00 | 07:30 | 7h30 |
| 06:00 | 13:30 | 7h30 |
| 12:00 | 19:30 | 7h30 |
| 18:00 | 01:30 next day | 7h30 |

Google states typical variance of ±15 minutes and occasional variance of ±60 minutes or more. This is a publication schedule, not an SLA. The catalog was current through the 18 UTC run on 6 August when checked on 7 August.

The common gridded fields are:

- Surface: six-hour accumulated precipitation; 100 m u/v wind; 10 m u/v wind; 2 m temperature; mean sea-level pressure; sea-surface temperature.
- Upper air at 50, 100, 150, 200, 250, 300, 400, 500, 600, 700, 850, 925, and 1,000 hPa: geopotential, specific humidity, temperature, u wind, v wind, and vertical velocity.

That is eight surface fields plus six families × 13 levels = **86 advertised full-ensemble bands** in Earth Engine. The feed does **not** advertise gusts, a sustained-duration wind diagnostic, cyclone IDs, track points, cyclone-existence probability grids, intensity scalars, radius of maximum wind, or 34/50/64-knot quadrant radii.

## BigQuery/Analytics Hub is the strongest ingestion candidate, but its live schema is not yet verified

### Availability and table names

The [BigQuery guide](https://developers.google.com/weathernext/guides/bigquery) confirms two US-location Analytics Hub listings: [WeatherNext 2 full ensemble](https://console.cloud.google.com/bigquery/analytics-hub/exchanges/projects/871883017250/locations/us/dataExchanges/weathernext_19397e1bcb7/listings/weathernext_2_19a39fe59dd) and [WeatherNext 2 mean](https://console.cloud.google.com/bigquery/analytics-hub/discovery/projects/gcp-public-data-weathernext/locations/us/dataExchanges/weathernext_19397e1bcb7/listings/weathernext2_mean_19dbf307dec). After approval and subscription, the subscriber chooses the project and linked-dataset name; the tables are:

- `[project].[linked_dataset].weathernext_2_0_0`
- `[project].[linked_dataset].weathernext_2_0_0_mean`

The subscriber/querying project pays query charges. The listings themselves do not establish a production SLA.

### Published schema, with an unresolved inconsistency

Google's [official BigQuery starter notebook](https://storage.googleapis.com/weathernext-public/colabs/WeatherNext_2_Starter_Guide_BigQuery.ipynb) publishes this nested shape for the full-ensemble table:

| Level | Field | Type/mode | Meaning |
|---|---|---|---|
| Root | `init_time` | `TIMESTAMP`, daily partition | Forecast initialization |
| Root | `geography` | `GEOGRAPHY` | Grid-cell point |
| Root | `geography_polygon` | `GEOGRAPHY` | Grid-cell polygon for spatial predicates |
| Root | `forecast` | repeated `RECORD` | Lead-time records |
| `forecast` | `time` | `TIMESTAMP` | Valid time |
| `forecast` | `hours` | `INTEGER` | Forecast lead hours |
| `forecast` | `ensemble` | repeated `RECORD` | Member values |
| `ensemble` | `ensemble_member` | `STRING` | Member identifier |
| `ensemble` | `10m_u_component_of_wind` | `FLOAT` | 10 m eastward wind, m/s |
| `ensemble` | `10m_v_component_of_wind` | `FLOAT` | 10 m northward wind, m/s |
| `ensemble` | `100m_u_component_of_wind` | `FLOAT` | 100 m eastward wind, m/s |
| `ensemble` | `100m_v_component_of_wind` | `FLOAT` | 100 m northward wind, m/s |
| `ensemble` | `2m_temperature` | `FLOAT` | Temperature, K |
| `ensemble` | `mean_sea_level_pressure` | `FLOAT` | Pressure, Pa |
| `ensemble` | `total_precipitation_6hr` | `FLOAT` | Six-hour accumulation, m |

The mean table removes the member array and adds derived `10m_wind_speed` and `100m_wind_speed` fields. However, the same official notebook later queries `sea_surface_temperature`, which its schema table omits, and discusses upper-air data without publishing its exact nested BigQuery field contract. It also contains inconsistent mode labels around repeated records.

Therefore this is a **published schema, not a frozen ingestion contract**. Before any pipeline code, an approved project must capture:

1. `bq show --schema --format=prettyjson` for both tables.
2. `INFORMATION_SCHEMA.COLUMN_FIELD_PATHS` and partition metadata.
3. Representative dry runs for global and Australia-only queries.
4. A schema checksum and dated fixture.

The notebook's demonstrated query pattern unnests both `forecast` and `ensemble` and uses `ST_INTERSECTS` against `geography_polygon`. It reports 1,038,240 rows for one global lead and 99,840 rows for a 26-cell New York example over 60 leads × 64 members. Only initialization-time partitioning is documented; no clustering or spatial block-pruning guarantee was found.

### Quotas and pricing

Relevant [BigQuery quotas](https://docs.cloud.google.com/bigquery/quotas) and [on-demand pricing](https://cloud.google.com/bigquery/pricing) at the evidence date include:

- $6.25 per TiB processed after the first 1 TiB/month free tier.
- A 10 MB minimum billed per query and 10 MB minimum per referenced table.
- 200 TiB/day on-demand query usage is shown in current quota material as an adjustable project default; project custom quotas can be lower.
- Up to 1,000 queued interactive queries and 20,000 queued batch queries/project.
- Query attempts can run for up to six hours, with retry behavior subject to the documented limit.
- API quotas include 100 requests/second/user/method and 300 concurrent API requests/user; result/streaming endpoints have separate limits.

These are service ceilings, not a reason to make one query per app refresh.

### Worked BigQuery projection for 10,000 users

For a transparent **selected-value payload floor**, one global numeric field per run is:

`1,038,240 cells × 60 leads × 64 members × 8 bytes/FLOAT = 29.704 GiB = 0.029008 TiB`

The estimate excludes `GEOGRAPHY`, timestamps, member strings, nested-record overhead, output storage, and transfer. A spatial predicate may require scanning the polygon field. More importantly, the notebook documents a **day** partition on `init_time`, not one partition per run. A partition filter alone cannot isolate one of four same-day initializations. An exact `init_time` predicate may benefit from storage-block pruning, but no such guarantee or clustering is documented. Only a BigQuery dry run can resolve billed bytes.

| Selected fields/run | Unique-run payload/month at 120 runs | Payload-floor cost after one shared free TiB | Interpretation |
|---:|---:|---:|---|
| 1 | 3.481 TiB | $15.51 | One global `FLOAT` field |
| 2 | 6.962 TiB | **$37.26** | 10 m u/v minimum for wind magnitude |
| 4 | 13.924 TiB | $80.77 | Example u/v + pressure + precipitation |
| 6 | 20.886 TiB | $124.29 | Wider surface extraction |

For the two-wind-field case, the unique-run payload floor is about $0.36 per source cycle before free tier, $0.31 per cycle averaged after it, or **$0.000031 amortized per application refresh**. It is not the expected bill.

With four scheduled queries against a daily partition, and no effective pruning beyond the partition, the day's queries can scan 1 + 2 + 3 + 4 = **10 run-equivalents** while only four unique runs arrive. That is 2.5× the payload floor: for two wind fields, about **17.405 TiB/month and $102.53/month** after the free TiB, or $0.000085 amortized per app refresh. A once-daily query would avoid repeated same-day scans but add unacceptable source latency; a staged one-query-per-partition strategy cannot retrieve runs that have not arrived yet.

Direct per-user querying is rejected:

- Absolute billing-floor model: 1.2 million queries × 10 MB = 10.914 TiB/month, about **$61.96/month** after the free tier—even if every point query somehow hit only the minimum.
- One-run-scan risk bound: if each request scanned the equivalent of two global wind fields for one initialization, 0.058016 TiB × 1.2 million = about 69,619 TiB/month, or roughly **$435,115/month**.
- Daily-partition no-pruning bound: if a request made late in the UTC day scanned all four initializations, 0.232065 TiB × 1.2 million = about 278,478 TiB/month, or roughly **$1.74 million/month**.

The last two figures are deliberately conservative bounds, not observed billing.

The scheduled-ingest estimates make BigQuery economically plausible. The enormous range between payload floor, partition sensitivity, and direct-query bounds is why authenticated dry-run bytes are a Milestone 2 gate.

## Earth Engine exposes the clearest public schema, but not a predictable per-query cost

### Availability and schema

The gated assets are:

- Full ensemble: `projects/gcp-public-data-weathernext/assets/weathernext_2_0_0`
- Ensemble mean: `projects/gcp-public-data-weathernext/assets/weathernext_2_0_0_mean`

The [full collection catalog](https://developers.google.com/earth-engine/datasets/catalog/projects_gcp-public-data-weathernext_assets_weathernext_2_0_0) documents one image per initialization, `forecast_hour`, and `ensemble_member`: 60 × 64 = 3,840 images/run. Image properties are `start_time` (string), `end_time` (string), `forecast_hour` (integer), `ingestion_time` (double), and `ensemble_member` (string). The full band list is the 86-field contract summarized above. The [mean collection](https://developers.google.com/earth-engine/datasets/catalog/projects_gcp-public-data-weathernext_assets_weathernext_2_0_0_mean) additionally provides derived 10 m and 100 m wind speeds.

Google's [Earth Engine starter notebook](https://storage.googleapis.com/weathernext-public/colabs/WeatherNext_2_Starter_Guide_Earth_Engine.ipynb) deliberately demonstrates 10 rather than all 64 members because the full ensemble is too slow for its Colab example, and recommends BigQuery for tabular workflows. That supports using Earth Engine for validation and visualization, not 1.2 million point requests.

### Commercial topology, quotas, and pricing

Commercial operational use requires a paid Earth Engine configuration. Under the [Google Cloud service-specific terms](https://cloud.google.com/terms/service-terms), Windward cannot let end users access or interact directly with Earth Engine APIs unless each user has their own eligible Google Cloud account. Windward would need a controlled backend.

Current [Earth Engine pricing](https://cloud.google.com/earth-engine/pricing) and [usage limits](https://developers.google.com/earth-engine/guides/usage) include:

- Limited: usage charges, no platform fee, credits, or SLA.
- Basic: $500/month, including 100 batch EECU-hours, 33 online EECU-hours, and 100 GB storage.
- Professional: $2,000/month, including 500 batch EECU-hours, 166 online EECU-hours, 1 TB storage, and an SLA.
- Compute: $0.40/EECU-hour for the first 10,000 monthly hours, $0.28 for the next tier, then $0.16.
- Storage: $0.000035616/GiB-hour, about $0.026/GiB-month.
- General defaults include 40 concurrent standard and 40 high-volume requests, 100 requests/second per project and account, average two concurrent batch tasks, 3,000 queued tasks, 100 MiB computed-result size, and a 10 MB request payload. Plan entitlements can differ—for example, Limited/Basic list 20 high-volume concurrent requests, while Professional lists up to 500.

Earth Engine states that EECU use for the same request can vary with caching, data, and platform changes. Cell counts cannot be converted reliably into EECU-hours, so cost needs a measured benchmark.

Let `E` be measured EECU-hours for one centrally batched 10,000-location cycle. Under Limited and the first compute tier:

`monthly compute = 120 cycles × E × $0.40 = $48E`

| Measured EECU-hours/cycle | Limited compute estimate/month |
|---:|---:|
| 0.25 | $12 |
| 1 | $48 |
| 5 | $240 |
| 10 | $480 |

This is sensitivity analysis, not a forecast. At one EECU-hour/cycle, Basic would use 120 batch hours, so its rough total would be $500 + 20 excess hours × $0.40 = **$508/month**.

For comparison, direct client-like requests would cost `1,200,000 × s EECU-seconds ÷ 3,600 × $0.40`: one second/request is about $133/month; five seconds is $667; 30 seconds is $4,000, before concurrency and platform-fee concerns. Direct access is both architecturally and contractually unsuitable.

## GCS Zarr is efficient for bulk arrays, not point reads

The [GCS guide](https://developers.google.com/weathernext/guides/gcs) documents:

- `gs://weathernext/weathernext_2_0_0/zarr`
- `gs://weathernext/weathernext_2_0_0_mean/zarr`

The official [dissemination schedule](https://developers.google.com/weathernext/guides/dissemination) says all 64 members and the mean are released to GCS at 06:50, 12:50, 18:50, and 00:50 UTC for the 00, 06, 12, and 18 UTC initializations respectively—**6 hours 50 minutes after init**, typically ±15 minutes and occasionally ±60 minutes or more. This is a documented scheduled real-time feed, not an SLA.

Google's [Zarr starter notebook](https://storage.googleapis.com/weathernext-public/colabs/WeatherNext_2_Starter_Guide_Zarr_on_Google_Cloud_Storage.ipynb) shows the full array contract:

| Dimension | Size/values |
|---|---|
| `sample` | 64 members, IDs 0–63 |
| `time` | 60 lead times, +6h through +360h |
| `lat` | 721, −90° through +90° |
| `lon` | 1,440, 0° through 359.75° |
| `level` | 13 pressure levels |

Variables are `float32`. Surface fields have dimensions `(sample, time, lat, lon)` and upper-air fields add `level`. The notebook displays roughly 1 TB logical size for a full run and roughly 22 GB for a mean run.

The critical access detail is chunking: a surface variable chunk is one member × one lead × the **entire 721 × 1,440 global grid**. Reading a single location can therefore fetch a global field chunk. GCS is suitable for sequential/bulk Dask-style work, not uncoordinated point lookups.

Two 10 m wind components across a full run contain:

`1,038,240 × 60 × 64 × 2 × 4 bytes = 29.704 GiB logical/run`

At 120 runs/month that is 3.481 TiB of logical reads if every chunk is fetched once. Under the current [Cloud Storage pricing](https://cloud.google.com/storage/pricing):

- Same-location Google Cloud transfer may be $0.
- At an illustrative $0.10/GiB North America-to-Oceania inter-region rate: about **$356/month**.
- With the current internet-to-Australia tiers—first 1,024 GiB at $0.19 and the remaining 2,540.5 GiB at $0.18—the 3.481 TiB logical sensitivity is about **$652/month**.
- 64 × 60 × 2 × 120 = 921,600 Class B reads would be about **$0.37** at $0.0004/1,000 operations, if those operations are billable.

Cloud Storage's [current quotas and request-rate guidance](https://docs.cloud.google.com/storage/quotas) lists object reads/writes as autoscaling rather than fixed-rate quotas: a flat-namespace bucket initially supports roughly 5,000 reads/second and 1,000 writes/second, then scales, subject to bandwidth limits and ramp guidance. The proposed 921,600 reads/month averaged across scheduled batches is not a rate concern, but burst parallelism and the remote bucket's configuration still require measurement.

These are uncompressed logical-byte sensitivities. Actual compressed bytes, bucket location, cache hit rate, requester-pays status, and object visibility were not authenticated. Official documentation is internally awkward: the Earth Engine catalog calls the same 2022-present raw Zarr path historical, while the GCS and dissemination guides describe scheduled real-time releases in the documented buckets. Treat the real-time feed as documented but verify object naming, completeness, access controls, and billed transfer before making it a dependency.

## Vertex AI can generate selected hourly fields, but it is an expensive EAP job path

The [Vertex access guide](https://developers.google.com/weathernext/guides/access-vmg) confirms that only WeatherNext 2 is available and only through a limited Early Access Program. It is not a low-latency prediction endpoint: an approved project launches serverless Vertex Custom Training jobs on dedicated/single-tenant A100 80 GB or H100 80 GB resources, then reads Zarr output from its own GCS bucket.

The [Vertex model specification](https://developers.google.com/weathernext/guides/model-specs-vmg) gives:

- 0.25° global resolution.
- Initialization at 00, 06, 12, and 18 UTC.
- Default/customizable 15-day horizon.
- Default 64 members = four model seeds × 16 members.
- Historical generation from January 2024-present.
- Down-to-hourly output for only: 300/500 hPa geopotential and temperature; 1,000 hPa u/v wind; 100 m and 10 m u/v wind; 2 m temperature; mean sea-level pressure; and sea-surface temperature.
- Precipitation and the full atmospheric variable families remain six-hourly.

The official [EAP notebook](https://github.com/GoogleCloudPlatform/vertex-ai-samples/blob/main/notebooks/community/weathernext/weathernext_2_early_access_program.ipynb) exposes `num_samples`, `horizon_hrs`, `forecast_init_time`, `model_seed`, and `enable_hourly_prediction`; it creates custom jobs using `a2-ultragpu-{1,2,4,8}g` or `a3-highgpu-{1,2,4,8}g` shapes. Default relevant accelerator quota is zero and must be requested regionally. FLEX_START capacity may wait up to a configured day or indefinitely, so queue-to-output freshness is uncontrolled. No public SLA, maximum sample count, runtime benchmark, immutable model version, or total job cost was found.

An accelerator-only list-rate sensitivity from the current [Vertex AI pricing](https://cloud.google.com/vertex-ai/pricing) illustrates the scale. Assume 64 member-tasks each consume `R` GPU-hours:

| Accelerator snapshot | Rate/GPU-hour | Cost/cycle = 64 × rate × R | Monthly = 120 cycles × cycle cost |
|---|---:|---:|---:|
| A100 80 GB | $4.517292 | $289.11R | **$34,692.80R** |
| H100 80 GB | $11.75586073 | $752.38R | **$90,285.01R** |

If `R = 0.25` hours, the accelerator-only monthly examples are about $8,673 and $22,571. If `R = 1`, they are about $34,693 and $90,285. Host/management CPU and RAM, disk, GCS, queue time, retries, and failed jobs are excluded. Because runtime and the service's member-to-GPU scheduling are undocumented, these are not budget quotes.

Like scheduled BigQuery ingestion, one centrally generated forecast would serve all 1.2 million app refreshes; user count should not multiply Vertex runs. The unresolved EAP terms, legal restrictions, quota, latency, cost, and absence of advertised tracker outputs make Vertex unsuitable as the initial production path.

## The reference cyclone tracker cannot consume the advertised public schema

The pinned repository proves WeatherNext's cyclone implementation is substantive. Its direct tracker consumes model-native cyclone-existence probability grids and scalar fields for intensity, pressure, radius of maximum wind, and 34/50/64-knot quadrant radii. The `WeatherNext2` configuration predicts those quantities when the model itself is run.

The reviewed managed-data schemas advertise only meteorological grids. A tag saying “cyclones” in the Earth Engine catalog does not add cyclone bands or track records. A naive replacement based on local maximum 10 m wind would violate the brief and would not reproduce DeepMind's tracker.

This leaves three possible paths, none yet established:

1. Google provides an undocumented track product or cyclone-native fields to approved users.
2. Vertex output includes undocumented cyclone-native fields that can drive the pinned tracker.
3. Windward runs an appropriately licensed model/checkpoint itself with licensed HRES initialization data—explicitly out of scope for v1 and materially different in cost and operations.

Until one is evidenced and validated, Windward cannot implement per-member cyclone tracks, formation probability, wind radii, track fans, or a faithful ensemble-to-address cyclone probability.

## Recommended next steps, if the hold is cleared

1. Obtain the legal/service resolution listed in [`LICENSING.md`](LICENSING.md): either confirmation that the described use falls outside Google's High Risk Activities prohibition or an executed contractual exception, plus counsel review.
2. Submit the WeatherNext Data Request and archive the exact terms accepted.
3. Inspect and checksum the live BigQuery full and mean schemas; run dry runs for one initialization over global and Australian bounding-box workloads.
4. Ask `weathernext@google.com` whether an approved path exposes per-member cyclone tracks or the native existence/intensity/radius fields used by `direct_tracker.py`.
5. Pull one authorized cycle only after steps 1–4; measure availability latency, bytes billed, missing members/leads, and schema stability.
6. Benchmark Earth Engine EECU and, only if separately approved and budgeted, one small Vertex EAP run.
7. Decide explicitly whether the public product remains six-hourly or whether an independently validated hourly method is required. Do not label interpolated values as native hourly forecasts.

## Unresolved questions and blockers

| Priority | Question/blocker | Why it blocks progress | Required evidence |
|---|---|---|---|
| **P0** | Does Google's High Risk Activities restriction permit Windward's life-safety preparedness use? | All proposed managed paths are Google Cloud-backed and the app influences family/property decisions. | Written confirmation that the use is outside the prohibition, or an executed contractual exception, plus legal review. |
| **P0** | Can a public consumer Windward product lawfully display live probabilities, ensemble tracks/fans, or gridded layers under the real-time terms? | Public sharing is limited to non-retrievable value-added services; the terms also say the data is not intended for consumer use. | Written Google classification of the exact product outputs; see `LICENSING.md`. |
| **P0** | Where are the cyclone-native tracker inputs or an official per-member track product? | The pinned reference tracker cannot run on the advertised managed-feed schema. | Approved schema/output inspection or written Google answer. |
| **P0** | What is the exact live BigQuery schema and dry-run byte count? | The official notebook is internally inconsistent and cost spans orders of magnitude depending on pruning. | Authenticated `bq show`, `INFORMATION_SCHEMA`, and dry-run artifacts. |
| **P0** | Which timestamp controls the terms' 48-hour historic/real-time boundary? | The PDF says data “relates to a time,” while access guides refer to generation; init, valid, generation, and ingestion timestamps differ materially for a 15-day run. | Written Google rule for per-run/per-lead classification and any later transition. |
| **P1** | Is a live Zarr feed supported, what is its object naming, bucket location, and latency, and is requester-pays enabled? | Only the historical raw Zarr archive and general bucket paths are clear. | Authorized bucket listing and billing/config inspection. |
| **P1** | Can Vertex return cyclone-native outputs, how many samples are supported, and what runtime/SLA/version applies? | The public EAP schema does not answer these production questions. | EAP model card, order/terms, quota grant, and measured job. |
| **P1** | Is six-hourly temporal sampling acceptable for the arrival-window promise? | The brief currently asks for hourly endgame timing that the preferred public feed does not provide. | Product decision informed by meteorological validation. |
| **P1** | Is the 2022-present archive large and stable enough to calibrate Australian cyclone probabilities? | It is shorter than assumed and 64-member resolution may limit reliability analysis. | Event inventory, archive completeness audit, and pre-registered calibration plan. |

**Milestone 1 status: data access investigated; ingestion remains on hold.**
