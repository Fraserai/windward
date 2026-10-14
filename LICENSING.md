# WeatherNext 2 licensing and commercial-use review

**Evidence date:** 7 August 2026 (Australia/Brisbane)  
**Release reviewed:** upstream `v0.3.0`, commit `89c4b2a77a1c57b328b909c575550fd2e5aadc9c`, committed 6 August 2026  
**Scope:** code, checkpoints, self-run outputs, historical forecast data, real-time forecast data, BigQuery/Analytics Hub, Earth Engine, GCS, and Vertex AI  
**Status:** factual engineering review, **not legal advice**

## Plain answer

**Windward is not presently cleared for a commercial life-safety launch.** The open-source components are more permissive than the brief assumed, but the managed real-time path is substantially more restrictive.

| Material/path | Commercial use in plain English | Windward decision |
|---|---|---|
| WeatherNext repository code and notebooks | **Yes.** Apache License 2.0 permits commercial use, modification, and distribution if its conditions are followed. | Usable as a pinned reference or dependency. |
| Downloadable WN2 and WeatherNext Cyclones checkpoints | **Yes.** The repository says non-code materials are CC BY 4.0, and inspected checkpoint metadata embeds CC BY 4.0. There is no NonCommercial or ShareAlike term. | Legally permissive at the artifact level; self-hosting remains out of v1 scope and input-data rights still need resolution. |
| Historical managed data “that relates to a time” more than 48 hours in the past | **Yes.** Google expressly licenses it under CC BY 4.0. | Suitable for research, Replay, and calibration subject to attribution, service terms while accessing it, and methodology/safety review. The controlling timestamp is unresolved. |
| Real-time managed data “that relates to a time” no more than 48 hours in the past | **Conditional, not a blanket yes.** Commercial activity is not expressly banned, but use and sharing are confined to the real-time terms' enumerated purposes and value-added-service rules. | **Blocked for the proposed public consumer product** until Google classifies the exact outputs in writing and counsel approves. The controlling timestamp is unresolved. |
| Earth Engine in a commercial backend | **Conditional.** Paid commercial use exists, but direct end-user API access is restricted and all Google Cloud terms still apply. | Backend exploration only after clearance; not a client-facing API. |
| Vertex WeatherNext 2 | **Unknown/blocked.** It is an allowlisted pre-GA EAP; no public WeatherNext-specific output licence, Model Garden terms, or order terms were available for review. | Do not architect around it until the complete EAP contract and written use-case approval are obtained. |

Two independent issues produce the hold:

1. The [Google Cloud Terms of Service](https://cloud.google.com/terms) prohibit use of the services for “High Risk Activities,” defined broadly around reasonably expected death, injury, or property/environmental damage if the service is used or fails. Windward is expressly designed to influence household and property preparation during cyclones. Whether a non-authoritative preparedness layer is inside that definition is a legal/contract question, but assuming it is outside would be unsafe.
2. The [real-time WeatherNext terms](https://storage.googleapis.com/weathernext-public/terms-of-use.pdf) call the data experimental and not intended for consumer use. Public sharing is permitted for findings and **Non-Retrievable Value Added Services**, not for raw data, mere formatting/subsetting, or freely available retrievable products. Windward's address probabilities may be sufficiently transformed; its ensemble fans, tracks, and gridded layers may not be. Only Google can give a dependable classification of the actual screens and APIs.

## What “August 2026 release” means

There are two timelines that should not be conflated:

- The pinned open-source `v0.3.0` commit dated 6 August 2026 is titled **“Add WN2 support.”** Its repository README and file headers establish the code/material licences reviewed here.
- Google's [WeatherNext release notes](https://developers.google.com/weathernext/release-notes) list the managed WeatherNext 2 v1.0 dataset as available in BigQuery and Earth Engine from 3 October 2025. The July 2026 access pages describe migration away from deprecated WeatherNext Graph and Gen datasets. No separate August 2026 managed-feed licence was found.

The applicable real-time forecast terms are a PDF last modified 12 November 2025. The BigQuery and GCS guides were last updated 29 July 2026. The open-source guide was updated 6 August 2026. Online terms can change, so the version, acceptance timestamp, URLs, and checksums must be archived when access is granted.

## Repository code: Apache 2.0 permits commercial use

The pinned [repository README licence section](https://github.com/google-deepmind/weathernext/blob/89c4b2a77a1c57b328b909c575550fd2e5aadc9c/README.md#L161-L178) says Colab notebooks and associated code are under Apache 2.0. Python source files, including the WN2 and cyclone tracker modules, carry Apache headers, and the repository includes the full [Apache 2.0 text](https://github.com/google-deepmind/weathernext/blob/89c4b2a77a1c57b328b909c575550fd2e5aadc9c/LICENSE).

Commercial answer: **yes**.

If Windward distributes the code or a derivative, the important engineering obligations include:

- Provide a copy of Apache 2.0.
- Retain required copyright, patent, trademark, and attribution notices from source form.
- Mark files that Windward modifies with prominent change notices.
- Preserve a `NOTICE` file's attributable notices if the upstream work includes one. The pinned repository has no root `NOTICE`, but dependencies may have their own requirements.
- Do not imply Google endorsement or treat Apache 2.0 as a trademark licence.

Apache 2.0 includes an express patent grant from contributors, subject to its termination conditions. It provides the work as-is and does not make the model suitable for safety decisions.

## Checkpoints: CC BY 4.0, not the historical non-commercial licence assumed in the brief

The repository README separately identifies the [provided WN2 and WeatherNext Cyclones weights](https://github.com/google-deepmind/weathernext/blob/89c4b2a77a1c57b328b909c575550fd2e5aadc9c/README.md#L43-L84) and states the [Apache/CC BY licence split](https://github.com/google-deepmind/weathernext/blob/89c4b2a77a1c57b328b909c575550fd2e5aadc9c/README.md#L161-L178). Google's [open-source model guide](https://developers.google.com/weathernext/guides/osmodel) repeats that split.

This review also inspected the embedded `license.npy` metadata in two downloadable checkpoint objects:

- [WeatherNext2_<2025 model 1 checkpoint — direct `.npz` download](https://storage.googleapis.com/dm_graphcast/weathernext2/params/WeatherNext2_%3C2025_model1.npz)
- [WeatherNext Cyclones Mini <2024 checkpoint — direct `.npz` download](https://storage.googleapis.com/dm_graphcast/weathernext2/params/WeatherNextCyclones_Mini_%3C2024.npz)

Both identify CC BY 4.0 and point to the ERA5/HRES/IBTrACS attribution material. The checkpoint loader exposes the licence field in [`fgn.py`](https://github.com/google-deepmind/weathernext/blob/89c4b2a77a1c57b328b909c575550fd2e5aadc9c/weathernext/weathernext2/fgn.py#L48-L52).

Commercial answer: **yes at the checkpoint-licence level**. [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/legalcode) permits sharing and adaptation for any purpose, including commercial use. When Windward shares licensed material or an adaptation, its attribution conditions apply: appropriate credit, a licence link, an indication of modifications, retention of supplied notices, and no implication of endorsement. CC BY 4.0 does not license patent or trademark rights; moral, privacy, and publicity rights are waived or not asserted only to the limited extent stated, and third-party rights remain separate. It expressly addresses applicable sui generis database rights.

The earlier GraphCast/GenCast non-commercial history does not control these WN2 artifacts. The pinned legacy READMEs separately record their August 2026 switch to commercial-use-compatible weights, but Windward's decision should be based on each exact WN2 object's embedded metadata, not the family history.

## Self-run model outputs: potentially commercial, but not a cleared v1 route

The open checkpoint licence does not itself impose a non-commercial restriction on generated forecasts. The repository disclaimer says users are responsible for determining whether using or distributing models and outputs is appropriate, and stresses that the project is experimental, unsupported, not government-endorsed, and not a replacement for official alerts or warnings.

That does **not** create a complete commercial production clearance:

- WN2 initialization uses HRES; training and examples also reference ERA5, HRES, Copernicus, and IBTrACS. Their exact licence and access terms must be mapped for Windward's use, storage, redistribution, and territory.
- If inference runs on Google Cloud, the Google Cloud High Risk Activities restriction still applies independently of the checkpoint licence.
- If current HRES or WeatherNext managed data is used as an input, its own real-time or ECMWF terms may attach.
- The repository offers research code with no API-stability, support, operational latency, or safety assurance.
- Self-hosted inference is explicitly outside Windward v1 and materially changes cost, staffing, validation, and operational risk.

Commercial answer: **potentially yes, but not cleared from the checkpoint licence alone**.

## Historical forecast data: CC BY 4.0 permits commercial use

The [BigQuery guide](https://developers.google.com/weathernext/guides/bigquery), [GCS guide](https://developers.google.com/weathernext/guides/gcs), [Earth Engine full collection](https://developers.google.com/earth-engine/datasets/catalog/projects_gcp-public-data-weathernext_assets_weathernext_2_0_0), [Earth Engine mean collection](https://developers.google.com/earth-engine/datasets/catalog/projects_gcp-public-data-weathernext_assets_weathernext_2_0_0_mean), and real-time terms all draw a 48-hour boundary, but do not describe it consistently:

- The controlling PDF calls data “Historic” or “Real-Time” according to whether it **relates to a time** more than, or no more than, 48 hours in the past.
- The BigQuery and GCS guides instead paraphrase the real-time class as forecasts **generated within** the past 48 hours.

For a 15-day forecast, initialization time, valid/forecast time, generation time, and ingestion time can produce radically different classifications. The public sources do not say which timestamp controls, whether classification is per run or per lead cell, or whether a run transitions later. Windward must not automate a 48-hour reclassification until Google answers this in writing.

Commercial answer for historical data: **yes**.

Windward must still:

- Attribute the exact WeatherNext dataset/model and link CC BY 4.0. The catalog-prescribed historical notices differ: the full ensemble uses a 2025 Google/DeepMind notice, while the mean collection specifies ©2026 DeepMind Technologies Limited. Do not substitute one generic notice for both.
- Indicate transformations and derived calculations.
- Carry the dataset's Google, ECMWF, Copernicus, HRES, and other supplied acknowledgements/disclaimers where required.
- Avoid implying Google, DeepMind, ECMWF, BOM, or another meteorological agency endorses Windward.
- Follow the Google Cloud agreement while using BigQuery, Earth Engine, or GCS to retrieve the data.

Historical Replay and calibration are lower-risk uses than a live consumer recommendation, but that is an inference, not a contractual exemption. The High Risk Activities clause turns on the expected consequences of use or failure, so counsel should assess the actual workflow.

### Analytics Hub delivers the dataset; it does not license it

Analytics Hub is the BigQuery subscription/delivery mechanism, not a grant of commercial rights. The WeatherNext dataset page and the GDM real-time PDF govern the dataset licence, while BigQuery use separately remains subject to the Google Cloud agreement. The [BigQuery Sharing terms](https://cloud.google.com/terms/service-terms) also state that Google is not responsible for datasets or publisher/subscriber terms. Windward must archive and comply with both layers.

## Real-time forecast data: allowed purposes are narrower than “commercial use”

The [GDM Real-Time Weather Forecasting Experimental Data Terms of Use](https://storage.googleapis.com/weathernext-public/terms-of-use.pdf) apply across BigQuery, Earth Engine, GCS, and other linked surfaces for data that, in the PDF's wording, relates to a time no more than 48 hours in the past. The timestamp ambiguity described above remains unresolved.

### Rights granted

The licence is non-exclusive, royalty-free, **revocable**, non-transferable, and non-sublicensable except where the terms expressly allow. It permits:

- Any internal purpose.
- Creation and sharing of a qualifying Value Added Service (VAS).
- Limited controlled sharing with identified third parties for education, controlled subsidiaries, and service contractors.

Google does not currently charge a data-access fee under this document but can introduce reasonable fees with at least one month's written notice. BigQuery, Earth Engine, storage, and compute usage charges remain separate.

### A qualifying VAS must add functional value

A VAS must be derived from the real-time data for a specific purpose that the raw data alone cannot achieve. The terms expressly say that simple access/hosting and operations such as colouring, formatting, compression, fixed percentage adjustment, geometric transformation, geographic subsetting, or custom combinations of timesteps/parameters/runs do not by themselves create a VAS.

Windward's calibrated address probability, arrival distribution, trend, and action layer may be a genuine VAS because they answer a household-specific decision question. That is an **inference**, not confirmation from Google.

### Public sharing depends on retrievability

- A **Non-Retrievable VAS**, from which the source data cannot be recovered or reverse engineered without significant technical effort or expense, may be published to third parties.
- A **Retrievable VAS** may be supplied only through controlled transmission to known, identified third parties for their internal use, with no onward sharing or generation of another VAS.
- Raw or merely reformatted/subset real-time data cannot be offered as an anonymous public product.

This produces a screen-by-screen risk:

| Windward output | Preliminary classification | Why it needs Google confirmation |
|---|---|---|
| One calibrated probability and broad arrival band at an address | Plausibly non-retrievable VAS | Aggregation may prevent reconstruction, but repeated address/time queries could reveal underlying fields. |
| Per-member cyclone tracks and an ensemble fan | Likely retrievable or close to source output | It exposes member-level spatiotemporal structure and may be a direct tracker transformation. |
| Rain, wind, pressure, or density raster layers | Likely raw/modified or retrievable | Colouring, geometric transformation, subsetting, and combining fields are expressly insufficient on their own. |
| Public API returning member values or track points | Likely not permitted anonymously | It makes extraction and onward use straightforward. |
| Aggregate research finding about hindcast performance | Expressly contemplated as a finding | Still requires the prescribed citation and disclaimer. |

Do not use this preliminary classification as a launch decision. Send Google the exact mockups, response payloads, query-rate controls, and reconstruction analysis and ask for a written determination.

### Required notices, geography, termination, and safety statements

The terms impose different notices for raw/retrievable sharing versus findings/non-retrievable services. The latter requires citation of the Google access product/service, a Google copyright statement, a link to the terms, and Google's full experimental/not-real-world-use disclaimer. Copy the required wording directly from the operative PDF at implementation time rather than relying on this summary.

Additional operational constraints include:

- No access/use or product availability in Japan, South Korea, Indonesia, Cuba, Iran, North Korea, Crimea, or the so-called Donetsk and Luhansk People's Republics unless Google agrees otherwise in writing; export and sanctions restrictions also apply.
- The data is approximate, informational, experimental, and not intended for consumer use; it does not replace government alerts, warnings, or notices.
- No representation of Google or meteorological-agency sponsorship, endorsement, or official status.
- Google may change or discontinue the data and may suspend or terminate rights. Following termination by Google, the user must delete and stop using both real-time data and VAS in its control and notify recipients to stop using the VAS.
- Google's aggregate liability under these terms is capped at USD $500, subject to applicable law.
- The [Google Generative AI Prohibited Use Policy](https://policies.google.com/terms/generative-ai/use-policy) is incorporated.

For an Australian-only v1, the listed-country availability restriction is not an immediate market limitation, but it affects travellers, API access, CDN policy, and any future multi-country release.

## Google Cloud terms independently block an assumption of production safety clearance

### High Risk Activities

Section 3.3 of the [Google Cloud Terms](https://cloud.google.com/terms) prohibits customers and end users from accessing or using services for High Risk Activities. Its definition covers activities where service use or failure would reasonably be expected to cause death, personal injury, or environmental or property damage.

Windward is designed to tell households when to prepare property and family for cyclones. The product intends to remain subordinate to BOM and never direct evacuation, which may reduce risk, but the brief itself says people may act on its numbers. **Engineering cannot resolve this contract interpretation.** BigQuery, Analytics Hub, Earth Engine, GCS, and Vertex use must stay out of production until Google confirms the intended use in writing and Australian counsel reviews that confirmation.

### Earth Engine

The [service-specific terms](https://cloud.google.com/terms/service-terms) say the free/research Earth Engine edition has separate non-commercial terms, while the [commercial-transition guide](https://developers.google.com/earth-engine/guides/transition_to_commercial) directs operational users to a paid Cloud project. Windward also may not allow end users to access or interact directly with Earth Engine APIs unless those users have their own eligible Google Cloud accounts and access. A backend that returns Windward-generated derived outputs is the practical topology for an anonymous consumer app, subject to the other clearances.

### Vertex pre-GA and AI/ML restrictions

The [WeatherNext Vertex guide](https://developers.google.com/weathernext/guides/access-vmg) and the service-specific terms make this EAP/pre-GA path subject to the following constraints:

- Available as-is; changeable, suspendable, or discontinuable without prior notice.
- Outside SLAs and Google indemnity unless Google expressly says otherwise.
- Unless an express written notice or Google documentation says otherwise, no data-processing terms—including the Cloud Data Processing Addendum—apply, and the customer should not use the offering to process personal or legally regulated data.
- Terminable by either party at any time on written notice.

The AI/ML provisions prohibit using an AI/ML service or generated output to develop a similar or competing product/service when a Google pre-trained model is involved, using output to substitute for or improve a similar Google model, and reverse engineering model components. Windward appears to be a downstream preparedness application rather than a competing weather model, but that is not an adequate basis for launch. The EAP Model Card, Scope of Use, order form, output rights, and written non-competition interpretation are required. No identifiable address or geolocation may be sent to the Vertex EAP unless its operative documentation expressly permits that processing.

## Minimum compliance design if Google and counsel clear the product

This is a requirements checklist, not authorization to proceed:

1. Store the accepted version, timestamp, URL, and checksum of every applicable term and dataset notice.
2. Record the licence class with every source run: `historic_cc_by_4_0` or the precise real-time-terms version.
3. Preserve Google's supplied classification and relevant init/valid/ingestion timestamps, but do not automatically move a run or lead from real-time handling to CC BY 4.0 until Google defines the controlling 48-hour timestamp and transition rule in writing.
4. Keep raw/member-level live data server-side with least-privilege access and no public download endpoint.
5. Test that every public live output is non-retrievable under the written Google classification; rate limiting alone is not sufficient if reconstruction remains easy.
6. Render the exact prescribed attribution and experimental-use disclaimer wherever required, alongside an always-visible BOM authority link and current official status.
7. Do not expose Google, DeepMind, WeatherNext, ECMWF, or BOM marks as if they sponsor or endorse Windward.
8. Geofence and enforce restricted-country rules at access and delivery layers if those terms remain unchanged.
9. Provide a kill switch and deletion/recipient-notification runbook for termination of real-time rights.
10. Review every model/checkpoint and upstream input object independently; do not infer a licence from its filename or family.

## Unresolved questions and blockers

| Priority | Required answer | Owner/evidence needed |
|---|---|---|
| **P0** | Is Windward's specific preparedness use outside the Google Cloud High Risk Activities prohibition? | Either written confirmation that the described use is outside the prohibition or an executed contract amendment/exception that expressly permits it, plus an Australian technology/product counsel opinion. |
| **P0** | Are address probabilities and arrival bands Non-Retrievable VAS, and may they be shown anonymously to consumers? | Written determination from `weathernext@google.com` after reviewing exact outputs and reconstruction risk. |
| **P0** | May Windward publicly render per-member tracks, track fans, density maps, and gridded weather layers? | Written Google permission; the current terms make these high-risk classifications. |
| **P0** | Does the “not intended for consumer use” language permit this product at all, even with strong disclaimers and BOM precedence? | Google confirmation and counsel review. |
| **P0** | What terms govern Vertex WeatherNext output and this downstream use? | EAP acceptance, Model Card, Scope of Use, order form, output licence, and written non-competition interpretation. |
| **P0** | Which timestamp controls the 48-hour historic/real-time boundary: initialization, valid time, generation, ingestion, or another rule? | Written Google interpretation covering per-run versus per-lead classification and any transition over time. |
| **P1** | What exact HRES/ERA5/Copernicus/IBTrACS rights attach if Windward self-runs or stores inputs/derived output? | Object-level provenance and licence register; ECMWF/Google clarification where necessary. |
| **P1** | What exact attribution stack and placement satisfy Google, CC BY, ECMWF, and product-safety requirements without implying endorsement? | Counsel-approved attribution/disclaimer specification and, if marks are used, trademark approval. |
| **P1** | How will restricted territories, termination deletion, and recipient notification be enforced? | Approved product policy and operational runbook before any real-time access. |

**Milestone 1 licensing status: open code, weights, and historical data are commercially permissive; the proposed live Windward service remains blocked.**
