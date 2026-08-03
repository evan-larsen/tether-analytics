# PostHog Rules

## Identity

- `user_analytics_id` is the canonical post-commit user identifier. Always use it for user-level metrics and downstream user behavior.
- `distinct_id` is the anonymous/install identifier. Use it for device- or install-level metrics, such as app build and OTA rollout; do not use it as the canonical user identifier.
- A dedicated, stable device/install ID should replace `distinct_id` for device metrics if and when one is captured. Until then, `distinct_id` is the approved device/install proxy.
- Do not treat PostHog person profiles or identified users as Tether's canonical user model.
- SQL/HogQL can bridge anonymous entry on `distinct_id` into post-commit steps on `user_analytics_id`, but the stitch must be explicit.

## Anonymous-to-canonical limits

- `app_opened` may be anonymous when `is_in_onboarding = true`.
- `welcome_continue_pressed` and `welcome_login_pressed` are intentionally pre-commit.
- `onboarding_intro_completed` and later onboarding steps are post-commit and can use `user_analytics_id`.
- Do not silently mix `distinct_id` and `user_analytics_id` in one metric without saying where the identity handoff happens.
- For stitched onboarding metrics, define the handoff timestamp explicitly:
  - regular new-user flow: first event after onboarding entry with a non-empty `user_analytics_id`
  - login-then-auto-create flow: `account_auth_completed` with `source = 'onboarding'` and `needs_profile_setup = true`
- After the handoff timestamp, retention and downstream lifecycle logic should prefer `user_analytics_id` over the original anonymous `distinct_id`.

## Minimum trustworthy version

- Canonical user-level metrics are trustworthy on app version `1.6.0+`.
- Default canonical filter:
  - `properties['$app_version'] >= '1.6.0'`
- Pre-`1.6.0` data can still be useful for anonymous onboarding analysis, but not for canonical user metrics.

## Session deduplication

- Reading session events must be deduplicated by `properties.session_id` whenever the question is about sessions rather than raw event count.
- This especially applies to:
  - `reading_session_started`
  - `reading_session_completed`
  - `reading_session_exited`

## Meaningful active users

- Tether's meaningful active user is a user who completes a reading session.
- Default active-user event:
  - `reading_session_completed`
- Avoid using generic `app_opened` as the primary habit-health metric.

## Retention families

- Tether has two valid retention families.
- `Executive onboarding retention`
  - anchor: first `app_opened` where `is_in_onboarding = true`
  - identifier: `distinct_id`
  - return: any later event
- `Canonical reader retention`
  - anchor: first `reading_session_completed`
  - identifier: `user_analytics_id`
  - return: later `reading_session_completed`
- These families answer different questions and must not be merged.

## Retention labeling

- Every retention metric must say whether it is:
  - `exact-day`
  - `fixed observation window`
- Current Executive retention semantics:
  - `D1` = exact-day
  - `D7–13` = at least one event in days 7 through 13
  - `D14–29` = at least one event in days 14 through 29
  - `D30–59` = at least one event in days 30 through 59
- Only include cohorts whose entire return window has completed. This gives every included new user the same opportunity to return.

## Internal/test-user limitations

- Internal/test traffic cannot yet be reliably excluded in PostHog.
- The old PostHog-native test-user cohort does not match Tether's identity model.
- Until a Tether-native exclusion property exists on events, assume dashboards may contain some internal/test traffic.

## Query bias

- Prefer HogQL/SQL for any query that must:
  - use `user_analytics_id`
  - dedupe by `session_id`
  - preserve current Executive retention semantics
  - stitch anonymous onboarding into post-commit lifecycle analysis

## Materialized cohort snapshots

- Use a materialized SQL view when cohort membership requires custom logic or historical daily membership; PostHog-native cohorts represent current membership and are not the source of truth for these historical snapshots.
- Model historical cohorts at one row per user and `snapshot_day`. Define the lookback relative to the snapshot day and exclude the in-progress Utah-time day unless the cohort explicitly needs intraday behavior.
- Treat rolling behavioral cohorts as daily states, not permanent labels: users can enter, exit, and re-enter them.
- Retain the latest 90 snapshot days by default. Use a different history horizon only when the analytical requirement is explicit.
- For completed-day cohorts used in daily reporting, use the hourly (`1hour`) materialization cadence. PostHog supports cadence, not an exact scheduled run time.
