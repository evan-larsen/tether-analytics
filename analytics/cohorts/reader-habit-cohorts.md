# Reader Habit Cohorts

This is a family of materialized SQL views that record historical daily membership snapshots for reading habits. They all use `reading_session_started`, `user_analytics_id`, Utah-time calendar days, and the same rolling 90-day snapshot history.

| Reader group | PostHog saved query | Qualification for snapshot day `D` | Event scan |
| --- | --- | --- | --- |
| Habit-forming readers | `cohort_habit_forming_users` | At least 3 distinct days in `[D - 7 days, D)` | 96 days |
| Active readers | `cohort_habit_active_readers` | At least 7 distinct days in `[D - 14 days, D)` | 104 days |
| Regular readers | `cohort_habit_regular_readers` | At least 15 distinct days in `[D - 30 days, D)` | 120 days |
| Near-daily readers | `cohort_habit_near_daily_readers` | At least 25 distinct days in `[D - 30 days, D)` | 120 days |

## Shared behavior

- The active snapshot day is excluded, so every membership decision relies only on complete Utah-time days.
- Each view has one row per `snapshot_day` and `user_analytics_id`; it also exposes the number of qualifying active days for that window.
- Each view retains 90 snapshot days. Its event scan includes the retained history plus the required lookback window; the 30-day views conservatively scan 120 days.
- All views refresh hourly (`1hour`). PostHog supports a cadence rather than an exact scheduled time, so hourly refresh keeps completed-day snapshots fresh after midnight.
- These are rolling behavioral states, not permanent labels. A user can enter, leave, and later re-enter any group. The groups can overlap and are not guaranteed to be strictly nested.
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
