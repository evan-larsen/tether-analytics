# Supporter subscription event contract

This document is the reference for Supporter monetization analytics. It covers both the client paywall flow and the server-authoritative RevenueCat lifecycle events that are forwarded by the Supabase `revenuecat-webhook` Edge Function.

Last reviewed: 2026-07-28

## Two complementary event sources

| Source | What it measures | Treat as authoritative for |
| --- | --- | --- |
| Client app / paywall SDK | Prompt exposure and the user's journey through the paywall | Discovery, paywall UX, purchase intent, and store-flow abandonment |
| RevenueCat webhook -> Supabase -> PostHog | Store subscription lifecycle after RevenueCat delivers its webhook | Successful purchases, renewals, cancellations, pauses, transfers, and expirations |

Do not call a `paywall_purchase_started` event a purchase. The completed subscription metric is `subscription_initial_purchase`, emitted from RevenueCat after a successful store purchase.

## Client paywall events

| Event | Meaning | Useful fields |
| --- | --- | --- |
| `supporter_prompt_viewed` | In-app Supporter prompt was displayed before the full paywall. | `source`, `prompt_display_count`, `user_analytics_id`, `platform`, `app_version` |
| `paywall_viewed` | Full paywall opened. | `paywall_session_id`, `entry_point`, `surface`, `paywall_variant`, `user_analytics_id` |
| `paywall_offer_loaded` | Offers loaded for a paywall session. | `paywall_session_id`, `entry_point`, offer context |
| `paywall_offer_load_failed` | Offer retrieval failed; this is not a payment failure. | `failure_type`, `failure_code`, `failure_message`, `paywall_session_id` |
| `paywall_cta_pressed` | User pressed the paywall purchase CTA. | `paywall_session_id`, offer/product context |
| `paywall_purchase_started` | Store purchase flow opened. | `paywall_session_id`, `product_id`, `package_id`, `offering_id`, `price`, `currency`, `period_unit`, `period_count` |
| `paywall_purchase_cancelled` | User cancelled or backed out of the client/store purchase flow. It is not a declined payment. | Same offer and price fields as `paywall_purchase_started` |
| `paywall_restore_pressed` | User chose Restore Purchases. | `paywall_session_id`, `entry_point` |
| `paywall_restore_completed` | Restore Purchases completed in the client. This does not prove a new purchase. | `paywall_session_id`, `user_analytics_id` |
| `paywall_closed` | Paywall was dismissed. | `paywall_session_id`, `entry_point`, `dismiss_method` |

Observed entry points include `profile_cta`, `settings_cta`, and `post_reading_prompt`. The post-reading prompt has a reading-count eligibility gate; the counter is carried as `user_state.total_reading_starts` on reading events, not as a top-level event property.

## RevenueCat lifecycle events

The webhook maps only the RevenueCat types below. Any other RevenueCat type is deliberately skipped by the forwarding code.

| RevenueCat webhook type | PostHog event | Meaning |
| --- | --- | --- |
| `INITIAL_PURCHASE` | `subscription_initial_purchase` | A new subscription purchase completed. Primary new-conversion event. |
| `RENEWAL` | `subscription_renewed` | A subscription renewed. |
| `CANCELLATION` | `subscription_cancelled` | Auto-renewal was cancelled; entitlement may remain active until expiry. |
| `UNCANCELLATION` | `subscription_uncancelled` | A previously cancelled subscription resumed auto-renewal. |
| `TRANSFER` | `subscription_transfer` | RevenueCat transferred the subscription between app users. |
| `SUBSCRIPTION_PAUSED` | `subscription_paused` | Subscription is paused. |
| `EXPIRATION` | `subscription_expired` | Entitlement expired. |

### Shared RevenueCat properties

- Identity and lineage: `user_analytics_id`, `revenuecat_app_user_id`, `revenuecat_aliases`, `revenuecat_event_id`, `revenuecat_event_type`, `source = revenuecat_webhook`
- Product / entitlement: `app_id`, `product_id`, `new_product_id`, `entitlement_id`, `entitlement_ids`, `period_type`, `presented_offering_id`, `store`, `new_store`
- Transaction: `transaction_id`, `original_transaction_id`, `revenue`, `price_in_purchased_currency`, `currency`, `offer_code`, `is_family_share`
- Timing: `purchased_at`, `expiration_at`, `event_timestamp_ms`, `purchased_at_ms`, `expiration_at_ms`
- Cancellation: `cancel_reason` (only when supplied by RevenueCat)

`revenuecat_event_id` is also sent as `$insert_id`, so RevenueCat webhook retries should deduplicate in PostHog.

## Delivery prerequisites and caveats

The webhook forwards an event only when all of these are true:

1. RevenueCat sends one of the mapped event types.
2. The webhook can retrieve the RevenueCat customer information and match `event.app_user_id` to a `profiles` row.
3. The matched profile has a non-empty `user_analytics_id`.
4. `POSTHOG_PROJECT_API_KEY` is configured in the Supabase Edge Function environment.

Forwarding failures are caught and logged by the function so that profile synchronization can still return successfully. The response labels that case as `disabled`, the same label used when forwarding is not configured; inspect Edge Function logs to distinguish the two.

`POSTHOG_PROJECT_API_KEY` determines the destination project. A successful `forwarded` response only proves that PostHog accepted the capture using that key; it does not prove the event arrived in Tether Production. Verify the secret belongs to project `471847` (`Tether Production`), not project `470659` (`Tether Dev`).

The code currently labels all webhook revenue as `USD`. Confirm this before reporting multi-currency revenue. It also uses `purchased_at` as the PostHog event timestamp whenever RevenueCat supplies it; lifecycle events whose occurrence differs from their original purchase time should be validated before time-series reporting.

## Current production status

On 2026-07-28, RevenueCat webhooks were found to be successfully forwarded but initially landed in Tether Dev because the Supabase `POSTHOG_PROJECT_API_KEY` pointed there. The key was corrected to Tether Production, and the first $4.99 monthly `subscription_initial_purchase` plus its subsequent `subscription_cancelled` event were replayed through the webhook into Production. Verify future live events land in Production before relying on the conversion dashboard.

When the first successful store purchase occurs, verify that `subscription_initial_purchase` appears with `source = revenuecat_webhook`, a non-empty `revenuecat_event_id`, and the matching `user_analytics_id`.
