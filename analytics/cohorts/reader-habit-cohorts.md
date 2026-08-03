# Reader Habit Cohorts

This is a family of exclusive reader-habit tiers. Each user appears in at most one tier for a given Utah-time `snapshot_day`.

| Reader group | Virtual cohort view | Qualification for snapshot day `D` |
| --- | --- | --- | --- |
| Habit-forming readers | `cohort_habit_forming_users` | At least 3 distinct days in `[D - 7 days, D)`, but no higher tier |
| Active readers | `cohort_habit_active_readers` | At least 7 distinct days in `[D - 14 days, D)`, but not Regular or Near-daily |
| Regular readers | `cohort_habit_regular_readers` | 18–24 distinct days in `[D - 30 days, D)` |
| Near-daily readers | `cohort_habit_near_daily_readers` | At least 25 distinct days in `[D - 30 days, D)` |

## Source and wrappers

- `cohort_reader_habit_tier_snapshots` is the only materialized view in this family. It refreshes hourly and holds each user's active-day counts for prior 7, 14, and 30 complete days plus their assigned `habit_tier`.
- The four named cohort views are virtual wrappers over that table. They filter `habit_tier`, making each group simple to query without duplicating event scans or materialized storage.
- The source scans 120 days of events: a rolling 90-day snapshot history plus the longest 30-day lookback.
- To avoid a global cross join between all session days and all snapshot days, the source expands each distinct user session day only into the 30 later snapshot days that it can affect, then aggregates the 7-, 14-, and 30-day counts.
- Tier priority is Near-daily, then Regular, then Active, then Habit-forming. This highest-qualifying-tier assignment is the canonical segmentation rule.

## Shared behavior

- The active snapshot day is excluded, so every membership decision relies only on complete Utah-time days.
- Each cohort view has one row per `snapshot_day` and `user_analytics_id`; it also exposes the source active-day counts for all three windows.
- The materialized source retains 90 snapshot days. The virtual wrappers always reflect its latest refresh.
- PostHog supports a cadence rather than an exact scheduled time, so hourly refresh keeps completed-day snapshots fresh after midnight.
- These are rolling behavioral states, not permanent labels. A user can enter, leave, and later re-enter a tier, but cannot belong to multiple tiers on the same snapshot day.
- The SQL stored in each PostHog saved query is the executable source of truth.

## Querying

Use the snapshot date when counting or joining membership:

```sql
SELECT
    toDate(snapshot_day) AS cohort_day,
    countDistinct(user_analytics_id) AS readers
FROM cohort_habit_regular_readers
GROUP BY cohort_day
ORDER BY cohort_day
```
