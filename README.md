# Avenue Guard

Avenue Guard is the Discord utility bot for GD Avenue. It handles moderation guardrails, live level request waves, weekly activity rewards, staff tickets, DM help flows, forum reminders, sticky notices, configurable auto-responses, server icon rotations, and a few community fun commands.

Now, you are probably wondering: **"Where is the code?"** Totally fair question. Due to recent events and for server security reasons, mainly to prevent the bot from being abused, the **code is in a private repository**.

Even though I cannot publish the source code here, this page explains what Avenue Guard does, how it is organized, and why it is so essential to the server. Think of this as a walkthrough of the bot as a real community system. I'm just not going to expose private implementation details, secrets, or logic that could be abused.

## What Avenue Guard Is

GD Avenue is a Geometry Dash Discord community, and even if it does not seem like it, running a server of this size generates a lot of recurring operational work: member support, staff logs, level requests, ticket transcripts, forum post checks, and activity reward tracking. Avenue Guard was built to make those tasks smoother and partially or fully automated.

This bot is not really a moderation tool. It is closer to a community operations assistant built specifically for GD Avenue. It connects several systems: request management, activity tracking, support tickets, private help menus, forum reminders, moderation checks, analytics, backups, and small interactive features that make the server feel more alive... and fun.

The most important goal I had in mind was consistency. If a workflow starts, Avenue Guard remembers it, logs it, and preserves enough state to recover it after an interruption. A level request should not disappear because a third-party bot goes offline. That can still happen with our own infrastructure, but at least we control the failure point instead of depending on someone else.

A server this big also benefits from QOL improvements such as ticket transcripts, staff appeal reviews, and weekly rewards that can be adapted to the way our community actually works. The bot is built around that kind of practical reliability.

## Level Requests

One of Avenue Guard's biggest jobs is managing level requests. Instead of relying on manual messages, the bot can open and close request waves. When requests are open, members press a request button, fill out a form, and submit their level information. When requests are closed, the bot simply tells them that requests are unavailable.

Request waves can be opened with different limits. Staff can open requests for a certain number of successful submissions, for a certain amount of time, indefinitely, or through a scheduled opening. The bot only counts a request after the form is successfully submitted, so simply clicking a button does not waste a slot.

The request system also prevents common problems. A user can only submit once per wave, repeated level IDs are blocked during the same wave, and users can edit their request for a limited time if they notice a mistake. Reviewers see submitted levels in a staff queue, where they can mark each request as sent, rejected, or closed under another specific review outcome.

Avenue Guard also checks Geometry Dash level information before a request reaches staff, using RobTop's and Colon's APIs. It validates level IDs, checks whether a level appears to exist, detects whether a showcase may be required, and can warn reviewers when level metadata looks unusual or requires closer review. This does not replace human judgment, obviously, but it saves staff time and catches obvious issues early.

We do not need every feature that other request bots such as Request Helper provide, but we do need scheduled openings and pending checks. That is one of the reasons we use our own bot: it lets us tailor the workflow for both members and staff.

It also lets us see what works and what does not in our community. We track telemetry, wave statistics, and request behavior, which gives us more control over how the system evolves.

## Weekly Activity Rewards

The bot tracks eligible weekly activity and can reward active members with a weekly request opportunity. After a lot of testing, the workflow can roughly be divided into three consecutive stages: excluding configured roles and channels, filtering obvious activity-farming patterns, and storing activity history for weekly rewards.

When a member earns a weekly request, Avenue Guard can contact them privately, guide them through the claim process, and send the submitted level into the same review process used by normal request waves. The weekly reward workflow remains separate from normal request waves, though.

The weekly system also tracks streaks, so members who stay active across multiple weeks can be recognized. This helps the server reward consistent participation instead of only sudden bursts of activity.

## Help, Reports, Appeals, and Tickets

Avenue Guard includes a DM-based help system. Members can open a private help menu, search FAQ entries, check basic status information, submit appeals, report users, report bot issues, request transcripts, or create a private staff ticket.

The help flow is designed to reduce confusion. Before opening a ticket, the bot can suggest relevant FAQ entries to avoid unnecessary pings. If the member still needs help, the bot can create a private channel between the member and staff.

Tickets have statuses such as waiting for the user, waiting for staff, and resolved. These states help staff triage, organize, and assign tickets.

When a ticket closes, Avenue Guard saves a transcript before deleting the channel. It can also ask for satisfaction feedback, which helps us measure whether support workflows are actually helping members. Transcripts make tickets easier to review later and preserve a record of the interaction when evidence is needed.

## Forum and Message Organization

Big Discord servers can become messy quickly. Avenue Guard helps keep important instructions visible through sticky notices and forum reminders. In selected channels, the bot can repost a reminder so members do not miss important rules or formats.

For forum channels, Avenue Guard can send a first-message reminder when a new post is created. It can also enforce a required word or format marker in certain forum posts to help ensure that users have read the channel guide. If a thread does not follow the required format, the bot can DM the author, delete the thread, and log what happened for staff.

This is especially useful in channels where structure matters, such as collabs. A lot of posts get removed daily, but the goal is not punitive. The goal is to reduce the repetitive workload created by posts that completely ignore required formats.

## Moderation Checks

Avenue Guard includes small moderation automations for repeated issues. For example, it can watch specific channels where only certain users should post. If someone posts or reacts where they should not, the bot can remove the action and apply a configured restriction role, as in our creator points channel.

It can also send configurable DMs when users receive certain roles, such as restriction roles. This is especially important for bot interactions such as requests, so members understand why and when a restriction was applied automatically.

These checks are deliberately narrow in scope so there are as few false positives as possible. They are not meant to replace staff judgment, but rather to handle repetitive, predictable cases so staff can focus on situations that actually need human attention.

## Admin Tools and Reliability

Behind the scenes, Avenue Guard has several tools for keeping itself healthy. It can show an admin dashboard, check configuration issues, diagnose missing permissions, create database backups, restore local database copies when using local storage, and generate impact reports.

This is important because the bot depends on many moving parts: channels, roles, permissions, messages, buttons, background tasks, and persistent data. If the live Discord configuration drifts from the bot's expected state, we can at least identify what is missing and how to fix it.

The bot stores important workflow state persistently. In production, Avenue Guard uses **Turso/libSQL** as its durable database layer. Turso is SQLite-compatible, which means the bot can keep the simplicity of SQLite while syncing important state to remote storage instead of relying on a host's temporary filesystem. This matters a lot on platforms where clearing cache or restarting a service could otherwise wipe local files.

The implementation uses a local embedded replica for fast reads and writes, then syncs that replica with the Turso database. In practice, this lets the bot behave like a normal SQLite bot during development while still providing durable persistence for real server workflows.

The database wrapper also includes startup checks, token validation, retry handling for temporary Turso/libSQL sync errors, and safer fallback behavior if a configured storage path is not writable.

That persistent storage includes request waves, ticket data, transcripts, weekly tracking, help submissions, request reviews, validation cache, backups, restore history, and impact snapshots. In other words, the bot preserves the operational state that matters across restarts and infrastructure failures.

Some of the reliability methods behind Avenue Guard include:

1. **Turso/libSQL persistent storage for long workflows**: request waves, tickets, weekly rewards, scheduled openings, reviews, transcripts, backups, and impact snapshots are stored in SQLite-compatible remote storage so they can survive restarts and host cache clears.
2. **Atomic counters and protected state updates**: ticket numbers, request counts, duplicate checks, and review transitions are handled carefully so two users or two reviewers cannot accidentally claim the same state at the same time.
3. **Persistent Discord components**: buttons and menus are registered again after restarts, so old request buttons, review buttons, ticket controls, and help-menu controls can still route to the correct workflow.
4. **Pre-action validation**: the bot checks roles, channels, permissions, level IDs, URLs, request state, and duplicate submissions before allowing important actions to continue.
5. **External validation with caching and fallbacks**: Geometry Dash level checks use external sources, cached results, cooldowns, and provider backoff so one failing service does not break the whole request system.
6. **Recovery and repair commands**: staff can refresh request buttons, rebuild summaries, relock reviewed requests, check storage, run diagnostics, create backups, and restore local uploaded database copies when running on local SQLite.
7. **Audit trails and logs**: request edits, reviewed levels, weekly reward events, ticket transcripts, forum deletions, admin actions, backups, restores, and impact reports all leave records.
8. **Safe backup flow**: the bot can create zipped database backups, validate local uploaded database copies when appropriate, migrate restored data, and log the recovery. Turso remains the main production source of truth, while backups act as an extra safety layer.
9. **Rate limits and cooldowns**: activity tracking, help flows, validation checks, fun commands, and auto-responses use limits to reduce spam and accidental overload.
10. **Config checks and permission diagnostics**: the bot can scan for missing roles, missing channels, bad template variables, broken permissions, and unhealthy background tasks before they become bigger problems.

## Impact Reports

Avenue Guard also uses its analytics data to generate a combined community impact report. We may make this public later so members can better understand the patterns the bot tracks and how the systems they interact with are being used.

## Fun

Not everything in Avenue Guard is about automation and QOL. Avenue Guard also includes small community features, such as fun commands and server icon rotation. These features originated from community polls and were not a planned part of the system, but they help the bot feel like part of the server rather than just another piece of background infrastructure.

That balance is part of the idea behind Avenue Guard. It should be useful, dependable, and structured, but it should still feel like it belongs in GD Avenue.

The server icon rotates through different themes every five minutes, and we also include member-created server art in the rotation!

## How the Bot Is Built

The public version of this repository does not include the source code, but the architecture can still be described in some detail.

Avenue Guard is built as a modular Discord bot. Each major feature area is separated into its own internal component: requests, tracking, help and tickets, forum reminders, moderation, background summaries, admin tools, and shared utilities. Those components communicate with a persistent database and a central configuration system.

Configuration controls server-specific behavior such as which channels are used, which roles are allowed to perform certain actions, how embeds are worded, where logs are sent, and how request waves behave. This makes the bot adaptable without hardcoding every server decision directly into the logic.

Persistent storage is used because many workflows last longer than a single bot session. Important state needs to survive a restart.

Real server architecture:

| Area                | What it contains                                                                                                                                             | Purpose                                                                                 |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| `main.py`           | Bot startup, configuration loading, database connection, cog loading, keepalive startup, persistent view registration                                        | Starts the bot and wires every major system together                                    |
| `cogs/`             | Feature modules such as requests, tracking, help/tickets, moderation checks, sticky/forum automation, background jobs, commands, and message responses       | Keeps each major bot workflow separated instead of putting everything in one giant file |
| `utils/`            | Shared helpers for config, database access, checks, safe mentions, time handling, transcripts, validation, persistent views, server icons, and error logging | Holds reusable logic used by multiple parts of the bot                                  |
| `config.json`       | Server-specific settings for roles, channels, request behavior, help text, embeds, backups, summaries, and permissions                                       | Lets the bot be customized without changing source code every time                      |
| `responses.json`    | Configurable message-triggered auto-responses                                                                                                                | Controls simple automatic replies outside the main Python logic                         |
| `data/`             | Local database fallback or Turso replica location                                                                                                            | Keeps development simple and lets Turso sync through a local embedded replica           |
| `docs/`             | Private manuals and generated documentation                                                                                                                  | Used for understanding and maintaining the bot privately                                |
| `scripts/`          | Documentation and maintenance scripts                                                                                                                        | Helps generate internal documentation and supporting files                              |
| `requirements.txt`  | Runtime dependencies                                                                                                                                         | Defines the packages needed to run the bot                                              |
| `TEST_CHECKLIST.md` | Manual testing checklist                                                                                                                                     | Helps verify important Discord workflows after changes                                  |

In a simplified scheme, the flow looks like this:

```text
Discord events / slash commands / buttons
        |
        v
main.py loads the bot and routes work into cogs
        |
        v
cogs handle feature-specific workflows
        |
        v
utils provide shared helpers and database access
        |
        v
persistent storage remembers long-running state
        |
        v
Discord receives embeds, logs, tickets, reviews, DMs, and summaries
```

## Why the Code Is Private

The code is private for security and server safety reasons. A bot like this contains implementation details that could be abused if copied or studied too closely. Some features involve moderation behavior, staff review flows, server-specific assumptions, validation rules, internal recovery paths, and configuration patterns that should not be exposed publicly.

Keeping the source private protects GD Avenue while still allowing the project to be explained honestly. This repository exists as a public overview of what the bot does, what problems it solves, and how it supports the server.

## Project Summary

Avenue Guard is a custom operations bot for GD Avenue. It helps the server manage level requests, reward active members, support users privately, keep forums organized, protect sensitive channels, generate useful logs, and measure community impact over time.

The code is private, but the project itself is a complete and fairly large Discord bot: persistent workflows, staff tools, scheduled automation, review queues, support systems, backups, analytics, and community interactions all working together for one server.
