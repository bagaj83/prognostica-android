# AGENTS.md — PROGNOSTICA DAS MVP

## 1. Project context

Project: **PROGNOSTICA — minimal DAS module**.

Primary working specification:
- `DAS_WORKING_SPEC_v0.2_2026-10-05.docx`
- Status: WORKING / NON-CANONICAL pilot specification, not proof of implemented field capability.
- The authoritative project constraints for this work are also defined by `01_ARCHITECTURE_AND_TECHNICAL_SPECIFICATION`, `03_AI_ML_MODELS`, `04_DAS_DFOS`, and `07_DECISIONS_AND_PROTOCOLS`.
- If the DOCX is not available in the coding environment, follow this `AGENTS.md` for M1–M2 and do not invent missing requirements; request the exact specification excerpt when needed.

Current development scope: **M1–M2** from the specification:
1. data model and test scenarios;
2. software MVP with event ingestion, map, event card, verification workflow, audit log, export and role-based access.

M3+ depends on actual interrogator data format, equipment documentation, surveyed fiber route and independent field measurements.

## 2. Core rules

1. Do not claim field, diagnostic or geotechnical capability that has not been verified on real data.
2. Do not treat DAS anomaly detection as a diagnosis of seepage, instability, deformation or failure.
3. Keep `TEST`, `REPLAY` and `LIVE` strictly separated.
4. TEST/REPLAY data must never change the operational LIVE state or trigger field alerts.
5. Missing or poor-quality data must not be interpreted as `normal` or low risk.
6. Do not invent thresholds, coefficients, equipment parameters, units, detection ranges or alarm criteria.
7. Do not add direct equipment-control commands. The MVP is monitoring/verification only.
8. Preserve provenance for every imported record, event, processing result and operator decision.
9. The same input data + same code/configuration must produce the same reproducible result.
10. Do not modify or overwrite source/raw records; corrections must be versioned and auditable.

## 3. Canonical terminology

Use the following internal meanings consistently:

- `data_mode`: `TEST | REPLAY | LIVE`
- `record_origin`: e.g. `synthetic | field | calibration`
- `observed_at`: time of physical observation/event; canonical meaning = `phenomenon_time`
- `received_at`: time accepted by the server; canonical ingestion meaning = `ingested_at`
- `result_time`: separate time when a measurement/processing result is produced later than the physical observation
- `replayed_at`: time of replay execution, only for REPLAY
- `data_quality`: `unknown | good | degraded | insufficient`
- `verification_state`: workflow state of event verification
- `alert_state`: operational alert level; do not use this field as a substitute for formal Risk Engine `Hazard` or `RiskAssessment`

Recommended alert workflow:
`unknown -> normal -> observation -> warning -> alarm`

Verification workflow:
`new -> acknowledged -> verification_requested -> in_verification -> confirmed/dismissed -> closed`

`confirmed` means the registered event was confirmed by evidence. It does **not** by itself confirm a geotechnical mechanism.

## 4. Data and provenance

Every record/event should retain, where applicable:

- source identifier;
- route and segment identifiers;
- input record identifier;
- data mode and origin;
- observation and receipt times;
- source clock synchronization status;
- measured quantity, original unit and normalized `unit_code` (UCUM for quantitative data);
- raw-file or raw-fragment reference;
- checksum;
- adapter version;
- processing/algorithm version;
- parameter/configuration version;
- calibration identifier;
- mapping/georeferencing version;
- data-quality state and reasons;
- actor, action, time and reason for every manual decision.

For quantitative processing, units are mandatory. If units or calibration are unknown, retain the data but mark it unavailable for quantitative interpretation.

## 5. API and ingestion behavior

Minimum API behavior should support:

- route/segment lookup;
- normalized event ingestion;
- normalized observation ingestion;
- event filtering;
- event detail/card;
- event actions and verification;
- audit history;
- JSON/CSV export.

Ingestion must be idempotent.

Recommended idempotency key:
`ingestion_context + source_id + external_event_id`

If the same key arrives with different content, return a conflict and require an explicit correction path with reason/audit trail.

Server assigns `received_at` and the internal platform event ID.

## 6. Geometry and fiber mapping

Do not infer geographic coordinates from optical distance unless a surveyed mapping exists.

Keep separate versions for:
- fiber/optical mapping;
- geographic geometry;
- route passport.

If mapping is not verified, show optical coordinate and explicit status: **"georeferencing not confirmed"**.

## 7. Signal processing and ML

For the initial detector, make all processing parameters explicit and versioned:

- frequency band;
- window size;
- features;
- spatial aggregation of neighboring channels;
- threshold;
- anomaly duration;
- object-specific configuration.

Do not transfer TEST thresholds automatically into LIVE.

ML classification is allowed only after there is a labeled object-specific dataset and an independent validation set.

Do not expose a probability-like `confidence` unless its meaning and calibration are documented. Otherwise keep `confidence = null` and, if useful, store a separate diagnostic score with a defined scale.

`dv/v`, low-frequency strain-rate and hydrogeological interpretation are research functions and are not acceptance criteria for the base software MVP.

## 8. Security and audit

- Enforce authentication and RBAC on the server.
- Roles: observer/operator/engineer/administrator/integration service as required.
- Do not trust client-side role checks as authorization.
- Keep the audit log append-only from normal application workflows.
- Do not store secrets, tokens, API keys or certificates in source code, client bundles or ordinary application records; use server-side secret storage and `secret_ref`.
- Use protected transport for remote access.

## 9. Acceptance for M1–M2

The MVP is ready for software acceptance when all of the following are demonstrated on controlled TEST/REPLAY datasets:

1. deterministic import and replay;
2. correct separation of TEST/REPLAY/LIVE;
3. correct route/segment opening from an event;
4. event card contains source, time, segment, data quality, verification state, provenance and links to materials;
5. duplicate delivery does not create duplicate events;
6. conflicting duplicate content is rejected with an explainable error;
7. stale/late records do not automatically become the current operational state;
8. insufficient data is visibly distinct from normal state;
9. operator actions and engineering decisions are auditable;
10. export preserves IDs, provenance and data mode;
11. restart/recovery preserves relationships and audit history;
12. the same input + same versioned configuration yields the same result.

Passing these tests proves software behavior only. It does **not** prove field detection accuracy or structural-safety diagnostics.

## 10. Out of scope for M1–M2

Do not implement as production claims:

- guaranteed seepage detection;
- guaranteed erosion detection;
- quantitative ground displacement from DAS without verified calibration;
- prediction of dam failure;
- automated structural-safety conclusion;
- automatic emergency actuation;
- unverified numerical alarm thresholds;
- manufacturer-specific assumptions without current documentation.

## 11. Change discipline

Before a material change:
1. identify the requirement or issue;
2. record the reason;
3. update schema/API/config version if behavior changes;
4. add or update an acceptance test;
5. preserve backward traceability to the previous behavior.

If a requirement depends on equipment/vendor/field data that are not available, mark it as an open external dependency instead of inventing a value.
