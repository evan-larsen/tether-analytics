# New User Cohort

## Object

- PostHog saved query name: `cohort_new_users`
- Type: materialized SQL view
- Refresh cadence: hourly (`1hour`)
- Project: `Tether Production`

## Materialization schedule

- Keep this complete-day cohort on an hourly cadence so yesterday's entrants appear soon after the project day rolls over and remain fresh throughout the day.
- PostHog does not support choosing an exact materialization time (for example, 12:05 AM Utah time); it schedules runs by cadence only.
- Available materialization cadences are `15min`, `30min`, `1hour`, `6hour`, `12hour`, `24hour`, `7day`, `30day`, and `never` (pause scheduled materialization).
- Use `1hour` as the normal cadence for cohorts that feed daily metrics. Use a faster cadence only when the freshness benefit justifies the additional query and storage work.

## Columns

- `distinct_id`
- `joined_at`

## Window

- Only keep members whose `joined_at` is in the last 90 complete days
- Exclude the current in-progress day
- Downstream day-based queries should use `toDate(joined_at)` when needed

## Purpose

- Use this as the default source for `new_users`
- Reuse one stable SQL-first cohort across retention and onboarding analyses
- If you need the explicit definition, look at the SQL in the `cohort_new_users` view in PostHog
- Keep the cohort keyed only on `distinct_id`

## Example

```sql
SELECT
    toDate(joined_at) AS joined_day,
    count() AS new_users
FROM cohort_new_users
GROUP BY joined_day
ORDER BY joined_day
```
