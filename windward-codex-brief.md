# Windward — Engineering Brief

**For: an autonomous coding agent (Codex). Read this whole document before writing any code.**

---

## 0. How to use this brief

This is a product and engineering brief, not a line-by-line spec. Where it states *what* and *why*, treat it as binding. Where it leaves *how* open, use your judgement and document the decision in `DECISIONS.md`.

Three rules that override everything else:

1. **Verify before you build.** This project depends on a Google DeepMind data release that is only days old (August 2026). Do not trust any dataset name, table schema, API shape, or licence term asserted in this brief — including the ones below — without confirming it against live documentation. Where you cannot confirm something, build behind an interface, use a fixture, and record the open question. **Never invent a schema and proceed as if it were real.**
2. **Ask when blocked on a decision that costs money, changes scope, or touches safety.** Do not guess on those. Guess freely on everything else and note it.
3. **This app touches life safety.** A person may make a decision about their family or property based on a number this code renders. Correctness and honest uncertainty beat features, polish, and delivery speed. If you must choose between shipping a feature and shipping it correct, ship it correct or don't ship it.

---

## 1. What we're building

**Windward** — a cyclone preparedness app for Australia, built entirely on Google DeepMind's **WeatherNext 2** model family.

**The promise, in one line: one more day to get ready.**

WeatherNext's headline result is that it delivers roughly a day of extra lead time on tropical cyclone forecasts versus conventional approaches. That extra day is the product. It sits at the *front* of the warning sequence, where preparation is still calm — shops stocked, roads open, daylight available, choices intact. Everything the app does exists to hand a household that day and tell them how to spend it.

**What it does concretely:** a user saves their address. The app reads the WeatherNext 2 ensemble — hundreds to a thousand simulations of the coming fortnight — counts how many of those scenarios bring damaging cyclone winds over their location, and turns that count into one honest number with a deadline attached: *"23% chance of gale-force winds in the next 15 days. If it comes, winds arrive Friday 2–9am. Do the calm jobs by Tuesday."*

**Why it doesn't exist yet:** official warnings are authoritative but necessarily late — they fire once a system exists and is close. Public spaghetti-plot sites are raw and unreadable to normal people. Nobody turns a 1,000-member ensemble into a single actionable number, for a single Australian address, with a plan attached, a week or more out.

**Who it's for:** residents of cyclone-exposed Australia — Coral Sea coast (Cairns, Townsville, Mackay, Whitsundays, SE QLD), Top End (Darwin, Katherine, Gulf), and the north-west (Pilbara, Kimberley, Exmouth, Broome, Port Hedland). Secondary: anyone with exposed assets — boats, caravans, tourism bookings, remote crews, livestock, elderly relatives up north.

---

## 2. Non-negotiable constraints

These are product requirements, not legal boilerplate. Violating them is a failed build.

**2.1 We are a preparedness layer, never a warning service.** The Bureau of Meteorology (BOM) and state emergency services issue official warnings. Windward never contradicts, replaces, or visually outranks them.

- Persistent, always-visible status strip linking to BOM.
- When a BOM watch/warning is current for a user's area, an official banner takes over the top of every screen and our own risk tiers become visually secondary. This behaviour is mandatory and must be covered by tests.
- Never use the words "warning", "watch", or "alert" for our own tiers in a way that could be mistaken for official terminology. Our tiers are: **All clear / Watching / Heads up / Get ready**.
- Never advise evacuation, and never advise *against* acting on an official instruction. Our copy prepares; theirs directs.

**2.2 Honest uncertainty.** Every probability displays with its definition (threshold, time window, location basis). The lead-time benefit is described as an average improvement, not a per-storm guarantee. Never round a probability up for drama. Never suppress a scenario because it's inconvenient — the tracks that miss get drawn too; showing the misses is what earns trust for the hits.

**2.3 Licensing.** DeepMind has historically released model *code* under Apache 2.0 while releasing model *weights* under a non-commercial licence (CC BY-NC-SA). **Confirm the exact terms attached to the August 2026 WeatherNext release before the architecture depends on them.** Default to consuming forecast *data* through Google's commercial data products (BigQuery / Earth Engine / Vertex) rather than self-hosting weights. Record findings in `LICENSING.md` and flag any blocker immediately rather than working around it.

**2.4 Privacy.** Saved locations are personal safety data. Store the minimum, never sell or share it, never put coordinates in third-party analytics. Australian Privacy Principles apply. Location permission is optional — manual address entry must be a first-class path.

**2.5 Accessibility and resilience.** People use this during storms, on bad connections, on old phones, stressed. Therefore: cached last-known state that renders offline with a clear staleness timestamp; readable in bright sun; screen-reader correct (probabilities and time windows must be announced meaningfully, not as bare numbers); respects reduced-motion; works on a 4-year-old mid-range Android. Performance target: usable first paint under 3s on a throttled 3G connection.

---

## 3. Data layer — WeatherNext 2 is the only forecast engine

**Every number in this app derives from WeatherNext 2 output. There is no other forecast source anywhere in the value chain.** The only external feed is BOM official warnings, which is a safety/credibility overlay and must never feed our computed numbers.

### 3.1 Access — investigate first, then choose

Known entry points, all requiring verification:

- **BigQuery** — WeatherNext datasets published via Analytics Hub. Likely the primary production path: queryable, no model execution, commercially sanctioned.
- **Google Earth Engine** — WeatherNext catalog assets, gridded/geospatial access.
- **Vertex AI / Gemini Enterprise Agent Platform** — model-garden access for running custom forecasts.
- **GitHub `google-deepmind/weathernext`** — open-source code including cyclone tracking utilities. Even if we don't run the model, **the tracker implementation here is the reference we should follow.**
- **Developer docs** at `developers.google.com/weathernext`.

**Task 1 is a written access report** (`DATA_ACCESS.md`) covering, for each path: what's actually available, real schema, spatial/temporal resolution, ensemble member count, update cadence, latency from init time to availability, historical archive depth, quota limits, and cost per query at our expected volume. Include a worked cost projection for 10k users polling four times daily. **Do not begin the pipeline until this report exists.**

### 3.2 What we consume

Two uses of the same ensemble, not two models:

1. **Cyclone tracks** — per-member cyclone position, intensity, and wind structure, derived using DeepMind's own tracking approach applied across ensemble members. Do not hand-roll naive grid-max wind detection; follow the reference implementation.
2. **Gridded fields** — hourly wind, precipitation, and pressure from the same members, for the fine-grained endgame view and rainfall risk.

Cadence: forecasts issue roughly four times daily; ingest every cycle and **retain every historical run** — the archive powers Replay, calibration, and the "fan tightening" visualisation, and is unrecoverable if not captured from day one.

### 3.3 Derived quantities (the actual product logic)

For a given location and ensemble:

| Quantity | Definition |
|---|---|
| **Gale probability** | Fraction of members where sustained winds ≥ 63 km/h occur at the location within the horizon |
| **Destructive probability** | Same, at ≥ 125 km/h |
| **Arrival window** | Distribution of first-crossing times over threshold, across members that cross; report a central band (e.g. 10th–90th percentile) plus earliest |
| **Formation probability** | Fraction of members producing a tropical cyclone within a defined basin region and horizon |
| **Rapid intensification flag** | Fraction of members intensifying by a significant margin within 24h before closest approach |
| **Rainfall risk** | Percentile accumulations over the location/catchment |
| **Trend** | Change in each probability versus the previous run and versus 24h ago |
| **Lead-time gap** | For historical storms: hours between our threshold crossing and the official watch issuance |

**Design decisions you must make and document:** how a point location maps to model grid cells (nearest cell vs. interpolation vs. radius); how sustained wind relates to the model's wind fields; how to handle members where a cyclone forms but dissipates. These choices materially move the displayed numbers — reason them out explicitly in `METHODOLOGY.md`, written so a meteorologist could audit it.

**Calibration is a requirement, not a nice-to-have.** Hindcast the available archive (roughly 2020 onward) and produce reliability statistics: when we said 25%, how often did it happen? Publish this in-app. If the raw ensemble fraction is poorly calibrated, apply and document a correction — never silently.

---

## 4. Product surface

Three reference mockups accompany this brief. Match their intent and information hierarchy; improve on their execution.

### Home — the day, then the number
Head-start banner at the very top (the "+1 day" framing, timestamped when the head start began), then the probability as hero, definition, trend, arrival window, destructive-wind secondary figure, 15-day probability strip, a **"Use the day"** action card naming calm jobs with a deadline, and a watching-systems card. BOM status strip pinned.

### Map — the ensemble fan
Every member drawn as a faint track; median emphasised; misses included. Formation zone with probability. User location pinned with their number. Layers: scenarios / density / rain / official. Forward scrub through the horizon; **reverse scrub through past runs to watch the fan tighten** — this is the best uncertainty-education device in the app and it's nearly free given the archive.

### Prepare — checklist that unlocks by risk tier
Tasks staged so the calm jobs surface early and the urgent ones surface late. Progress persists across storms and seasons. Each tier links matching official guidance (Get Ready Queensland, SecureNT, DFES WA). This is the retention engine.

### Replay — proof of the day *(highest-priority differentiator)*
Rewind real storms (Alfred 2025, Jasper 2023, Kirrily 2024, Zelia 2025) and step through what Windward would have shown, day by day, against the official timeline. Headline output: **the lead-time gap in hours** between our "get ready" threshold and the official watch. Must be labelled a historical reconstruction, never a forecast. Methodology must be honest — no look-ahead bias, no threshold tuned per storm after the fact; thresholds are set globally and applied uniformly.

### Alerts
Tiers: **All clear** → **Watching** (basin formation probability above threshold) → **Heads up** (gale probability above a lower bar within 10 days) → **Get ready** (above a higher bar within 7 days, or destructive-wind probability meaningfully non-zero). Exact thresholds are a product decision — propose values grounded in the calibration work, don't invent them blind. Every notification contains: the number, the change, one action. Never a bare siren. Rate-limit aggressively; a de-escalation notice matters as much as an escalation.

---

## 5. Architecture

Stack is your call — choose for reliability, hiring pool, and cost, and justify it in `DECISIONS.md`. Constraints that should drive the choice:

- **Web-first, mobile-first.** During an event, links are what spread. A shareable URL for a location's risk beats an app-store install. Native wrapper later; design for it now.
- **Push notifications are core**, so account for web push limitations on iOS.
- **Scheduled ingestion** on the forecast cycle, decoupled from serving. Users must never wait on BigQuery.
- **Precompute** probabilities for populated locations rather than computing per request; serve from a fast store.
- **Traffic is extremely spiky** — near-zero for months, then a national event. Design and load-test for a two-order-of-magnitude spike. A cyclone approaching Brisbane is your worst day and your best day simultaneously.
- **Every displayed number must be traceable** to a specific model run and computation version. Store run IDs and methodology version alongside cached outputs; a number we can't explain later is a liability.

Structure the codebase so the **ensemble→probability computation is a standalone, pure, heavily-tested library** independent of both the data source and the UI. That module is the product's core IP and must be testable against fixtures with no network.

---

## 6. Build order

Each milestone ends in something demonstrable. Do not run ahead.

1. **Data access report** — `DATA_ACCESS.md` + `LICENSING.md`. Nothing else starts first.
2. **Ingestion spike** — pull one real forecast cycle, persist it, document the true schema.
3. **Core computation library** — derived quantities against fixtures, with tests. `METHODOLOGY.md` written here, not after.
4. **Replay / hindcast** — run the pipeline over Cyclone Alfred's history and produce the day-by-day probability series and lead-time gap. **This is the highest-value early artifact:** it validates the entire method and is the launch pitch. If the gap doesn't materialise, we need to know now, and we report it honestly rather than tuning until it appears.
5. **Calibration study** — reliability across the archive; publishable numbers.
6. **Frontend** — Home, Map, Replay, Prepare, per mockups.
7. **Alerts** — tiering, push, rate limiting, BOM override behaviour.
8. **Hardening** — offline, accessibility, load test the spike, monitoring.

**Timing context:** Australian cyclone season opens 1 November. Milestone 4 should be demo-able well before then.

---

## 7. Definition of done

- Every displayed number traceable to a model run and methodology version.
- Core computation library has meaningful tests against real fixtures, not just mocks.
- BOM override behaviour is tested, not merely implemented.
- Calibration results exist and are shown in-app, whatever they say.
- Accessible: keyboard navigable, screen-reader sensible, reduced-motion respected, sunlight-readable.
- Renders usable cached state offline with visible staleness.
- Load-tested for a 100× traffic spike.
- `DECISIONS.md`, `METHODOLOGY.md`, `DATA_ACCESS.md`, `LICENSING.md` all current.
- No copy anywhere that could be mistaken for an official warning.

---

## 8. Explicitly out of scope for v1

Storm surge modelling (requires bathymetry and hydrodynamics WeatherNext doesn't provide — link to official surge products instead). Self-hosted model inference. Non-cyclone hazards. Multi-country. Paid tiers. B2B dashboards. All are plausible later; none are v1.

---

## 9. Where to push back

If, while building, you conclude that any part of this brief is wrong — the method won't produce a real lead-time gap, the data access economics don't work, a threshold is unsafe, the architecture won't survive the spike — **say so directly and early rather than building it anyway.** A brief written before the data was inspected is a hypothesis, not an instruction.
