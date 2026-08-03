# Active Subscription Snapshots

- PostHog saved query name: `subscription_active_daily_snapshots`
- Type: materialized SQL view
- Refresh cadence: `1hour`

This view is the canonical historical daily snapshot for active subscriptions in Tether Production. It is intentionally subscription-based rather than supporter-based, so one family plan or future multi-seat subscription still counts as one active subscription.

## Definition

- One row per complete US/Mountain `snapshot_day` and `original_transaction_id`
- Source events are RevenueCat lifecycle events forwarded into PostHog
- Production-only filter: `revenuecat_environment = 'PRODUCTION'`
- The in-progress Utah-time day is excluded

A subscription is active on snapshot day `D` when the latest lifecycle record for its `original_transaction_id` as of the end of day `D` still has entitlement access beyond that day.

## Why this is the source of truth

- `original_transaction_id` is the subscription identity, so it survives user-level access changes and keeps subscription counts separate from people counts
- `subscription_cancelled` does not remove a subscription immediately; it remains active until its expiration timestamp
- App-event state snapshots such as `is_supporter` are not sufficient for this metric because a subscriber can remain active on days when they emit no product events

## Materialization behavior

- The view retains the latest 90 complete snapshot days
- It scans the trailing 455 days of subscription lifecycle events so the 90-day history can still include subscriptions with long billing periods, such as annual plans
- Materialize hourly because PostHog supports cadence, not an exact run time, and daily reporting should refresh shortly after the Utah-time day boundary

## Querying

Count active subscriptions from the snapshot rows:

```sql
SELECT
    snapshot_day,
    countDistinct(original_transaction_id) AS active_subscriptions
FROM subscription_active_daily_snapshots
GROUP BY snapshot_day
ORDER BY snapshot_day
```

## Notes

- This metric is about active subscriptions, not active paying people
- If Tether later adds plans longer than one year, revisit the 455-day source-event lookback
- Treat the SQL saved in PostHog as the executable source of truth
