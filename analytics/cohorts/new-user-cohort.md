# New User Cohort

## Object

- PostHog saved query name: `cohort_new_users`
- Type: materialized SQL view
- Refresh cadence: daily (`24hour`)
- Project: `Tether Production`

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
