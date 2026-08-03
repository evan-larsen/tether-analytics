# Subscription Status

## Decisions

- How many active subscriptions does Tether have each complete day?
- Is the subscription base growing, flat, or shrinking over time?
- Are we looking at subscription objects rather than supporter-access counts?

## Approved insights

- Active subscriptions

## Exact query logic

- Pull from `subscription_active_daily_snapshots`
- Group by `snapshot_day`
- Count distinct `original_transaction_id`
- Treat `subscription_cancelled` as still active until `expires_at`
- Keep the metric subscription-based rather than user-based so future family or multi-seat plans still count as one subscription

## Known limitations

- The current production implementation has only recently begun collecting RevenueCat lifecycle events, so older history is limited
- The saved view currently retains the latest 90 complete US/Mountain snapshot days
- Internal/test traffic is not the main risk for this metric, but RevenueCat sandbox events must remain excluded through the underlying saved view
