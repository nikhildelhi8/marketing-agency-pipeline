# Schema V1 — Marketing Agency Pipeline

## Grain Definitions

### fact_creator_metrics

Grain: One row per YouTube creator per ingestion date.

Natural grain keys (for design docs and explanation):
(channel_id, ingestion_date)
Use these when explaining the uniqueness constraint to a colleague or
in design documents. These are real-world identifiers from the source.

Physical enforcement (in dbt, Phase 4):
(creator_sk, date_id)
After dbt transforms the raw data, the fact table stores surrogate keys.
The dbt unique test runs on these columns.

These two expressions are NOT different constraints.
They describe the same constraint in two different representations.

Silent duplicate scenario:
If the pipeline runs twice on 2026-07-15 for creator UC123456:
→ Two rows with channel_id = UC123456, ingestion_date = 2026-07-15
→ SUM(subscriber_count) returns double the real value
→ BigQuery raises no error. The warehouse accepts it silently.
→ Prevention: dbt unique test on (creator_sk, date_id) in Phase 4

---

### fact_campaign_matches

Grain: One row per brand-creator pairing.

Natural grain keys: (brand_id, channel_id)
Physical enforcement: (brand_id, creator_sk)

Implication:
The same brand cannot be matched to the same creator more than once in V1.
A second insert of the same brand-creator pair will be caught by the
dbt unique test.
