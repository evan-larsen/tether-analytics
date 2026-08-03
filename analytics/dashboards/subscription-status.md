# Subscription Status

## Decisions

- How many active subscriptions does Tether have right now?
- What share of today's active readers currently has a subscription?
- What share of trailing-30-day active readers currently has a subscription?
- How many active subscriptions does Tether have each complete day?
- Is the subscription base growing, flat, or shrinking over time?
- Are we looking at subscription objects rather than supporter-access counts?

## Approved insights

- Current active subscriptions
- % of DAU with subscription
- % of MAU with subscription
- Active subscriptions

## Exact query logic

- For the current metric, use the latest RevenueCat-backed lifecycle state per `original_transaction_id` and count subscriptions whose `expires_at` is still beyond `now()`
- For `% of DAU with subscription`, use distinct `user_analytics_id` on today's `reading_session_completed` events as the denominator, then divide subscribed users by total DAU users
- For `% of MAU with subscription`, use distinct `user_analytics_id` on trailing-30-day `reading_session_completed` events as the denominator, then divide subscribed users by total MAU users
- For both percentage metrics, derive the subscribed-user numerator from the latest active subscription state and format the saved insight as `percent` in PostHog
- Pull from `subscription_active_daily_snapshots`
- Group by `snapshot_day`
- Count distinct `original_transaction_id`
- Treat `subscription_cancelled` as still active until `expires_at`
- Keep the metric subscription-based rather than user-based so future family or multi-seat plans still count as one subscription

## Known limitations

- The current production implementation has only recently begun collecting RevenueCat lifecycle events, so older history is limited
- The bold-number current metric is intraday and can change immediately as new lifecycle events arrive
- The percentage metrics are also intraday because today's DAU and the current active-subscription set both change throughout the day
- The saved view currently retains the latest 90 complete US/Mountain snapshot days
- Internal/test traffic is not the main risk for this metric, but RevenueCat sandbox events must remain excluded through the underlying saved view
