# Retention And Streak Health

## Decisions

- Are onboarding entrants coming back?
- Are activated readers building habit strength?
- Are streaks growing, breaking, or being saved by freezes?

## Approved insights

- Executive D1 retention
- Executive D7+ retention
- Executive D14+ retention
- Executive D30+ retention
- Streak started
- Streak continued
- Streak lost
- Streak freeze prompt shown -> streak freeze used

## Exact query logic

- Preserve current Executive retention semantics exactly:
  - first onboarding `app_opened`
  - `distinct_id`
  - any-event return
  - D1 exact-day, D7+/D14+/D30+ on-or-after
- Always label retention as exact-day or on-or-after

## Known limitations

- Executive retention is onboarding retention, not canonical reader retention
- Internal/test traffic cannot yet be reliably excluded
