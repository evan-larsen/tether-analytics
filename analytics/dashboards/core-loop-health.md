# Core Loop Health

## Decisions

- Is the reading loop healthy enough to support daily habit formation?
- Is session completion improving or getting weaker?
- Where are sessions breaking by source, platform, or version?

## Approved insights

- Completion DAU
- Completion WAU
- Completion MAU
- Started reading sessions
- Completed reading sessions
- Session completion rate
- Verses read
- Exit-stage distribution
- Breakdown by `platform`, `$app_version`, `session_source`

## Exact query logic

- Deduplicate reading sessions by `properties.session_id`
- Canonical user metrics require `user_analytics_id` and app version `1.6.0+`
- Completion DAU, Completion WAU, and Completion MAU are based on `reading_session_completed`; these are distinct from full DAU, WAU, and MAU, which use all events

## Known limitations

- Historical duplicates make raw `reading_session_completed` event totals less trustworthy than deduped session counts
- Internal/test traffic cannot yet be reliably excluded
