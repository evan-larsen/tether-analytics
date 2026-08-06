# Subscription Status

## Decisions

- How many active subscriptions does Tether have right now?
- What share of average full DAU across the last 7 complete days had a subscription on those same days?
- What share of full rolling 30-day MAU across the last 30 complete days had a subscription as of yesterday?
- How many active subscriptions does Tether have each complete day?
- Is the subscription base growing, flat, or shrinking over time?
- Which paywall entry points are driving unique viewers, initial purchasers, and conversion?
- Are we looking at subscription objects rather than supporter-access counts?

## Approved insights

- Current active subscriptions
- DAU Subscribed % (7d Avg Rolling)
- MAU Subscribed % (30d Rolling)
- Active subscriptions
- Paywall Entry Point Conversion
- Supporter prompt paywall rate by appearance (30d)

## Exact query logic

- For the current metric, use the latest RevenueCat-backed lifecycle state per `original_transaction_id` and count subscriptions whose `expires_at` is still beyond `now()`
- For `DAU Subscribed % (7d Avg Rolling)`, use the last 7 complete US/Mountain days only, count distinct `user_analytics_id` across all events per day, join each day to `subscription_active_daily_snapshots` on `snapshot_day` and `user_analytics_id`, then divide average subscribed full DAU by average full DAU
- For `MAU Subscribed % (30d Rolling)`, use the last 30 complete US/Mountain days only, count distinct `user_analytics_id` across all events, anchor subscriber state to yesterday's `subscription_active_daily_snapshots`, then divide subscribed full MAU by total rolling full MAU
- For `Supporter prompt paywall rate by appearance (30d)`, deduplicate `supporter_prompt_viewed` and prompt-origin `paywall_viewed` records by `user_analytics_id`, `reading_session_id`, and `supporter_prompt_appearance_number`; join matching prompt/paywall records, group open rate by prompt appearance number, and add a volume-weighted `Total` row
- For both percentage metrics, format the saved insight as `percent` in PostHog
- For `Paywall Entry Point Conversion`, group `paywall_viewed` users by `entry_point`, count unique viewers, join to each user's first Production `subscription_initial_purchase`, add an `Unknown` row for purchasers with no tracked paywall view in the three approved entry-point buckets, and add a de-duplicated `Total` row
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
