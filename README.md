# Avenue Guard

Avenue Guard is the Discord utility bot for GD Avenue. It handles moderation guardrails, live level request waves, weekly activity rewards, staff tickets, DM help flows, forum reminders, sticky notices, configurable auto-responses, and a few community fun commands.

The bot is intentionally built around one configured server. Most behavior is controlled from `config.json`, with message-trigger responses in `responses.json` and persistent state in the configured SQLite database path. On Render, the bot auto-uses `/var/data/avenue-guard/bot.db` when a Persistent Disk is mounted there.

## Core Features

### Server Guardrails
- Restricts the bot to one guild using `guild.allowed_guild_id`.
- Auto-deletes messages and reactions in the configured creator-points proof channel.
- Applies a restriction role to users who post/react where they should not.
- Allows configured whitelist roles to bypass that restriction flow.
- Sends configurable DMs when users gain watched roles.

### Weekly Activity Requests
- Counts eligible member messages per configured server week.
- Skips configured roles and channels.
- Skips and logs repeated low-effort message farming patterns before they count.
- Buffers activity writes briefly with `tracking.activity_flush_seconds` so busy chat does less SQLite work.
- Provides `/tracking top`, `/tracking me`, `/tracking reset`, `/tracking force_dm`, `/tracking disable_reward`, and `/tracking enable_reward`.
- Tracks top-5 weekly streaks and shows streak markers in tracking embeds.
- DMs weekly winners with a configurable request embed.
- Supports claim, decline confirmation, timeout, reminders, and automatic offer to the next eligible member.
- Posts weekly request submissions as configurable embeds with `Send`, `Reject`, and `Other` review buttons.
- Weekly submitted requests use the same staff review/result workflow as live wave requests, but do not count toward or appear in any request wave summary.
- Weekly submitted request embeds include submission timing and reviewer action buttons.
- Admins can disable the automatic weekly request reward for the current tracking week.
- Logs weekly request events, including manual `/tracking force_dm` outcomes, to SQLite and optionally to a log channel.
- Logs weekly request recording failures instead of silently closing a claim when the staff request channel cannot be used.
- Weekly request log embeds use readable event names, colors, member context, and structured details.
- Posts a private weekly recap embed to the daily-summary/log channel with activity, weekly request, review, streak, and anti-farm signals.

### Live Level Request Waves
- Posts a persistent request embed/button in `level_requests.request_channel`.
- Mods can recreate or refresh the request button with `/refresh-request-button`.
- Admins can open request waves with `/open-requests`.
- Request waves can optionally be limited by type, including needs showcase, only demons, only platformers, only classic, non-demon variants, and Long/XL levels.
- Admins can schedule future request openings with `/open-requests when:<HH:MM> day:<optional>`.
- Sends a configurable opening announcement when a wave opens, with a default ping to the configured request role.
- Admins can list, edit, delete, refresh, or immediately open scheduled openings from the `/pending-openings` interactive panel.
- “Open now” asks for confirmation if requests are already open, and automatic scheduled openings will not silently replace an active wave.
- Admins can close request waves with `/close-requests`.
- Anyone can check the current state with `/requests-are`.
- Supports open, closed, limited-count, timed, and indefinite request waves.
- Counts a request only after the modal form is successfully submitted.
- Blocks duplicate users and duplicate level IDs inside the same wave.
- Warns staff when a submitted level ID has appeared in previous waves.
- Validates level IDs as 7 to 9 digits, validates showcase links as URLs, and checks level existence through GDBrowser plus the direct GD/Boomlings endpoint.
- Enforces the active wave type after GD validation and before the request counts toward the wave.
- Reuses one validation HTTP session, rate-limits validation attempts per user, and temporarily backs off providers that fail repeatedly.
- Auto-rejects confidently missing level IDs while surfacing uncertain validation warnings to reviewers.
- Warns reviewers when a level appears rated, when validation sources disagree, or when validation will refresh after the configured cache time.
- Automatically requires a showcase URL when validation detects a demon or platformer.
- Lets users edit their pending request with `/edit-request` or by pressing the request button until the wave closes, plus the configured grace period.
- Stores every request edit in an audit trail and exposes it with `/requests history`.
- Shows request age with Discord relative timestamps.
- Resets per-user and per-level duplicate tracking when a new wave starts.
- Checks configurable required roles before showing the request form.
- Supports the first-request `I will` / `I won't` choice flow with configurable roles.
- Sends staff review embeds to `level_requests.level_requested`.
- Configured reviewer roles, admins, and owners can choose `Send`, `Reject`, or `Other`.
- Staff can filter pending live-wave and weekly requests with `/requests pending`.
- Admins can run `/requests repair` to refresh the request button, rebuild wave summaries, recreate missing pending request messages, refresh stale validation warnings, and relock reviewed messages.
- Review actions verify the original request message and result channel before marking the request reviewed.
- Sends final result embeds to `level_requests.sent_channel` or `level_requests.rejected_channel`.
- Disables review buttons after a request is processed.
- Posts one live summary embed per closed wave in `level_requests.level_requested`, including requested, reviewed, sent, not sent, percentages, remaining reviews, and reviewer stats.
- Stores request state, request button message ID, wave count, submitted users, and submitted level IDs in SQLite so restarts do not wipe the wave.

### Help Menu And Staff Tickets
- DMs members a help dashboard with active tickets, weekly activity, live request state, recent help submissions, and cooldowns.
- Supports Back, Cancel, and Start Over controls during active DM help flows.
- Cleans up the previous DM help screen when members select a new option, press a flow button, cancel, or start over.
- Hides the current help screen from the menu so members are not offered the same page they are already viewing.
- Supports FAQ browsing and keyword search from the menu or by typing phrases like `faq request`.
- Suggests relevant FAQ entries before opening a staff ticket.
- Supports FAQ, punishment appeals, user reports, bot issue reports, weekly status checks, transcript requests, and staff contact tickets.
- Appeals, reports, and bot issue reports show a preview before submission, keep attachment links, receive tracked IDs, and can be checked later from My submissions.
- Staff can reply to a tracked appeal/report/bug log embed to relay a response back to the submitter by DM.
- Creates routed private ticket channels for the requester and staff, using the selected topic in the ticket name and opening message.
- Uses atomic ticket IDs to avoid duplicate ticket numbers during simultaneous ticket creation.
- Caches active ticket channels so normal server messages do not hit the database for ticket checks.
- Tracks ticket inactivity and prompts staff to close stale tickets.
- Tracks ticket statuses: Waiting for user, Waiting for staff, and Resolved.
- Keeps the ticket opening message status in sync when users or staff reply, when staff changes status, and before closure transcripts are saved.
- Saves transcripts before deleting tickets.
- Lets staff search saved ticket transcripts by user or ticket ID.
- Sends a configurable satisfaction prompt after a ticket closes.
- Lets staff approve or deny transcript requests.
- Posts appeals, reports, bot issues, transcript requests, ticket transcripts, and bot errors as structured staff-log embeds with safe mention behavior and audit logs.

### Forum And Sticky Automation
- Posts sticky reminder messages at the bottom of configured text channels.
- Sends first-message reminder embeds in configured forum channels.
- Supports tag-specific forum reminder embeds.
- Can enforce a required word in forum post title/body.
- Required-word matching supports `contains`, `whole_word`, and `regex` modes with basic text normalization.
- If the required word is missing, Avenue Guard DMs the thread owner and deletes the forum thread.
- Logs required-word forum deletions with the deleted thread and author.
- Admins can view or change the required word with `/forum required_word`.

### Configurable Auto-Responses
- Uses `responses.json` for message-triggered replies.
- Supports whole-message or contains matching.
- Supports plain messages or embeds.
- Supports channel filters and per-user cooldowns.
- Stops after the first matching rule.
- Blocks mass mentions from configured auto-responses and caps configured response length.

### Background Utilities
- `/bot dashboard` opens a button-driven admin dashboard with system health, request state, tracking state, icon rotation, config issues, and repair tips.
- `/bot impact` generates an owner-only community impact and forecast report, stores a database snapshot, and posts Markdown, CSV, trend CSV, breakdown CSV, and JSON exports to the configured impact/log channel.
- `/bot backup` creates a zipped database backup and posts it to the configured backup/log channel.
- `/bot storage` shows the active database path, whether it looks persistent, automatic backup status, and the latest backup record.
- `/bot health` shows database, background task, request, ticket, and weekly workflow status.
- `/bot config_check` validates configured channels, roles, request embed template variables, and `responses.json` rule shape/channel references.
- `/bot doctor` runs deeper permission diagnostics for channels, ticket category access, managed role hierarchy, and request-button state.
- `/requests pending` shows and filters pending live-wave and weekly request reviews.
- Optional rotating bot status with placeholders like `{members}`, `{online}`, `{week_msgs}`, `{week_top}`, `{open_tickets}`, and `{today_msgs}`.
- Optional server icon rotation from configured image URLs, with `disabled`, `linear`, and `random` modes.
- Optional daily server summary embeds with highlights, day-over-day movement, active members/channels, moderation signals, command health, voice/presence, and top channels/members/commands.
- Tracks daily messages, edits, deletes, reactions, joins, leaves, bans, boosts, voice minutes, command usage, and top channels/users.
- Includes a small keepalive HTTP server for hosted environments.

### Fun Commands
- `/dance` sends the configured GIF.
- `/rock-paper-scissors` runs a button-based game with per-user cooldown and optional streak reward role.
- `/gambling` runs a small slot animation with optional rare reward role.

## Main Slash Commands

Command options include Discord-side descriptions for confusing fields such as request limits, close timers, scheduled opening time, day of month, filters, message IDs, and target members.

- `/tracking top` shows the current weekly leaderboard.
- `/tracking me` shows your weekly count and rank.
- `/tracking reset` resets this week's tracking data.
- `/tracking force_dm` manually sends a weekly request DM, even to members excluded from normal tracking or during a disabled automatic reward week, and logs the outcome.
- `/tracking disable_reward` disables the automatic weekly request reward for the current tracking week.
- `/tracking enable_reward` re-enables the automatic weekly request reward for the current tracking week.
- `/bot health` shows a compact live health report.
- `/bot config_check` checks configured channels, roles, request template variables, and `responses.json`.
- `/bot doctor` runs deep permission diagnostics.
- `/bot impact` generates an owner-only community impact and forecast report with Markdown, CSV, trend CSV, breakdown CSV, and JSON exports.
- `/bot backup` creates a zipped database backup in the configured backup channel.
- `/bot storage` shows database storage and backup status.
- `/server_icon status` shows the server icon rotation mode, interval, current image, and configured URLs.
- `/server_icon mode mode:<random|linear|disabled>` changes automatic server icon rotation mode.
- `/server_icon add url:<url>`, `/server_icon replace number:<n> url:<url>`, and `/server_icon remove number:<n>` manage configured server icon URLs.
- `/server_icon set number:<n>` changes to a specific configured server icon immediately.
- `/server_icon next` changes to the next configured server icon immediately.
- `/requests pending scope:<optional> status:<optional> wave:<optional>` shows filtered live and weekly request reviews.
- `/requests history message_id:<optional> user_id:<optional> wave:<optional>` shows the edit audit trail for a live-wave request.
- `/requests repair` runs request-system recovery and message refresh tasks.
- `/refresh-request-button` refreshes or recreates the live request button.
- `/open-requests number:<optional> time:<optional> when:<optional> day:<optional> type:<optional> message:<optional>` opens or schedules a request wave.
- `/pending-openings action:<list|edit|delete> opening_id:<optional> message:<optional>` manages scheduled request openings and shows an interactive management panel by default.
- `/edit-request` lets a user edit their current pending live-wave request during the edit window.
- `/close-requests` closes the active request wave.
- `/requests-are` shows whether requests are currently open or closed.
- `/ticket close` closes the current ticket channel.
- `/ticket status status:<waiting_user|waiting_staff|resolved>` updates the current ticket status.
- `/ticket transcripts user:<optional> ticket_id:<optional>` searches saved ticket transcripts.
- `/forum required_word` views or changes the forum required word. Discord administrators only.
- `/resync` reloads config and response rules without restarting.
- `/restart` flushes buffered tracking/daily stats, then exits the bot so the host can restart it.
- `/dance`, `/rock-paper-scissors`, `/gambling` are public fun commands.

## Required-Word Forum Enforcement

Required-word checks live in `forum_first_message.entries[]` in `config.json`.

Example:

```json
{
  "forum_channel_id": "1104487618026143754",
  "required_word": "cubical",
  "required_word_match_mode": "contains",
  "missing_required_word_dm": "Your thread \"{thread_name}\" was removed because it did not include \"{required_word}\".",
  "required_word_delete_delay_seconds": 5,
  "templates": {
    "default": {
      "title": "Make sure your collab follows our format!",
      "description": "Your collab must have a name, theme, song, appeal, and purpose!",
      "color": "blurple"
    }
  }
}
```

`missing_required_word_dm` supports:
- `{required_word}`
- `{thread_name}`
- `{guild}`

To change the word without editing files, use:

```text
/forum required_word word:cubical
```

To use stricter matching:

```text
/forum required_word word:cubical match_mode:whole_word
```

If multiple forum channels are configured, provide the forum channel ID:

```text
/forum required_word word:cubical forum_channel_id:1104487618026143754
```

Use `off`, `disable`, `none`, or `clear` as the word to disable enforcement for that forum.

## Configuration Files

- `config.json` controls guild IDs, roles, channels, live request waves, weekly tracking, tickets, sticky messages, forum reminders, role DMs, fun rewards, help menu FAQ, server icon rotation, persistent database storage, automatic database backups, background summaries, and impact report exports.
- `responses.json` controls automatic message responses.
- The configured SQLite path stores persistent bot data such as live request waves, request submissions, request edit audits, GD validation cache, weekly counts, help submissions, tickets, cooldowns, transcript pointers, reminders, daily stats, impact snapshots, and database backup records.

### Database And Backup Config

Database storage lives under `database` in `config.json`.

- `path`: optional SQLite database path. Leave blank to auto-detect `/var/data/avenue-guard/bot.db` when a Render Persistent Disk is mounted, then fall back to `data/bot.db` if it is not writable.
- `AVENUE_GUARD_DB_PATH`: optional environment variable that overrides `database.path`.
- `backups.enabled`: enables scheduled zipped SQLite backups.
- `backups.channel_id`: where scheduled and manual `/bot backup` files are posted.
- `backups.interval_hours`: how often automatic backups are posted.
- `/bot storage` is the fastest way to verify whether the running bot is using the expected path.

### Impact Report Config

Impact reports live under `impact` in `config.json`.

- `report_channel_id`: where `/bot impact` posts the persistent Markdown, CSV, trend CSV, breakdown CSV, and JSON attachments. If empty, the bot falls back to `channels.general_logging_channel_id`.
- `allowed_user_ids`: the only users allowed to run `/bot impact`, `/bot backup`, and `/bot storage`. If empty, the bot falls back to admin/owner role checks.
- `/bot impact` stores the same report payload in `impact_snapshots`, so the bot keeps a database copy even after posting the files.
- The CSV exports are designed for direct import into Google Sheets.

### Server Icon Rotation Config

Server icon rotation lives under `background.server_icon_rotation` in `config.json`.

- `mode`: `disabled`, `linear`, or `random`.
- `interval_seconds`: time between automatic changes, with a minimum of 300 seconds.
- `urls`: direct image URLs used for the server icon.
- `current_index`, `current_url`, and `last_changed_ts`: saved state used by the rotation loop.
- `last_error` and `last_error_ts`: the most recent rotation failure shown in `/server_icon status`.

### Level Request Config

The live request system lives under `level_requests` in `config.json`.

Set these before opening requests:
- `request_channel`: where the public request button embed goes.
- `level_requested`: where submitted level request embeds go for review.
- `sent_channel`: where accepted/sent result embeds go.
- `rejected_channel`: where rejected and `Other` result embeds go.
- `required_role_ids`: roles allowed to request. Empty means everyone can request unless banned.
- `has_requested_role_id`: role used for users who already passed the first-request prompt.
- `request_banned_role_id`: role assigned by the `I will` choice and blocked from requesting.
- `reviewer_role_ids`: roles allowed to use request review controls and reviewer filters.
- `request_post_close_edit_minutes`: how long users may keep editing pending requests after the wave closes.
- `open_announcement`: controls the message sent when waves open. Blank `message` uses the default `<@&role>, requests have been opened for {condition_text}`.
- `level_validation.enabled`: enables GDBrowser plus GD/Boomlings existence/rating/showcase checks.
- `level_validation.cache_seconds`: how long validation warnings stay fresh before repair or the next submission refreshes them.
- `level_validation.per_user_cooldown_seconds`, `per_user_window_seconds`, and `per_user_max_checks`: protect validation from spam.
- `level_validation.provider_failure_threshold` and `provider_circuit_breaker_seconds`: pause failing validation providers briefly.
- `level_validation.auto_reject_missing`: blocks the modal when enabled sources confidently agree that the ID is missing.

The same section controls the request button text, all user-facing messages, request/review/result embed templates, wave summary embed, weekly request embeds, duplicate-history warnings, validation warnings, aging fields, edit-audit counters, and final-result colors.

Validated live requests also expose compact GD details for embeds:
- `{gd_info}`: clean preformatted difficulty, length, stars/status, and detected flags.
- `{gd_difficulty}`, `{gd_length}`, `{gd_stars}`, `{gd_rated}`.
- `{gd_demon}`, `{gd_platformer}`, `{gd_featured}`, `{gd_epic}`, `{gd_flags}`.
- `{gd_level_name}` and `{gd_creator}` when the validation provider returns them.

## Running The Bot

1. Install dependencies:

```bash
pip install -r requirements.txt
```

2. Set the bot token:

```bash
export DISCORD_TOKEN="your-token"
```

3. Start Avenue Guard:

```bash
python main.py
```

## Discord Intents And Permissions

Enable these in the Discord Developer Portal:
- Server Members Intent
- Message Content Intent

Useful bot permissions:
- Administrator, or at minimum:
- Manage Roles
- Manage Channels
- Manage Threads
- Manage Messages
- Read Message History
- Send Messages
- Embed Links
- Attach Files

## Persistence Notes

SQLite must live outside Render's clearable cache/project filesystem for true persistence. Mount a Render Persistent Disk at `/var/data`, or set `AVENUE_GUARD_DB_PATH` to another durable path. If no durable path is writable, the bot falls back to `data/bot.db` so it can start, but `/bot storage` will warn that data may be lost after cache clears. Automatic zipped backups also post to Discord as a second safety net.

## Local Testing

Use `TEST_CHECKLIST.md` for the full server-side test flow. It covers startup, moderation, live request waves, tracking, help sessions, ticket closure, transcript requests, sticky messages, forum reminders, required-word deletion, and fun commands.
