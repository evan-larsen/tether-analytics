# Habit-Forming Users Cohort

## Object

- PostHog saved query name: `cohort_habit_forming_users`
- Type: materialized SQL view with historical daily membership snapshots
- Refresh cadence: hourly (`1hour`)
- Project: `Tether Production`

## Definition

Each row records one user's membership on one `snapshot_day`.

- Identifier: `user_analytics_id`
- Qualifying event: `reading_session_started`
- A user qualifies on snapshot day `D` when they started a reading session on at least three distinct Utah-time calendar days during `[D - 7 days, D)`.
- `D` itself is excluded, so the calculation only uses complete days.
- `active_days_in_prior_7d` records the number of qualifying distinct days and is expected to be at least `3`.

## History and refresh behavior

- The view retains the latest 90 snapshot days. Unless a cohort explicitly needs a different horizon, materialized historical cohorts should use this same rolling 90-day history.
- This is a rolling behavioral state, not a fixed acquisition cohort: a user may qualify, stop qualifying, and later qualify again.
- Historical snapshot rows preserve membership for each day, making the cohort safe for trend analysis, retention joins, and later outcome analysis.
- Refresh hourly rather than daily. PostHog schedules by cadence and cannot run the view at an exact clock time; hourly refresh makes completed-day snapshots available shortly after the Utah-time day boundary.
- The active day is deliberately excluded from both the seven-day lookback and the event scan.

## Use

- Use `snapshot_day` as the cohort date when charting habit formation over time.
- Count distinct `user_analytics_id` for a daily membership total.
- Join on both `user_analytics_id` and the intended snapshot or outcome date; do not treat this as permanent user membership.
- Treat the SQL in PostHog as the source of truth for the executable definition.

## Example

```sql
SELECT
    toDate(snapshot_day) AS cohort_day,
    countDistinct(user_analytics_id) AS habit_forming_users
FROM cohort_habit_forming_users
GROUP BY cohort_day
ORDER BY cohort_day
```
