# WeatherNext repository inspection report

**Inspection date:** 7 August 2026 (Australia/Brisbane)  
**Upstream:** [google-deepmind/weathernext](https://github.com/google-deepmind/weathernext)  
**Local checkout:** [`vendor/weathernext`](vendor/weathernext)  
**Pin record:** [`vendor/weathernext.PIN.md`](vendor/weathernext.PIN.md)

## Result

The repository is **not** limited to GraphCast and GenCast. The pinned 6 August `v0.3.0` revision contains real WeatherNext 2 model code, serialized runtime configurations, checkpoint/loading utilities, demonstration notebooks, and a substantive cyclone-processing and direct-tracking package.

It does **not** contain a production forecast-feed client, operational forecast data, model weights in Git, a public cyclone-track feed, Windward's address-level probability logic, or production-grade WN2/cyclone tests. The reference tracker operates on model-native cyclone fields that the currently advertised BigQuery/Earth Engine/GCS schemas do not expose. That source-to-tracker mismatch is the repository audit's main blocker.

## Exact upstream pin

| Property | Recorded value |
|---|---|
| Remote | `https://github.com/google-deepmind/weathernext.git` |
| Upstream branch at clone | `main` (`origin/HEAD -> origin/main`) |
| Release tag | `v0.3.0` |
| Tag type | Lightweight |
| Commit | `89c4b2a77a1c57b328b909c575550fd2e5aadc9c` |
| Tree | `bbea42633bb2b9cbef3e89843d2f91d69b120abd` |
| Author timestamp | `2026-08-06T14:53:06+00:00` |
| Commit timestamp | `2026-08-06T14:54:52+00:00` |
| Subject | `Add WN2 support.` |
| Signature status | Unsigned |
| Clone state | Full history, detached at the exact commit; not shallow |
| Submodules | None declared |

The vendored repository is clean and unmodified. The pin file lives beside, not inside, the checkout so future Windward integration can wrap the upstream source without changing it.

Reproduction commands:

```bash
git clone https://github.com/google-deepmind/weathernext.git vendor/weathernext
git -C vendor/weathernext checkout --detach 89c4b2a77a1c57b328b909c575550fd2e5aadc9c
git -C vendor/weathernext rev-parse HEAD
git -C vendor/weathernext rev-parse HEAD^{tree}
```

## WeatherNext 2 implementation is present

The upstream [README at the pinned commit](https://github.com/google-deepmind/weathernext/blob/89c4b2a77a1c57b328b909c575550fd2e5aadc9c/README.md#L1-L18) identifies the repository as the home of WeatherNext 2, with GraphCast and GenCast retained as older models.

| Component | Evidence | What it implements |
|---|---|---|
| WN2 architecture | [`weathernext/weathernext2/architecture.py`](vendor/weathernext/weathernext/weathernext2/architecture.py) | `ForwardPass`, points-to-mesh/mesh-to-grid GNN blocks, a sparse-transformer mesh processor, and grid decoding. |
| Functional Generative Network wrapper | [`weathernext/weathernext2/fgn.py`](vendor/weathernext/weathernext/weathernext2/fgn.py) | Stochastic noise generation, ensemble prediction, inference/training paths, CRPS loss, and checkpoint metadata type. |
| Runtime configs | [`weathernext/weathernext2/configs`](vendor/weathernext/weathernext/weathernext2/configs) | `WeatherNext2`, `WeatherNextCyclones`, and `WeatherNextCyclones_Mini` serialized Fiddle configurations. |
| Shared model infrastructure | [`weathernext/utils`](vendor/weathernext/weathernext/utils) | Checkpoint/config loading, inputs, normalization, autoregressive rollout, sharding, graph/transformer blocks, loss functions, and JAX/xarray utilities. |
| End-to-end example | [`docs/weathernext2/wn2_demo.ipynb`](vendor/weathernext/docs/weathernext2/wn2_demo.ipynb) | Downloads a checkpoint and sample HRES input, creates the model, runs an ensemble rollout, computes loss/gradient examples, and runs the cyclone tracker per sample. |

The [upstream model inventory](https://github.com/google-deepmind/weathernext/blob/89c4b2a77a1c57b328b909c575550fd2e5aadc9c/README.md#L43-L88) describes:

- `WeatherNext2_<2025_model{1,2,3,4}.npz`: 0.25°, operational initialization from HRES, trained through 2024.
- `WeatherNextCyclones_<2025_model{1,2,3,4}.npz`: 0.25°, used live during the 2025 Atlantic season, trained through 2024.
- WeatherNext Cyclones 2024 and 2023 evaluation checkpoints.
- `WeatherNextCyclones_Mini` 2024 and 2023 checkpoints at 1°, intended for lower-memory testing rather than equivalent performance.

Weights are downloadable separately from Google's `dm_graphcast` bucket. They are not committed in this checkout.

## The WN2 configuration predicts cyclone-native fields

[`WeatherNext2.json`](vendor/weathernext/weathernext/weathernext2/configs/WeatherNext2.json) has 31 target-variable entries. The meteorological targets include pressure-level temperature, geopotential, u/v wind, vertical velocity, and specific humidity, plus surface temperature, pressure, 10 m and 100 m wind components, sea-surface temperature, and six-hour precipitation.

Its cyclone-specific targets include:

- `cyclone_exists_gaussian_unit_mode`
- `cyclone_all_wind_disc`
- `cyclone_usa_wind_disc`
- Northeast, southeast, southwest, and northwest 34-knot wind radii
- The same four quadrants for 50-knot and 64-knot radii
- `cyclone_usa_rmw_disc` (radius of maximum wind)
- `cyclone_usa_pres_disc` (pressure)

This matters because it proves the tracker is not meant to discover storms from a generic local wind maximum. It consumes a model deliberately trained to emit cyclone existence, intensity, and structure diagnostics.

The README says WeatherNext 2 and WeatherNext Cyclones use the same cyclone algorithm; WN2 additionally predicts 100 m wind, while independently trained checkpoints differ.

## Cyclone tracking support is substantive

The [`weathernext/cyclones`](vendor/weathernext/weathernext/cyclones) package contains 11 Python modules and 5,208 lines at the pinned revision.

| Module | Role |
|---|---|
| [`direct_tracker.py`](vendor/weathernext/weathernext/cyclones/direct_tracker.py) | Greedy direct tracker over cyclone-existence fields; observed-storm or direct-cyclogenesis initialization; momentum first guess; mean/mode/mode-then-mean location refinement; dissipation; intensity/radius extraction; nearby-track pruning; physical-consistency enforcement. |
| [`direct_tracker_6h_v1_config.py`](vendor/weathernext/weathernext/cyclones/direct_tracker_6h_v1_config.py) | Concrete six-hour tracker configuration, including search/refinement radii, genesis/dissipation thresholds, minimum duration, momentum, pruning, and physical-consistency settings. |
| [`tracker_base.py`](vendor/weathernext/weathernext/cyclones/tracker_base.py) and [`tracker_utils.py`](vendor/weathernext/weathernext/cyclones/tracker_utils.py) | Tracker interfaces and shared track utilities. |
| [`ibtracs_netcdf_to_csv.py`](vendor/weathernext/weathernext/cyclones/ibtracs_netcdf_to_csv.py) | Converts IBTrACS NetCDF data to track records. |
| [`ibtracs_processing_stages.py`](vendor/weathernext/weathernext/cyclones/ibtracs_processing_stages.py) | Pipeline for converting IBTrACS observations into gridded cyclone training/validation fields. |
| [`ibtracs_processing_utils.py`](vendor/weathernext/weathernext/cyclones/ibtracs_processing_utils.py), [`cyclone_utils.py`](vendor/weathernext/weathernext/cyclones/cyclone_utils.py), and [`constants.py`](vendor/weathernext/weathernext/cyclones/constants.py) | Variable definitions, geodesic/physical helpers, interpolation, normalization, and field conventions. |
| [`data_pipeline_utils.py`](vendor/weathernext/weathernext/cyclones/data_pipeline_utils.py) | Pipeline support for staged cyclone-data processing. |

The tracker algorithm's own docstring gives the essential sequence:

1. Initialize known cyclone centres from IBTrACS or detect genesis candidates from high-probability regions in the model's cyclone-existence field.
2. Advance each centre greedily using its previous motion as a first guess.
3. Refine the position using mean, mode, or mode-then-mean probability-field methods.
4. Stop a track when existence probability falls below the configured threshold, unless forced through all leads.
5. Extract matching scalar intensity/radius fields and return pandas track records.

The six-hour v1 configuration uses mode-then-mean refinement, 250 km location/refinement radii, a 500 km existence-probability radius, 500 km genesis separation/pruning distance, a 0.3 initial maximum-probability threshold, removal of cyclogenesis tracks lasting 2.5 days or less, 0.9 momentum, and physical quadrant/wind consistency enforcement. Those settings are upstream evidence, not automatically validated Windward methodology.

[`demo_gridding_and_tracking.ipynb`](vendor/weathernext/docs/cyclones/demo_gridding_and_tracking.ipynb) demonstrates an IBTrACS-to-grid-to-tracker round trip with numerical assertions. [`wn2_demo.ipynb`](vendor/weathernext/docs/weathernext2/wn2_demo.ipynb) loops over `predictions.sample`, preprocesses each member, and produces one track DataFrame per ensemble member.

## What the repository does not provide

| Missing capability/artifact | Evidence and consequence |
|---|---|
| Production forecast-feed client | BigQuery, Earth Engine, Vertex, and forecast-access pages appear as links, but no service client, live schema binding, ingestion scheduler, retry/freshness logic, or cost control exists. |
| Forecast datasets or weights in Git | No tracked NetCDF, Zarr, checkpoint, or forecast objects were found. The demo anonymously downloads fixed checkpoint/sample-HRES paths from `dm_graphcast`. |
| Official managed cyclone tracks | No code points to a public track API/table/asset. The tracker generates tracks only after receiving the necessary gridded cyclone diagnostics. |
| Address-level risk products | There is no gale/destructive probability, arrival-window distribution, formation probability by user basin, trend-versus-prior-run, calibration, or BOM overlay. |
| Ensemble aggregation | The demo tracks members independently; it does not combine them into Windward metrics or an ensemble fan product contract. |
| Operational guarantees | The README calls this unsupported research code, as-is, with no API-stability promise. |
| WN2/cyclone unit-test suite | Six shared-utility test files exist, but no WN2-specific or cyclone-module unit-test files were found. |
| Recorded notebook execution evidence | Committed notebooks have null execution counts and no stored outputs. Their assertions are examples, not an upstream CI record. |

GraphCast and GenCast remain under `weathernext/weathernext1_graph` and `weathernext/weathernext1_gen`, explicitly labelled older models. Their presence does not reduce the WN2/cyclone evidence above.

## Integration boundary for Windward

The pinned direct tracker is the correct behavioral reference, but Windward should not edit the vendored checkout. A future integration would need a wrapper outside `vendor/` that:

- Converts an authorized source run into the exact xarray dimensions, coordinates, names, units, and cyclone-native fields expected by the tracker.
- Runs the tracker separately for every ensemble member and retains misses, dissipations, and genesis cases.
- Records source run ID, upstream commit, tracker config/version, input checksum, and methodology version.
- Produces Windward's separately specified and calibrated ensemble-to-location quantities in a pure tested library.

No such wrapper or application code was created in Milestone 1. More importantly, the authorized managed data contract currently appears unable to supply the tracker inputs; creating an adapter cannot fix absent data.

## Verification performed

| Check | Result |
|---|---|
| `git fsck --full` | Passed; object database verified. |
| `git rev-parse HEAD` and tree resolution | Matched the commit/tree in the pin table. |
| Tag/commit inspection | `v0.3.0` points directly to the commit; tag is lightweight; commit signature status is `N` (unsigned). |
| Shallow/submodule inspection | Full clone; no `.gitmodules` or submodules. |
| `git diff --exit-code` and porcelain status | Clean vendor checkout. |
| `python3 -m compileall -q weathernext` | Python syntax compilation passed under Python 3.11.3; generated caches were removed afterward. |
| JSON parsing | All three WN2 config JSON files parsed successfully. |
| `python3 setup.py --name --version` | Returned `weathernext 0.3.0`. |
| Tracked artifact search | No weights/checkpoints, forecast datasets, or sample-data files committed; serialized model configurations are present. |
| Test inventory | Six shared-utility test modules; zero WN2-specific and zero cyclone-specific test modules. |

Full runtime tests were not run because the workspace does not have JAX and most declared scientific dependencies installed. Installing them or executing accelerated model inference would expand scope and cost; neither was needed to answer Milestone 1. Syntax compilation does not establish numerical correctness, dependency compatibility, tracker skill, or safety.

## Unresolved questions and blockers

| Priority | Question/blocker | Required resolution |
|---|---|---|
| **P0** | Can approved BigQuery, Earth Engine, GCS, or Vertex output provide `cyclone_exists_gaussian_unit_mode` and the intensity/radius fields used by the tracker? | Authenticated output/schema inspection or written Google confirmation. |
| **P0** | If managed outputs omit those fields, is there an official per-member WeatherNext track product? | Documented endpoint/table/object and licence. |
| **P0** | If neither exists, is self-hosting the only faithful tracker path—and can it be licensed, initialized, operated, and validated for this product? | Scope decision only after legal, data-input, compute, and safety diligence. It remains out of v1. |
| **P1** | Which exact upstream tracker version/config corresponds to the operational managed feed? | Google version mapping and a golden real-run fixture. |
| **P1** | Does the direct tracker generalize and calibrate adequately for Australian basins and Windward's address-level wind thresholds? | Independent meteorological validation and hindcast study; repository code alone does not establish this. |
| **P1** | What ongoing update process protects reproducibility without freezing known defects? | Vendor update policy, signed/checksummed releases, changelog review, regression fixtures, and explicit methodology versioning. |

**Milestone 1 repository status: cloned, pinned, and verified; genuine WN2/cyclone code confirmed; production data-to-tracker contract unresolved.**
