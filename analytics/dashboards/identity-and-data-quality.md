# Identity And Data Quality

## Decisions

- Can we trust canonical user metrics?
- Has the `1.6.0` identity rollout stabilized?
- Are session event duplicates still affecting dashboard quality?

## Approved insights

- `user_analytics_id` coverage by event
- `user_analytics_id` coverage by app version
- Signed-in vs anonymous split on major events
- Duplicate started sessions
- Duplicate completed sessions
- Duplicate exited sessions
- Sessions without a terminal event

## Exact query logic

- Canonical identity checks should segment by app version `1.6.0+`
- Session duplication checks should group by `properties.session_id`
- Separate anonymous onboarding behavior from post-commit canonical behavior

## Known limitations

- Internal/test traffic cannot yet be reliably excluded
- Some historical noise will persist until old app versions age out
