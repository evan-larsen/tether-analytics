# `streak_lost` Table

`streak_lost` is the materialized PostHog table for derived reading-streak losses. Use it as the source of truth for streak-loss dashboards and follow-up behavior analysis.

## Refresh and coverage

- Refreshes hourly.
- Recomputes from the prior 90 days of source events.
- Excludes the current Utah/Mountain calendar day and the two preceding Utah/Mountain calendar days.

## Columns

| Column | Meaning |
| --- | --- |
| `streak_lost_at` | Utah/Mountain calendar date assigned to the streak-loss incident. |
| `user_analytics_id` | Canonical user key for joins to later product events. |
| `last_streak_completion_date` | The completed-reading date that began the freeze/loss countdown. |
| `lost_streak_days` | Length of the streak that was lost. |
| `user_state_before_loss` | Full state snapshot from the last event before the loss date, stored as JSON text. |

## Definition

For each `user_analytics_id`, the table uses completed-reading local dates. A completion with `freeze_count = n` permits the next `n` missed days. If the next completion is after the resulting loss date, or no later completion exists, it creates one loss row.

Example: a completion on D1 with `freeze_count = 3` permits missed days D2-D4. A completion on D5 preserves the streak; no completion on D5 creates a D5 loss row.

## Dashboard use

- Group `streak_lost_at` by day for streak-loss volume.
- Count distinct `user_analytics_id` for users who lost a streak.
- Break down or filter `user_state_before_loss` fields to study traits at loss.
- Compare `last_streak_completion_date` and `streak_lost_at` to understand the missed-day window before a loss.
- Join later events on `user_analytics_id` and require their timestamp/date to be after `streak_lost_at` to analyze return, renewed reading, or a rebuilt streak.

Do not use the client-captured `streak_lost` event as the source for these questions: it fires when the app is opened or resumed and is not a complete loss record.
