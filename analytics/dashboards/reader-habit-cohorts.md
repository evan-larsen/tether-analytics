# Reader Habit Cohorts

## Purpose

Track the size, composition, and daily inflow of Tether's exclusive reader-habit tiers.

## Reader-habit tier system

- Habit-forming: at least 3 reading days in the prior 7 complete days
- Establishing: at least 7 reading days in the prior 14 complete days
- Active: 18 to 24 reading days in the prior 30 complete days
- Near-daily: at least 25 reading days in the prior 30 complete days
- A user belongs to only their highest qualifying tier on a given `snapshot_day`.

## Current insights

- `Reader habit tiers - last 14 complete days`: absolute daily membership counts for all four tiers, excluding today.
- `Reader habit tier mix - last 90 snapshot days`: 100% stacked daily tier composition across the available snapshot history.
- `Reader habit tier entries - daily`: users entering each tier. An entry means the user was not in that same tier on the prior snapshot day; it includes first entries, promotions, and re-entries.

## Interpretation

- Use absolute membership counts to understand the size of each tier.
- Use the 100% tier mix to see whether the composition of tiered readers is improving independently of total audience growth.
- Do not interpret daily entries as lifetime-first-time entries; the source retains a rolling 90-day history and the metric intentionally includes re-entry.
- For implementation details and the source table, see [reader-habit-cohorts.md](../cohorts/reader-habit-cohorts.md).
