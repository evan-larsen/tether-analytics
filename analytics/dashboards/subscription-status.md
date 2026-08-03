# Subscription Status

## Decisions

- How many active subscriptions does Tether have right now?
- What share of average daily active readers across the last 7 complete days had a subscription on those same days?
- What share of rolling 30-day active readers across the last 30 complete days had a subscription as of yesterday?
- How many active subscriptions does Tether have each complete day?
- Is the subscription base growing, flat, or shrinking over time?
- Are we looking at subscription objects rather than supporter-access counts?

## Approved insights

- Current active subscriptions
- 7d Avg DAU Subscribed %
- 30d MAU Subscribed %
- Active subscriptions

## Exact query logic

- For the current metric, use the latest RevenueCat-backed lifecycle state per `original_transaction_id` and count subscriptions whose `expires_at` is still beyond `now()`
- For `7d Avg DAU Subscribed %`, use the last 7 complete US/Mountain days only, count distinct `user_analytics_id` on `reading_session_completed` per day, join each day to `subscription_active_daily_snapshots` on `snapshot_day` and `user_analytics_id`, then divide average subscribed DAU by average DAU
- For `30d MAU Subscribed %`, use the last 30 complete US/Mountain days only, count distinct `user_analytics_id` on `reading_session_completed`, anchor subscriber state to yesterday's `subscription_active_daily_snapshots`, then divide subscribed MAU by total rolling MAU
- For both percentage metrics, format the saved insight as `percent` in PostHog
- Pull from `subscription_active_daily_snapshots`
- Group by `snapshot_day`
- Count distinct `original_transaction_id`
- Treat `subscription_cancelled` as still active until `expires_at`
- Keep the metric subscription-based rather than user-based so future family or multi-seat plans still count as one subscription

## Known limitations

- The current production implementation has only recently begun collecting RevenueCat lifecycle events, so older history is limited
- The bold-number current metric is intraday and can change immediately as new lifecycle events arrive
- The percentage metrics now exclude today, so they move only when a completed day rolls over or the underlying hourly snapshot refreshes
- The saved view currently retains the latest 90 complete US/Mountain snapshot days
- Internal/test traffic is not the main risk for this metric, but RevenueCat sandbox events must remain excluded through the underlying saved view
