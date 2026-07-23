# Metric Registry

Only metrics that are currently used in an existing dashboard or in the current approved dashboard batch belong here.

| Name | Definition | Event / property logic | Identifier | Important caveat |
| --- | --- | --- | --- | --- |
| New users | Net-new onboarding entrants | Pull from `cohort_new_users` | `distinct_id` | Materialized rolling 90 complete-day cohort recomputed daily; check the cohort view itself for the exact SQL definition |
| Completed onboarding | Users who finish start selection | `onboarding_start_selection_completed` | `user_analytics_id` | If the funnel includes first open, bridge from `distinct_id` in SQL |
| Activated users | Post-commit onboarding users who complete a first reading session | `onboarding_intro_completed` -> first `reading_session_completed`, usually with `session_source = 'onboarding'` | `user_analytics_id` | If anchored on first open, the funnel is stitched, not purely canonical |
| Started reading sessions | Unique reading sessions that start | `count(distinct properties.session_id)` on `reading_session_started` | `session_id` | Deduplicate raw events |
| Completed reading sessions | Unique reading sessions that complete | `count(distinct properties.session_id)` on `reading_session_completed` | `session_id` | Deduplicate raw events |
| Active users | Users who complete at least one reading session in the window | Distinct `user_analytics_id` on `reading_session_completed` | `user_analytics_id` | Canonical metric requires app version `1.6.0+` |
| DAU | Daily active users | Distinct `user_analytics_id` on `reading_session_completed` per day | `user_analytics_id` | Meaningful activity, not generic app opens |
| WAU | Weekly active users | Distinct `user_analytics_id` on `reading_session_completed` in trailing 7 days | `user_analytics_id` | Meaningful activity, not generic app opens |
| MAU | Monthly active users | Distinct `user_analytics_id` on `reading_session_completed` in trailing 30 days | `user_analytics_id` | Meaningful activity, not generic app opens |
| Session completion rate | Share of started sessions that complete | Deduped completed `session_id` / deduped started `session_id` | `session_id` | Must compare deduped sessions, not raw event totals |
| Verses read | Total verses completed in sessions | Sum `session_verse_count` on `reading_session_completed` | `session_id` | Session-level output metric, not user count |
| Executive D1 retention | New onboarding users who return on exact day 1 | First onboarding `app_opened`; return uses `distinct_id` before explicit new-user confirmation and stitched `user_analytics_id` after confirmation | `distinct_id` entry, then `user_analytics_id` after handoff | Current live query uses a two-flow stitched definition; D7+/D14+/D30+ still need the same upgrade |
| Executive D7+ retention | Onboarding entrants who return on or after day 7 | First onboarding `app_opened`; any later event on or after day 7 | `distinct_id` | Preserve current Executive semantics exactly |
| Executive D14+ retention | Onboarding entrants who return on or after day 14 | First onboarding `app_opened`; any later event on or after day 14 | `distinct_id` | Preserve current Executive semantics exactly |
| Executive D30+ retention | Onboarding entrants who return on or after day 30 | First onboarding `app_opened`; any later event on or after day 30 | `distinct_id` | Preserve current Executive semantics exactly |
| Current streak | Latest observed streak for a recent user | Latest `current_streak`, fallback `streak_count_at_event` | `user_analytics_id` | Snapshot metric; only as current as the latest emitted event |
| Streak started | Users whose streak moved from 0 to 1 | `reading_session_completed` where `did_streak_increment = true` and `streak_days_after = 1` | `user_analytics_id` | Canonical metric requires app version `1.6.0+` |
| Streak continued | Users whose existing streak extended | `reading_session_completed` where `did_streak_increment = true` and `streak_days_after > 1` | `user_analytics_id` | Canonical metric requires app version `1.6.0+` |
| Streak lost | Users whose streak broke | `streak_lost` | `user_analytics_id` | Measures emitted streak-loss detection, not an independent state table |
| Streak freeze conversion | Users shown a freeze prompt who use a freeze | `streak_freeze_prompt_shown` -> `streak_freeze_used` | `user_analytics_id` | Query carefully by user and time window, not raw event count alone |
