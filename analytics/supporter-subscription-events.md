# Supporter subscription event contract

This document is the reference for Supporter monetization analytics. It covers both the client paywall flow and the server-authoritative RevenueCat lifecycle events that are forwarded by the Supabase `revenuecat-webhook` Edge Function.

Last reviewed: 2026-08-03

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

The hardened webhook forwards the RevenueCat lifecycle and entitlement events below. `TEST` is deliberately acknowledged but never forwarded. Paywall UI events are not forwarded because Tether already captures its own client paywall event family.

| RevenueCat webhook type | PostHog event | Meaning |
| --- | --- | --- |
| `INITIAL_PURCHASE` | `subscription_initial_purchase` | A new subscription purchase completed. Primary new-conversion event. |
| `RENEWAL` | `subscription_renewed` | A subscription renewed; RevenueCat can also emit this when a lapsed subscriber resubscribes. |
| `CANCELLATION` | `subscription_cancelled` | Auto-renewal or a non-renewing purchase was cancelled/refunded. It is a state event, not a revenue reversal. |
| `UNCANCELLATION` | `subscription_uncancelled` | A previously cancelled subscription resumed auto-renewal. |
| `NON_RENEWING_PURCHASE` | `subscription_non_renewing_purchase` | A one-time, lifetime, or otherwise non-renewing purchase completed. |
| `SUBSCRIPTION_PAUSED` | `subscription_paused` | Subscription is paused. |
| `EXPIRATION` | `subscription_expired` | Entitlement expired. |
| `BILLING_ISSUE` | `subscription_billing_issue` | A renewal charge failed; do not treat this as an expiration. |
| `PRODUCT_CHANGE` | `subscription_product_changed` | Product switch was initiated or applied; it does not necessarily take effect immediately. |
| `SUBSCRIPTION_EXTENDED` | `subscription_extended` | Current subscription expiration was extended. |
| `REFUND_REVERSED` | `subscription_refund_reversed` | An App Store refund was reversed. |
| `INVOICE_ISSUANCE` | `subscription_invoice_issued` | An unpaid RevenueCat Billing invoice was issued. |
| `TRANSFER` | `subscription_transferred` | RevenueCat transferred transactions and entitlements between App User IDs. The event is sent for the destination user. |
| `TEMPORARY_ENTITLEMENT_GRANT` | `subscription_temporary_entitlement_granted` | Short-lived access during a store validation outage; expect a later purchase or expiration resolution. |

### Shared RevenueCat properties

- Identity and lineage: `user_analytics_id`, `revenuecat_app_user_id`, `revenuecat_aliases`, `revenuecat_event_id`, `revenuecat_event_type`, `source = revenuecat_webhook`
- Product / entitlement: `app_id`, `product_id`, `new_product_id`, `entitlement_id`, `entitlement_ids`, `period_type`, `presented_offering_id`, `store`
- Transaction: `transaction_id`, `original_transaction_id`, `renewal_number`, `period_type`, `offer_code`, `is_trial_conversion`, `is_family_share`
- Currency and value: `transaction_price_usd` (RevenueCat's USD `price`), `transaction_price_local`, `purchased_currency`, `revenue_usd`, `revenue_local`, `$revenue`, `$currency`
- Subscription state / MRR inputs: `subscription_state`, `subscription_period_days`, `mrr_usd`, `has_charge`
- Timing: `event_occurred_at`, `transaction_purchased_at`, `transaction_expires_at`, `event_timestamp_ms`, `purchased_at_ms`, `expiration_at_ms`
- Lifecycle details: `cancel_reason`, `expiration_reason`, `new_product_id`, `transferred_from`, `transferred_to`

`revenuecat_event_id` is also sent as `$insert_id`, so RevenueCat webhook retries should deduplicate in PostHog.

For production subscription and revenue metrics, always filter to `revenuecat_environment = 'PRODUCTION'`. RevenueCat sandbox events can use real store names, products, and prices (including apparent `subscription_renewed` revenue) but are not live customer revenue.

### Financial and timestamp rules

- The PostHog event `timestamp` is always RevenueCat's `event_timestamp_ms`, meaning the time RevenueCat generated the lifecycle event. A cancellation therefore appears when it happened, not at the original purchase time.
- `transaction_purchased_at` and `transaction_expires_at` describe the underlying transaction. They are dimensions, not the event time. This distinction matters for App Store renewals because payment collection can precede the new billing-period start.
- Only `INITIAL_PURCHASE`, `RENEWAL`, and `NON_RENEWING_PURCHASE` have `has_charge = true` and can carry `$revenue`. Cancellations, expirations, pauses, product changes, transfers, temporary grants, and billing issues must never be summed as revenue.
- `$revenue` and `$currency` are only emitted when RevenueCat supplies its USD `price`; `$currency` is then always `USD`. The buyer-facing price and its actual ISO currency are retained separately in `transaction_price_local` and `purchased_currency`. Never label a local-currency value as USD.
- `mrr_usd` is a normalized planning value: `transaction_price_usd * 30.4375 / subscription_period_days`. It is useful only for active paid subscriptions, after selecting the latest lifecycle record per `original_transaction_id`; it is not recognized revenue.
- A cancellation means `cancelled_active_until_expiry`, not inactive. Remove MRR only at `subscription_expired` or when the latest subscription state otherwise proves the entitlement is no longer active.

### Other RevenueCat webhook event types

These RevenueCat event types are intentionally not sent to PostHog by the current Edge Function. The list makes the boundary explicit; add a mapping and contract before enabling any of them for shared analysis.

| RevenueCat type | Current handling | Reason |
| --- | --- | --- |
| `TEST` | Acknowledge, skip | Dashboard test payload; not production analytics. |
| `PAYWALL_IMPRESSION`, `PAYWALL_CLOSE`, `PAYWALL_CANCEL`, `PAYWALL_EXIT_OFFER`, `PAYWALL_COMPONENT_INTERACTED` | Skip | Tether's client paywall event family is the canonical paywall UX source. |
| `VIRTUAL_CURRENCY_TRANSACTION` | Skip | Tether does not currently model RevenueCat virtual currency. |
| `EXPERIMENT_ENROLLMENT` | Skip | No RevenueCat experiment enrollment metric is currently approved. |
| `PURCHASE_REDEEMED` | Skip | Redemption is not a completed-charge metric; resulting transfers are captured through `TRANSFER`. |
| `SUBSCRIBER_ALIAS` | Skip | Legacy RevenueCat event; aliases are already used to resolve profiles. |
| `PRICE_INCREASE_CONSENT_REQUIRED`, `PRICE_INCREASE_CONSENT_APPROVED` | Skip | Price-consent monitoring is not yet an approved dashboard family. |

## Delivery prerequisites and caveats

The webhook forwards an event only when all of these are true:

1. RevenueCat sends one of the mapped event types.
2. The webhook can retrieve RevenueCat customer information and match the destination `app_user_id`, transfer destination, `original_app_user_id`, or an alias to a `profiles` row.
3. The matched profile has a non-empty `user_analytics_id`.
4. `POSTHOG_PROJECT_API_KEY` is configured in the Supabase Edge Function environment.

The function returns `503` on a lookup, Supabase, or PostHog delivery failure. That intentionally asks RevenueCat to retry instead of silently losing the analytics event. RevenueCat delivery is at-least-once, so `$insert_id = revenuecat_event_id` is required for deduplication.

`POSTHOG_PROJECT_API_KEY` determines the destination project. A successful `forwarded` response only proves that PostHog accepted the capture using that key; it does not prove the event arrived in Tether Production. Verify the secret belongs to project `471847` (`Tether Production`), not project `470659` (`Tether Dev`).

`REVENUECAT_WEBHOOK_AUTH` remains sufficient for webhook authentication. `REVENUECAT_WEBHOOK_SIGNING_SECRET` is optional and is only required when HMAC signing is enabled in RevenueCat; when configured, the function validates the raw request body signature and a five-minute replay window. Use both methods in production when possible.

The function requests current customer info after each webhook, including transfer sources, so `profiles.is_supporter` reflects present entitlement status rather than trying to infer status from each event type. Grace-period expiration is treated as active entitlement access.

RevenueCat retries failed deliveries only a finite number of times. For strict no-loss processing during a prolonged PostHog or Supabase outage, add a persistent webhook outbox and scheduled reprocessor; this Edge Function's retry behavior is not a durable queue.

## Current production status

On 2026-07-28, RevenueCat webhooks were found to be successfully forwarded but initially landed in Tether Dev because the Supabase `POSTHOG_PROJECT_API_KEY` pointed there. The key was corrected to Tether Production, and the first $4.99 monthly `subscription_initial_purchase` plus its subsequent `subscription_cancelled` event were replayed through the webhook into Production. A subsequent live `subscription_renewed` event arrived in Production on 2026-07-29, confirming the corrected routing.

Before publishing financial dashboards from the new contract, verify a live initial purchase, renewal, cancellation, and non-USD purchase where applicable. Confirm: `event_occurred_at` matches the lifecycle action, `$revenue` exists only on charge events, `purchased_currency` is correct, and all reporting filters to `revenuecat_environment = 'PRODUCTION'`.
