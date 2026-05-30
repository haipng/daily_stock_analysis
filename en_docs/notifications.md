# Notification Capability Baseline

This document records the notification capability P0-P7 terminal state: channels, configuration keys, GitHub Actions mapping, Web settings metadata, CLI diagnostic aperture, Web one-click testing, custom Webhook Body template semantics, notification routing strategy, noise reduction mechanism, aggregated report failure isolation, ntfy/Gotify first-class channels, WebPush/Apprise evaluation, as well as local/Docker/GitHub Actions/Desktop scenario-based configuration documentation. P0 establishes only baseline and read-only diagnostics; P1 adds Web single-channel real testing; P2 productizes existing Body templates; P3 adds report/alert/system_error routing; P4 adds in-process noise suppression; P5 strengthens test diagnostics and aggregated report per-channel failure isolation; P6-A adds ntfy; P6-C adds Gotify; P6-D evaluates WebPush/Apprise only; P7 closes documentation and Actions env mapping automation without adding runtime dependencies, configuration entry points, per-URL templates, cross-process persistence, real daily digests, or retry loops.

## Channel Baseline

| Channel | Type | Minimal Key | Advanced Key | Description |
| --- | --- | --- | --- | --- |
| WeChat Work | Static Config | `WECHAT_WEBHOOK_URL` | `WECHAT_MSG_TYPE` | Participates in batch notification sending after configuration |
| Feishu Webhook | Static Config | `FEISHU_WEBHOOK_URL` | `FEISHU_WEBHOOK_SECRET`, `FEISHU_WEBHOOK_KEYWORD` | `FEISHU_APP_ID` / `FEISHU_APP_SECRET` will not alone enable group Webhook push |
| Telegram | Static Config | `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` | `TELEGRAM_MESSAGE_THREAD_ID` | Token and chat id must coexist |
| Email | Static Config | `EMAIL_SENDER`, `EMAIL_PASSWORD` | `EMAIL_RECEIVERS`, `EMAIL_SENDER_NAME` | Sends to self when `EMAIL_RECEIVERS` is empty |
| Pushover | Static Config | `PUSHOVER_USER_KEY`, `PUSHOVER_API_TOKEN` | - | Two keys must coexist |
| ntfy | Static Config | `NTFY_URL` | `NTFY_TOKEN`, `WEBHOOK_VERIFY_SSL` | `NTFY_URL` must include topic path, e.g., `https://ntfy.sh/my-topic` |
| Gotify | Static Config | `GOTIFY_URL`, `GOTIFY_TOKEN` | `WEBHOOK_VERIFY_SSL` | `GOTIFY_URL` is server base URL, not including `/message`; token is sent via `X-Gotify-Key` Header |
| PushPlus | Static Config | `PUSHPLUS_TOKEN` | `PUSHPLUS_TOPIC` | `PUSHPLUS_TOPIC` only effective when token exists |
| Server Chan 3 | Static Config | `SERVERCHAN3_SENDKEY` | - | Mobile app push |
| Custom Webhook | Static Config | `CUSTOM_WEBHOOK_URLS` | `CUSTOM_WEBHOOK_BEARER_TOKEN`, `CUSTOM_WEBHOOK_BODY_TEMPLATE`, `WEBHOOK_VERIFY_SSL` | Supports multiple URLs, comma-separated |
| Discord | Static Config | `DISCORD_WEBHOOK_URL` or `DISCORD_BOT_TOKEN` + `DISCORD_MAIN_CHANNEL_ID` | `DISCORD_INTERACTIONS_PUBLIC_KEY` | Both Webhook and Bot can be enabled for sending |
| Slack | Static Config | `SLACK_WEBHOOK_URL` or `SLACK_BOT_TOKEN` + `SLACK_CHANNEL_ID` | - | Bot prioritizes text and image same-channel sending |
| AstrBot | Static Config | `ASTRBOT_URL` | `ASTRBOT_TOKEN`, `WEBHOOK_VERIFY_SSL` | `ASTRBOT_TOKEN` is optional |
| `UNKNOWN` | Fallback Enum | - | - | Only for unknown channels, not enabled by static environment variables |
| DingTalk Session | Runtime Context | - | - | Extracted from source message context, cannot be determined by `.env` static configuration |
| Feishu Session | Runtime Context | - | - | Extracted from source message context, interactive command results only return to source session |
| Telegram Session | Runtime Context | - | - | Extracted from source message context, interactive command results only return to source session |

## Minimal / Advanced Layering

- Minimal Key: Minimum configuration sufficient to enable a notification channel.
- Advanced Key: Only affects authentication, security, format, threading, groups, certificate verification, or display behavior; cannot alone enable a channel.
- `NOTIFICATION_*_CHANNELS` in P3 belongs to Advanced Keys: only narrows already-enabled channels, does not alone enable channels.
- `NOTIFICATION_DEDUP_TTL_SECONDS`, `NOTIFICATION_COOLDOWN_SECONDS`, `NOTIFICATION_QUIET_HOURS`, `NOTIFICATION_TIMEZONE`, `NOTIFICATION_MIN_SEVERITY`, `NOTIFICATION_DAILY_DIGEST_ENABLED` in P4 belong to Advanced Keys: only affect sending policy of already-enabled static channels, do not alone enable channels.
- `REPORT_SHOW_LLM_MODEL` is a report display switch: defaults to `true` to show the LLM model used in this analysis at the bottom of notification reports; set to `false` to hide. This parameter only affects report rendering and does not alter runtime provider/model/Base URL, LiteLLM routing, model saving, migration, or cleanup logic; fallback method is to change back to `true` or delete this variable.
- `WEBHOOK_VERIFY_SSL` is a shared certificate verification switch for webhook-style HTTPS notification requests using this configuration.
- WebPush, Apprise, finer-grained routing, cross-process noise reduction, and real daily digests are not entering runtime implementation this round; if related configuration is introduced in the future, should first update this document, `.env.example`, Web metadata, and regression tests.
- Bark maintains custom webhook baseline without adding first-class `BARK_*` configuration.

## GitHub Actions Mapping

The repository's built-in `.github/workflows/00-daily-analysis.yml` only explicitly imports fixed variable names. P0/P3/P4/P6 have incorporated Body templates, security items, PushPlus topic, routing, noise suppression, ntfy, and Gotify notification keys into the default workflow. The following table is generated by `scripts/generate_notification_actions_env_table.py` from the workflow `env:` and notification diagnostic metadata to avoid manual correspondence tables and real Actions mapping from continuing to drift.

<!-- notification-actions-env-table:start -->

| Key | Tier | Channel / Feature | Actions Source | Default |
| --- | --- | --- | --- | --- |
| `WECHAT_WEBHOOK_URL` | minimal | wechat | Secret | - |
| `WECHAT_MSG_TYPE` | advanced | wechat | Variable or Secret | `markdown` |
| `FEISHU_WEBHOOK_URL` | minimal | feishu | Secret | - |
| `FEISHU_WEBHOOK_SECRET` | advanced | feishu | Secret | - |
| `FEISHU_WEBHOOK_KEYWORD` | advanced | feishu | Variable or Secret | - |
| `TELEGRAM_BOT_TOKEN` | minimal | telegram | Secret | - |
| `TELEGRAM_CHAT_ID` | minimal | telegram | Secret | - |
| `TELEGRAM_MESSAGE_THREAD_ID` | advanced | telegram | Secret | - |
| `EMAIL_SENDER` | minimal | email | Variable or Secret | - |
| `EMAIL_PASSWORD` | minimal | email | Secret | - |
| `EMAIL_RECEIVERS` | advanced | email | Variable or Secret | - |
| `EMAIL_SENDER_NAME` | advanced | email | Variable or Secret | `daily_stock_analysis assistant` |
| `PUSHOVER_USER_KEY` | minimal | pushover | Secret | - |
| `PUSHOVER_API_TOKEN` | minimal | pushover | Secret | - |
| `NTFY_URL` | minimal | ntfy | Secret | - |
| `NTFY_TOKEN` | advanced | ntfy | Secret | - |
| `GOTIFY_URL` | minimal | gotify | Secret | - |
| `GOTIFY_TOKEN` | minimal | gotify | Secret | - |
| `PUSHPLUS_TOKEN` | minimal | pushplus | Secret | - |
| `PUSHPLUS_TOPIC` | advanced | pushplus | Variable or Secret | - |
| `CUSTOM_WEBHOOK_URLS` | minimal | custom | Secret | - |
| `CUSTOM_WEBHOOK_BEARER_TOKEN` | advanced | custom | Secret | - |
| `CUSTOM_WEBHOOK_BODY_TEMPLATE` | advanced | custom | Variable or Secret | - |
| `WEBHOOK_VERIFY_SSL` | advanced | ntfy, gotify, custom, astrbot | Variable or Secret | `true` |
| `DISCORD_WEBHOOK_URL` | minimal | discord | Secret | - |
| `DISCORD_BOT_TOKEN` | minimal | discord | Secret | - |
| `DISCORD_MAIN_CHANNEL_ID` | minimal | discord | Secret | - |
| `ASTRBOT_URL` | minimal | astrbot | Secret | - |
| `ASTRBOT_TOKEN` | advanced | astrbot | Secret | - |
| `SERVERCHAN3_SENDKEY` | minimal | serverchan3 | Secret | - |
| `SLACK_WEBHOOK_URL` | minimal | slack | Secret | - |
| `SLACK_BOT_TOKEN` | minimal | slack | Secret | - |
| `SLACK_CHANNEL_ID` | minimal | slack | Secret | - |
| `NOTIFICATION_REPORT_CHANNELS` | advanced | routing | Variable or Secret | - |
| `NOTIFICATION_ALERT_CHANNELS` | advanced | routing | Variable or Secret | - |
| `NOTIFICATION_SYSTEM_ERROR_CHANNELS` | advanced | routing | Variable or Secret | - |
| `NOTIFICATION_DEDUP_TTL_SECONDS` | advanced | noise | Variable or Secret | `0` |
| `NOTIFICATION_COOLDOWN_SECONDS` | advanced | noise | Variable or Secret | `0` |
| `NOTIFICATION_QUIET_HOURS` | advanced | noise | Variable or Secret | - |
| `NOTIFICATION_TIMEZONE` | advanced | noise | Variable or Secret | - |
| `NOTIFICATION_MIN_SEVERITY` | advanced | noise | Variable or Secret | - |
| `NOTIFICATION_DAILY_DIGEST_ENABLED` | advanced | noise | Variable or Secret | `false` |

<!-- notification-actions-env-table:end -->

The default workflow still does not map `MARKDOWN_TO_IMAGE_CHANNELS` and `MERGE_EMAIL_NOTIFICATION`. They are sending format or aggregation behavior switches, not channel credentials; automatically reading same-named Secret/Variable in Actions would introduce additional behavior changes.

## CLI Diagnostics

```bash
python main.py --check-notify
```

This command is read-only for configuration and does not send notifications or write to `.env`. It executes immediately after configuration loading and log initialization, completes, then exits directly without entering Web, scheduling, market review, or default analysis flows.

- Exit code `0`: No error-level diagnostics.
- Exit code `1`: Errors exist, such as 0 static notification channels configured, or paired keys only partially configured.

## Web One-Click Testing

The Web Settings page's "Notification Channels" section provides single-channel testing entry points. Testing uses the current page draft values to synthesize temporary configuration, sends a real test notification, but does not save `.env` or modify runtime global configuration.

- Test scope: 13 static notification channels, excluding `UNKNOWN` and runtime context channels.
- Regular channels: Return single send result, latency, and generic error code.
- Custom Webhook: Return attempts in URL order, showing each URL's success/failure, HTTP status, latency, and error code; partial success shows top-level message with success count / total count.
- Return results are desensitized: token, secret, password, Bearer, complete webhook query, and suspected path token.
- Missing configuration or send failure returns `success=false` and does not affect saved configuration or default analysis flow.

## Custom Webhook Body Template

`CUSTOM_WEBHOOK_BODY_TEMPLATE` is the global JSON body template for custom Webhooks. After configuration, it automatically takes effect before URL auto-identification, thus overriding auto-payload for Bark, Slack, Discord, DingTalk, etc. Without configuration, still uses original URL auto-identification; if rendered output is not a valid JSON object, logs error and falls back to default payload without interrupting main notification flow.

Available placeholders:

- `$content_json`: JSON-escaped notification content, recommended for default use.
- `$title_json`: JSON-escaped notification title, recommended for default use.
- `$content` / `$title`: Raw string, no JSON escaping. Content with quotes, backslashes, or newlines may result in invalid JSON and trigger fallback.

Generic webhook example:

```env
CUSTOM_WEBHOOK_BODY_TEMPLATE={"title":$title_json,"content":$content_json}
```

For Bark via custom webhook, directly place Bark endpoint in `CUSTOM_WEBHOOK_URLS` without additional `BARK_*` configuration. Without global template configuration, the system auto-generates `title` / `body` / `group` based on `api.day.app`; if global template is configured, write Bark body yourself:

```env
CUSTOM_WEBHOOK_URLS=https://api.day.app/YOUR_BARK_KEY
```

```env
CUSTOM_WEBHOOK_BODY_TEMPLATE={"title":$title_json,"body":$content_json,"group":"stock"}
```

AstrBot is already a first-class notification channel, prioritize using `ASTRBOT_URL` and optional `ASTRBOT_TOKEN`. Only use custom webhook template when needing to put AstrBot compatible endpoint in `CUSTOM_WEBHOOK_URLS`, for example:

```env
CUSTOM_WEBHOOK_BODY_TEMPLATE={"content":$content_json}
```

ntfy is already a first-class notification channel, prioritize using `NTFY_URL` and optional `NTFY_TOKEN`. `NTFY_URL` represents the complete topic endpoint, e.g., `https://ntfy.sh/my-topic` or `https://self-hosted:port/my-topic`; the system parses the last path segment as the topic and sends JSON publish to server root:

```env
NTFY_URL=https://ntfy.sh/my-topic
NTFY_TOKEN=
```

Gotify is already a first-class notification channel, prioritize using `GOTIFY_URL` and `GOTIFY_TOKEN`. `GOTIFY_URL` represents Gotify server base URL, may include reverse proxy path prefix but not `/message`; system appends fixed `/message` API when sending and sends application token via `X-Gotify-Key` Header. `NTFY_URL` is complete topic endpoint while `GOTIFY_URL` is server base URL—this is an intentional choice due to the two services' API design differences:

```env
GOTIFY_URL=https://gotify.example
GOTIFY_TOKEN=app-token
```

```env
# Reverse proxy path prefix example; actual request sends to https://example.com/gotify/message
GOTIFY_URL=https://example.com/gotify
GOTIFY_TOKEN=app-token
```

NapCat / OneBot HTTP API requires adjusting per actual endpoint and target type. Below are just common body form examples; user_id, group_id, URL path, and authentication should follow your NapCat configuration:

```env
# Private message: CUSTOM_WEBHOOK_URLS=http://127.0.0.1:3000/send_private_msg
CUSTOM_WEBHOOK_BODY_TEMPLATE={"user_id":123456,"message":$content_json}
```

```env
# Group message: CUSTOM_WEBHOOK_URLS=http://127.0.0.1:3000/send_group_msg
CUSTOM_WEBHOOK_BODY_TEMPLATE={"group_id":123456789,"message":$content_json}
```

## Notification Routing Strategy

P3 adds three notification routing configurations:

| Routing Type | Configuration Key | Current Producers |
| --- | --- | --- |
| `report` | `NOTIFICATION_REPORT_CHANNELS` | Single stock push, aggregated daily reports, market review, merged push, Feishu doc success links |
| `alert` | `NOTIFICATION_ALERT_CHANNELS` | EventMonitor triggered notifications |
| `system_error` | `NOTIFICATION_SYSTEM_ERROR_CHANNELS` | Reserved capability; currently no auto system error producers added |

Configuration value is comma-separated channel enum: `wechat,feishu,telegram,email,pushover,ntfy,gotify,pushplus,serverchan3,custom,discord,slack,astrbot`.

- Empty or unconfigured: Maintain old behavior, send to all configured static channels.
- Non-empty: Only send to intersection of routing list and configured channels; empty intersection does not fallback to all channels.
- `send_to_context()` is not subject to routing restrictions; bot session context still receives replies to triggered tasks.
- Interactive commands (DingTalk session, Feishu session, Telegram) with source context will skip static notification channels like `FEISHU_WEBHOOK_URL`; `SCHEDULE`, CLI, API, or contextless tasks still follow report routing.
- Routing filtering occurs before Markdown-to-image conversion; `MARKDOWN_TO_IMAGE_CHANNELS` only applies to channel subset after routing.
- `MERGE_EMAIL_NOTIFICATION` needs no additional configuration; as long as `email` remains in report-routed channels, existing merged email behavior is preserved.
- `--check-notify` reports unknown channel values as errors and legally-enabled routing targets as warnings.

## Aggregated Report Failure Isolation

P5 strengthens aggregated report notification path failure boundaries: `_send_notifications()` independently sends to each static notification channel after report routing filtering. If a channel throws an exception, it logs and is treated as that channel's failure but does not skip subsequent channels or interrupt analysis main flow.

- Email isolation by receiver group; failure in one recipient group does not prevent subsequent groups from sending.
- When any static channel sends successfully, P4 noise reservation writes to formal record; when all static channels fail or throw exceptions, reservation is released.
- `send_to_context()` remains independent of static channel route and noise reduction record, used for replying to Bot session context triggering the task.

## Notification Noise Reduction Mechanism

P4 adds in-process noise reduction, only affecting static-configured channels, not affecting Bot-triggered session receipts in `send_to_context()`. All configuration defaults to off; defaults preserve old behavior without configuration.

| Configuration Key | Default | Description |
| --- | --- | --- |
| `NOTIFICATION_DEDUP_TTL_SECONDS` | `0` | Same stable dedup key only sends once within TTL; `0` disables |
| `NOTIFICATION_COOLDOWN_SECONDS` | `0` | Same cooldown key rate-limited within window; `0` disables |
| `NOTIFICATION_QUIET_HOURS` | Empty | Silent period, format `HH:MM-HH:MM`, supports midnight cross |
| `NOTIFICATION_TIMEZONE` | Empty | Silent period timezone, e.g., `Asia/Shanghai`; empty uses Python runtime local timezone (usually from process `TZ` or system timezone) |
| `NOTIFICATION_MIN_SEVERITY` | Empty | `info`, `warning`, `error`, `critical`; empty does not filter |
| `NOTIFICATION_DAILY_DIGEST_ENABLED` | `false` | Reserved configuration; currently will not send daily digests or persist digest content |

Default severity levels:

- `report`: `info`
- `alert`: `warning`
- `system_error`: `error`
- Unknown or unset routing: `info`

Implementation boundaries:

- Dedup / cooldown state is current Python process internal dict, applies to `main.py` single process and `--serve` single worker.
- In `uvicorn --workers N`, multi-container, or multi-machine scenarios, state is not shared; noise reduction approximates per-worker.
- Pipeline single stock and aggregated report paths use stable keys to avoid report generation-time changes breaking dedup; other unreported `dedup_key` notifications dedup by content hash.
- Calls without explicitly passed `cooldown_key` share default cooldown slot by routing and severity level, e.g., report / info regular notifications share same slot.
- Concurrent sends of same key within process first occupy short-lived in-flight slots preventing burst repeats; when all static channels fail, in-flight slot is released without writing formal dedup / cooldown state.
- Noise reduction exception fail-open: log and continue sending static channels.
- When `NOTIFICATION_TIMEZONE` is empty, uses `datetime.now().astimezone()` resolved runtime local timezone; Actions/Docker scenarios recommend explicitly configuring `NOTIFICATION_TIMEZONE` to avoid timezone ambiguity.

## WebPush / Apprise Evaluation

P6-D evaluates only design, adding no dependencies, `.env` configuration, or runtime notification paths. Conclusion: both are unsuitable for direct mixing into this round's channel implementation PR.

If WebPush is implemented in future, must first separately design subscription lifecycle and security boundaries:

- Requires Web frontend Service Worker registration; `PushManager.subscribe()` depends on secure context, production usually requires HTTPS, local development can use localhost.
- Requires VAPID public/private keys; send public key during subscription, service holds private key and protects key rotation strategy.
- Requires browser permission interaction; subscription must trigger from user gesture, cannot silently enable in background.
- `PushSubscription` includes endpoint and encryption key; endpoint is capability URL and should treat as secret and display desensitized.
- Requires persisting subscriptions, handling subscription failure and device unbinding; current `.env` / single-process configuration model unsuitable for directly inserting multiple user/device subscriptions.
- API for submitting, deleting, updating subscriptions requires authentication and CSRF protection; cannot rely only on frontend hidden entry points.

If Apprise is introduced later, should first assess as optional dependency rather than default:

- Apprise is universal notification library with broad coverage but overlaps existing WeChat, Telegram, Discord, Slack, ntfy, Gotify, Pushover first-class channels.
- Must evaluate dependency size, install failure path, Docker image bloat, GitHub Actions dependency caching, and optional extras strategy.
- Secret passing cannot directly expose complete Apprise URLs; must unify desensitization, Web test target masking, and error log filtering.
- Send failure should isolate within Apprise channel, not affecting existing channel failure isolation semantics.
- If adopting Apprise, recommend first adding separate experimental channel or CLI-only spike before deciding whether to incorporate into Web settings page and Actions env.

## Local Configuration

Local operation prioritizes project root `.env`. Copy `.env.example` then populate at least one minimal key to enable corresponding static notification channel; advanced keys only change authentication, security, format, routing, or noise reduction behavior and do not alone enable channels.

```bash
python main.py --check-notify
```

`--check-notify` is read-only diagnostics: sends no notifications, does not write `.env`, does not enter analysis flow. After Web UI configuration, also can send real test messages in system settings page with single-channel testing; this test only uses page draft temporary configuration and does not save `.env`.

## Docker

Docker scenarios can inject runtime environment variables via `--env-file .env` / Compose `env_file`, or mount `.env` to let Web settings page and backend read/write same configuration file. Only injecting environment variables without mounting `.env` may cause Web settings page saved values to be rerun-covered after container restart.

Silent period noise reduction recommends explicitly configuring `NOTIFICATION_TIMEZONE` to avoid container default timezone mismatching expectations. Self-signed internal network webhooks can temporarily use `WEBHOOK_VERIFY_SSL=false` but never disable certificate verification on public network links.

## GitHub Actions

Default `00-daily-analysis.yml` only reads Secrets/Variables explicitly mapped in table. After adding repository Secret or Variable, only variable names already appearing in workflow `env:` enter running process; arbitrary-numbered variables like `STOCK_GROUP_N` / `EMAIL_GROUP_N` will not auto-import.

Secrets suit token, password, webhook URL and other sensitive items; Variables suit `WECHAT_MSG_TYPE`, `EMAIL_SENDER_NAME`, routing, noise reduction window, timezone and other non-sensitive behavior configuration. `MARKDOWN_TO_IMAGE_CHANNELS` and `MERGE_EMAIL_NOTIFICATION` default unmapped; if needed in own fork, should explicitly modify workflow and supplement corresponding tests.

## Desktop

Desktop reuses Web settings page notification configuration and single-channel testing entry points. Notification testing sends real test messages but only uses current page draft values and does not auto-save; persistence still requires clicking save configuration.

Desktop can restore `.env` via configuration export/import. When rolling back a notification channel, clear that channel's minimal key and save; advanced keys remaining do not alone enable channel but recommend clearing together to reduce subsequent troubleshooting noise.

## Rollback Methods

- Local / Docker: Restore old `.env`, or delete corresponding channel minimal key and restart process.
- GitHub Actions: Clear or delete corresponding Secret / Variable; unmapped keys do not enter workflow running process.
- Desktop: Use configuration backup to import old `.env`, or clear corresponding channel configuration in settings page and save.
- Version rollback: P6/P7 added `NTFY_*`, `GOTIFY_*`, routing, and noise reduction keys are ignored in older versions; to avoid confusion, should also remove from `.env` or Actions configuration.
