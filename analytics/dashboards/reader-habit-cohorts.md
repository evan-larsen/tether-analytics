# Reader Habit Cohorts

## Purpose

Track the size, composition, and daily inflow of Tether's exclusive reader-habit tiers.

## Reader-habit tier system

- Habit-forming: at least 3 reading days in the prior 7 complete days
- Establishing: at least 7 reading days in the prior 14 complete days
- Active: 18 to 24 reading days in the prior 30 complete days
- Near-daily: at least 25 reading days in the prior 30 complete days
- A user belongs to only their highest qualifying tier on a given `snapshot_day`.
- Pre-habit is a reusable virtual cohort, rather than a materialized tier. It is the remaining canonical DAU: users with a `reading_session_completed` event on the active day who have no assigned habit tier for that snapshot day.

## Current insights

- `Reader habit tiers - last 14 complete days`: absolute daily membership counts for all four tiers, excluding today.
- `Daily active reader habit mix — count`: stacked daily counts of every canonical DAU across Pre-habit plus the four exclusive reader-habit tiers. It uses completed days only.
- `Daily active reader habit mix — share`: the same daily breakdown normalized to 100%, so composition can be compared independently of DAU volume.
- `Average current DAU habit mix — last 7 complete days`: a single 100% stacked bar showing the average canonical DAU composition over the last 7 complete days, using the same Pre-habit plus four-tier segmentation as the daily mix charts.
- `Reader habit tier entries - daily`: users entering each tier. An entry means the user was not in that same tier on the prior snapshot day; it includes first entries, promotions, and re-entries.

## Tier color palette

When an insight displays counts for the reader-habit tiers without another categorical breakdown, use these fixed series colors. Keep these mappings stable across dashboards rather than relying on PostHog's automatic series order.

| Tier | Color | Hex |
| --- | --- | --- |
| Habit-forming | Blue | `#1D4AFF` |
| Establishing | Teal | `#42827E` |
| Active | Purple | `#621DA6` |
| Near-daily | Pink | `#CE0E74` |
| Pre-habit | Gray | `#A3A3A3` |

## Interpretation

- Use absolute membership counts to understand the size of each tier.
- Use the count view to follow total DAU and the absolute size of each segment. Use the share view to see whether the composition of active readers is improving independently of total audience growth.
- Use the 7-day average share view to summarize the current DAU mix in one snapshot when you want a stable segment breakdown without day-to-day volatility.
- Pre-habit is calculated by the virtual `cohort_habit_pre_habit_readers` view as canonical DAU minus users assigned to a reader-habit tier on that day. It may include brand-new or returning low-frequency readers, so it is not the same as the acquisition `New users` cohort.
- Do not interpret daily entries as lifetime-first-time entries; the source retains a rolling 90-day history and the metric intentionally includes re-entry.
- For implementation details and the source table, see [reader-habit-cohorts.md](../cohorts/reader-habit-cohorts.md).
