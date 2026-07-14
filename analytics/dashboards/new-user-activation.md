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

- New users stay anchored on first onboarding `app_opened` by `distinct_id`
- Completed onboarding and activated users should hold post-commit steps constant on `user_analytics_id`
- Use SQL to bridge `distinct_id` entry into `user_analytics_id` once proto identity exists

## Known limitations

- Top-of-funnel entry is still partially anonymous
- Internal/test traffic cannot yet be reliably excluded
