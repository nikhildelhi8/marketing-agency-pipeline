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

## Schema Pattern

Pattern: Constellation Schema
Reason: Two fact tables (fact_creator_metrics + fact_campaign_matches)
sharing three dimension tables (dim_creators, dim_brands, dim_dates).
Each fact table forms its own star. This is the definition of a
constellation schema.

## Diagram

dbdiagram.io URL : https://dbdiagram.io/d/6a3633009340ecc065da0e59

## Tables in Schema

Fact Tables:
fact_creator_metrics -- Grain : one row per creator per ingestion date
-- SCD2 join : always filter dim_creators WHERE is_active = TRUE

    fact_campaign_matches -- Grain: one row per brand-creator pairing

Dimension Tables :
dim_creators -- SCD Type 2 | Partition : valid_from | Cluster: creator_tier , country
dim_brands -- SCD Type 1 | Populated by dbt seed in Phase 4
dim_dates -- Statis calendar dimension | Populated by dbt seed in Phase 4

## Outside the Schema ( 1 table)

    quality_checks_log   -- Operation audit table
                         -- NO FK relationships (must write even with pipeline fails )

                         -- Partition: run_date | Cluster: pipeline_name , status
