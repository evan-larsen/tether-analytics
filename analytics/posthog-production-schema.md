# Tether Production PostHog Baseline

Last refreshed: 2026-07-12
Source: live PostHog MCP against `Tether Production` project `471847`

## Project

- Name: `Tether Production`
- PostHog project id: `471847`
- Organization: `Tether`
- Timezone: `US/Mountain`
- Person-on-events querying: enabled
- Notes:
  - Production has ingested events.
  - Session recording is currently disabled.
  - A PostHog-native test-account filter exists, but [posthog-rules.md](C:/Users/evanl/Documents/tether-analytics/analytics/posthog-rules.md:1) still treats internal/test exclusion as not yet trustworthy for shared analysis.

## Event Families

### Core product events

- `app_opened`
- `welcome_continue_pressed`
- `welcome_login_pressed`
- `onboarding_intro_completed`
- `onboarding_start_selection_completed`
- `reading_chapter_intro_completed`
- `reading_chunk_completed`
- `reading_session_started`
- `reading_session_completed`
- `reading_session_exited`
- `verse_save_toggled`
- `current_chapter_changed`

### Account/auth events

- `account_entry_opened`
- `account_method_selected`
- `account_setup_viewed`
- `account_email_submitted`
- `account_auth_completed`
- `account_setup_completed`
- `account_code_resent`
- `account_signed_out`

### Streak / lifecycle events

- `streak_lost`
- `streak_lost_prompt_shown`
- `streak_freeze_prompt_shown`
- `streak_freeze_used`

### Notifications

- `notification_permission_prompt_shown`
- `notification_permission_result`

### Supporter paywall and subscriptions

- Client paywall events: `supporter_prompt_viewed`, `paywall_viewed`, `paywall_offer_loaded`, `paywall_offer_load_failed`, `paywall_cta_pressed`, `paywall_purchase_started`, `paywall_purchase_cancelled`, `paywall_restore_pressed`, `paywall_restore_completed`, `paywall_closed`
- Server-authoritative RevenueCat lifecycle events expected from the Supabase webhook: `subscription_initial_purchase`, `subscription_renewed`, `subscription_cancelled`, `subscription_uncancelled`, `subscription_transfer`, `subscription_paused`, `subscription_expired`
- See [supporter-subscription-events.md](C:/Users/evanl/Documents/tether-analytics/analytics/supporter-subscription-events.md) for the event contract, delivery prerequisites, and interpretation rules.

### Autocaptured / system events also present

- Mobile autocapture such as `Application opened`, `Application became active`, `Application backgrounded`, `Application installed`, `Application updated`, `Deep link opened`
- Standard PostHog/system events such as `$identify`, `$screen`, `$pageview`, `$exception`

## Shared Property Conventions

- Canonical user identifier: `user_analytics_id`
- Anonymous/pre-commit identifier: `distinct_id`
- App version appears as both `app_version` and `$app_version`
- Common shared dimensions across many events:
  - `platform`
  - `theme_mode`
  - `account_state_at_event`
  - `days_since_first_open`
  - `streak_count_at_event`
  - `$device_type`
  - `$app_build`
  - geo properties such as `$geoip_country_code`

## Key Event Schemas

### `app_opened`

- Identity/context:
  - `user_analytics_id`
  - `account_state_at_event`
  - `is_in_onboarding`
  - `platform`
  - `theme_mode`
  - `app_version`
- Counters/state:
  - `days_since_first_open`
  - `current_streak`
  - `current_level`
  - `total_reading_starts`
  - `total_sessions_completed`
  - `total_distinct_reading_days`
  - `total_saved_verses`
  - `total_xp`
- Observed values:
  - `platform`: `ios`, `android`
  - `account_state_at_event`: `anonymous`, `signed_in`
  - `is_in_onboarding`: `true`, `false`

### `reading_session_started`

- Identifiers:
  - `session_id`
  - `reading_run_id`
  - `user_analytics_id`
- Reading context:
  - `book_id`
  - `chapter_id`
  - `chapter_number`
  - `session_source`
  - `mode`
- Session counters:
  - `session_index`
  - `reading_run_session_index`
  - `chapter_session_count`
  - `session_verse_count`
  - `session_chunk_count`
  - `session_character_count`
- State snapshots:
  - `current_streak`
  - `streak_days_before`
  - `current_level`
  - `total_reading_starts`
  - `total_sessions_completed`
  - `total_distinct_reading_days`
  - `total_saved_verses`
  - `total_xp`
- Booleans:
  - `is_first_read_today`
  - `is_streak_complete_today`
  - `has_freeze`

### `reading_session_completed`

- Identifiers:
  - `session_id`
  - `reading_run_id`
  - `user_analytics_id`
- Reading context:
  - `book_id`
  - `chapter_id`
  - `chapter_number`
  - `session_source`
  - `mode`
- Completion/output metrics:
  - `session_elapsed_seconds`
  - `session_verse_count`
  - `session_chunk_count`
  - `session_character_count`
  - `accuracy_percent`
  - `correct_question_count`
  - `total_question_count`
  - `xp_earned`
  - `xp_multiplier`
  - `levels_gained`
  - `verses_saved_this_session`
  - `verses_unsaved_this_session`
- State-change booleans:
  - `did_complete_chapter`
  - `did_level_up`
  - `did_streak_increment`
  - `saved_any_verse_this_session`
  - `unsaved_any_verse_this_session`
- State snapshots:
  - `streak_days_after`
  - `level_before`
  - `level_after`
  - `current_level`
  - `current_streak`
  - `total_sessions_completed`
  - `total_distinct_reading_days`
  - `total_saved_verses`
  - `total_xp`
- Observed values:
  - `session_source`: `path`, `onboarding`, `widget`, `notification`, `streak_lifecycle_prompt`
  - `mode`: `active`

### `onboarding_intro_completed`

- Identity/context:
  - `user_analytics_id`
  - `platform`
  - `theme_mode`
  - `account_state_at_event`
  - `app_version`
- State snapshots:
  - `days_since_first_open`
  - `streak_count_at_event`

### `account_auth_completed`

- Identity/context:
  - `user_analytics_id`
  - `platform`
  - `theme_mode`
  - `account_state_at_event`
  - `app_version`
- Auth-specific properties:
  - `method`
  - `source`
  - `needs_profile_setup`
- Observed values:
  - `method`: `google`, `email`, `apple`
  - `source`: `onboarding`, `profile`
  - `needs_profile_setup`: `true`, `false`

## Analysis Guardrails

- Follow [posthog-rules.md](C:/Users/evanl/Documents/tether-analytics/analytics/posthog-rules.md:1) for identity handoff, canonical-user rules, session deduplication, and retention semantics.
- Treat `reading_session_completed` as the meaningful active-user anchor, not generic opens.
- Deduplicate session metrics on `session_id`.
- Default canonical user analysis to app version `1.6.0+`.
- Do not assume PostHog person properties are Tether's canonical user model.

## Good Next Pulls

- Add a curated event dictionary only for the events that appear in shared dashboards.
- Validate the first `subscription_initial_purchase` delivery from RevenueCat before publishing a Supporter conversion metric.
- Validate property presence for `streak_lost`, `streak_freeze_used`, and `notification_permission_result` before building deeper lifecycle dashboards.
- Capture approved HogQL snippets for identity stitching and session deduplication alongside dashboard specs.
