# New User Activation

## Decisions

- How many onboarding entrants are we getting?
- How many users complete onboarding?
- How many become activated readers?

## Approved insights

- New users by day
- Completed onboarding
- Activated users
- Activation within 1 day
- Activation within 7 days
- Start-selection breakdown by `book_id` and `chapter_number`
- Anonymous-to-canonical handoff coverage

## Exact query logic

- New users should come from `cohort_new_users`
- `cohort_new_users` is a materialized rolling 90 complete-day cohort, recomputed daily
- If the exact definition ever changes, treat the SQL in `cohort_new_users` as the source of truth
- Completed onboarding and activated users should hold post-commit steps constant on `user_analytics_id`
- Use SQL to bridge `distinct_id` entry into `user_analytics_id` once proto identity exists

## Known limitations

- Top-of-funnel entry is still partially anonymous
- Internal/test traffic cannot yet be reliably excluded
