# PostHog Rules

## Identity

- `user_analytics_id` is the canonical post-commit user identifier.
- `distinct_id` is only for truly pre-commit anonymous analysis.
- Do not treat PostHog person profiles or identified users as Tether's canonical user model.
- SQL/HogQL can bridge anonymous entry on `distinct_id` into post-commit steps on `user_analytics_id`, but the stitch must be explicit.

## Anonymous-to-canonical limits

- `app_opened` may be anonymous when `is_in_onboarding = true`.
- `welcome_continue_pressed` and `welcome_login_pressed` are intentionally pre-commit.
- `onboarding_intro_completed` and later onboarding steps are post-commit and can use `user_analytics_id`.
- Do not silently mix `distinct_id` and `user_analytics_id` in one metric without saying where the identity handoff happens.

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
  - `returned-on-or-after`
- Current Executive retention semantics:
  - `D1` = exact-day
  - `D7+` = returned on or after day 7
  - `D14+` = returned on or after day 14
  - `D30+` = returned on or after day 30

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
