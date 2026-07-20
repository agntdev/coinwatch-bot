# Crypto Watchlist Alerts — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that lets users track cryptocurrency prices with customizable threshold/percentage alerts, daily summaries, and quiet hours. Owner panel provides aggregated analytics without exposing private user data. Alerts are non-spammy with cooldowns and retry-safe price validation.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual crypto watchers
- Bot owner/administrator

## Success criteria

- Users can add/remove coins via buttons or ticker input
- Price alerts trigger with correct conditions and cooldowns
- Daily summaries deliver at user-specified times
- Owner panel shows accurate aggregated metrics

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Begin onboarding and set timezone
  - inputs: timezone selection
  - outputs: timezone confirmation, main menu
- **/price** (command, actor: user, command: /price) — Request current price of a coin (with optional ticker parameter)
  - inputs: coin ticker
  - outputs: price data, 24h change
- **/owner_stats** (command, actor: owner, command: /owner_stats) — Display aggregated usage metrics to bot owner
  - inputs: none
  - outputs: user counts, top alerts
- **Add coin** (button, actor: user, callback: add_coin:start) — Initiate coin addition flow with popular coin buttons
  - inputs: coin selection
  - outputs: price confirmation, watchlist update
- **Manage watchlist** (button, actor: user, callback: manage_watchlist:view) — View and modify existing watchlist entries
  - inputs: coin selection
  - outputs: coin details, edit options

## Flows

### Onboarding
_Trigger:_ /start

1. Request timezone selection (quick replies)
2. Store user profile defaults
3. Display main menu with quick actions

_Data touched:_ user_profile

### Add Coin
_Trigger:_ add_coin:start

1. Show popular coin buttons
2. Handle ticker input normalization
3. Fetch current price
4. Confirm addition with quick reply buttons

_Data touched:_ watchlist_entry

### Set Threshold Alert
_Trigger:_ alert_threshold:configure

1. Request direction (above/below)
2. Request target price
3. Confirm rule with cooldown options

_Data touched:_ watchlist_entry

### Set Percentage Alert
_Trigger:_ alert_percent:configure

1. Request percentage value
2. Request period (default 1h)
3. Confirm rule with cooldown options

_Data touched:_ watchlist_entry

### Alert Delivery
_Trigger:_ price_update:check

1. Check quiet hours status
2. Validate price data
3. Evaluate alert conditions
4. Send alert with dismiss button
5. Update cooldown timestamp

_Data touched:_ watchlist_entry, alert_event

### Morning Summary
_Trigger:_ summary_time:trigger

1. Fetch current prices
2. Calculate 24h changes
3. Format summary message
4. Send to users with active summaries

_Data touched:_ user_profile, watchlist_entry

### Owner Analytics
_Trigger:_ /owner_stats

1. Verify owner ID
2. Aggregate user counts
3. Fetch top alerts
4. Format metrics dashboard

_Data touched:_ user_profile, alert_event, owner_metrics

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user_profile** _(retention: persistent)_ — User-specific configuration and preferences
  - fields: telegram_id, display_name, timezone, quiet_hours_start, quiet_hours_end, summary_time, language, global_cooldown
- **watchlist_entry** _(retention: persistent)_ — Monitored cryptocurrency and alert rules
  - fields: user_id, ticker, display_name, threshold_alerts, percent_move_alerts, last_alert_time, last_known_price, last_checked_time
- **alert_event** _(retention: persistent)_ — Record of triggered alerts for metrics and dismissal
  - fields: user_id, ticker, alert_type, old_price, new_price, percent_change, fired_time
- **owner_metrics** _(retention: persistent)_ — Aggregated system-wide statistics
  - fields: total_users, active_users_30d, top_tickers, top_alert_types, alerts_last_24h

## Integrations

- **Telegram** (required) — Bot API messaging and owner panel access
- **Price Feed API** (required) — Cryptocurrency price data
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /owner_stats (only accessible to configured owner ID)

## Notifications

- Price threshold alerts
- Percentage move alerts
- Daily morning summaries
- Quiet hours alert consolidation
- Error notifications for invalid tickers

## Permissions & privacy

- User watchlists remain private
- Owner cannot access individual alert rules
- Price data validation before alerting
- Quiet hours respect user preferences

## Edge cases

- Unknown/invalid ticker handling
- Price feed failures with retries
- Quiet hours during alert trigger
- Cooldown expiration tracking
- Multiple alert types per coin

## Required tests

- Verify alert suppression during quiet hours
- Test price threshold alert triggering
- Validate daily summary delivery timing
- Confirm owner metrics aggregation accuracy
- Test error handling for invalid tickers

## Assumptions

- Default timezone is UTC for deferred users
- 1-hour period is standard for percentage alerts
- 6-hour default cooldown prevents spam
- Price feed retries up to 3 times with backoff
- Owner metrics dashboard format is basic text
