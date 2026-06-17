## Business Questions this Schema Must Answer

1. How many subscribers did creator X have 6 months ago ?
2. Which creators grew fastest in the micro tier this quarter ?
3. Which brand-creator matches resulted in accepted outreach?
4. Was the pipeline healthy yesterday?

We need the business question because this will act as the primary source for designing the schema , we are listing down the problems which marketing agency faces daily.

## Data Classification

Rule used : "Do I compute something with this column (SUM , AVG , COUNT)

Yes --> Fact. No , it describes something --> Dimension "

| Column              | Classification   | Table                 |
| ------------------- | ---------------- | --------------------- |
| subscriber_count    | Fact             | fact_creator_metrics  |
| total_views         | Fact             | fact_creator_metrics  |
| video_count         | Fact             | fact_creator_metrics  |
| avg_views_per_video | Fact (derived)   | fact_creator_metrics  |
| subscriber_delta    | Fact (derived)   | fact_creator_metrics  |
| match_score         | Fact             | fact_campaign_matches |
| creator_name        | Dimension        | dim_creators          |
| creator_tier        | Dimension (SCD2) | dim_creators          |
| country             | Dimension        | dim_creators          |
| niche               | Dimension        | dim_creators          |
| brand_name          | Dimension        | dim_brands            |
| industry_vertical   | Dimension        | dim_brands            |
| outreach_status     | Attribute        | fact_campaign_matches |

Classification Rule for each row :

1. subscriber_count , total_views , video_count -- goes into the fact table because the only thing we do with it analytically is aggregate it — sum it, average it, track its growth. Anything we aggregate is a measurement, and measurements belong in fact tables.

2. avg_views_per_video -- is a derived column which is calculated using (total_views/video_count).

3. subscriber_delta -- is a derived column which calculates change in subsriber count since last measurement.

4. match_score -- how a brand and creator matches it is generated as score from 0.0 to 1.0

5. creator_name , country , niche -- is a dimension describing creators dimension.

6. brand_name , industry_vertical -- is a dimension describing the brand name and specific market in which it deals with.

7. outreach_status -- Its storing the current status of the reach ( pending , accepted , completed)

8. creator_tier describes the creator
   it changes over time as creators grow. It belongs in dim_creators but
   needs SCD2 because BQ2 asks about creators in the micro tier historically —
   if we overwrote tier on change, we could never answer that question.

## SCD Decisions

dim_creators: SCD Type 2
Reason: Creator tier changes over time (nano → micro → macro → mega).
BQ2 requires knowing which creators were micro-tier historically.
SCD1 would permanently destroy the ability to answer cohort questions.

Tracking columns: valid_from (DATE), valid_to (DATE), is_active (BOOL)
Surrogate key: creator_sk = hash(channel_id) — one SK per creator entity,
same value across all versions of the same creator.

Current record filter: WHERE is_active = TRUE
Historical record filter: valid_from <= target_date
AND (valid_to > target_date OR valid_to IS NULL)

dim_brands: SCD Type 1
Reason: Brand attributes rarely change and historical brand state is not
required for any V1 business question. Overwrite on change is acceptable.
