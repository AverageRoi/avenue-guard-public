# Avenue Guard

Avenue Guard is the Discord utility bot for GD Avenue. It handles moderation guardrails, live level request waves, weekly activity rewards, staff tickets, DM help flows, forum reminders, sticky notices, configurable auto-responses, server icon rotations, and a few community fun commands.

Now, you probably are wondering: **"Where is the code?"** Totally fair question. Due to recent events and for server security reasons, mainly to prevent the bot from being abused, the **code is on a private repository**.

Even though I cannot publish the source code here, this page explains what Avenue Guard does, how it is organized, and why it is so essential to the server. Think of this as a walkthrough of the bot as a real community system, I'm just not gonna expose any private implementation details, secrets, or logic that can be abused.

## What Avenue Guard Is

GD Avenue is a Geometry Dash Discord community, and even if it doesn't seem like it, it is a server that creates a lot of repeated operational work. You know, member support, staff logs, level requests, tickets transcripts, forum posts checks, and tracking of activity rewards. Avenue Guard was built to make those tasks smoother and semi or fully automated.

This bot isn't really a moderation tool. It is closer to a community operations assistant specific to our community. It connects several systems together: request management, activity tracking, support tickets, private help menus, forum reminders, moderation checks, analytics, backups, and small interactive features that make the server feel more alive... AND FUN.

The most important goal I had in mind is consistency. If a workflow starts, Avenue Guard remembers it, logs it, and keeps it recoverable. A level request should not disappear because the bot we depended on went offline (it can still happen, but at least it is 100% on us and not on another person). And you know, a server this big benefits from some QOL improvements like ticket transcripts, staff appeal reviews, weekly rewards... in a highly customizable way.  The bot is built around that kind of practical reliability.

## Level Requests

One of Avenue Guard's biggest jobs is managing level requests. Instead of relying on manual messages, the bot can open and close request waves. When requests are open, members press a request button, fill out a form, and submit their level information. When requests are closed, the bot simply tells them requests are unavailable.

Request waves can be opened with different limits. Staff can open requests for a certain number of successful submissions, for a certain amount of time, indefinitely, or through a scheduled opening. The bot only counts a request after the form is successfully submitted, so simply clicking a button does not waste a slot.

The request system also prevents common problems. A user can only submit once per wave, repeated level IDs are blocked during the same wave, and users can edit their request for a limited time if they notice a mistake. Reviewers see submitted levels in a staff queue, where they can mark each one as sent, rejected, or handled with other specific reasons.

Avenue Guard also checks Geometry Dash level information before a request reaches staff (Robtop's and Colon's APIs basically). It validates level IDs, checks whether a level appears to exist, detects whether a showcase may be required, and can warn reviewers when a level seems rated or suspicious. This does not replace human judgment (obviously), but it saves staff time and catches obvious issues early.

We don't need some features other request bots like Request Helper provide, but we do need scheduled openings and pending checks. This comes to say that we use this bot to customize both your experience as a user and our experience as staff. We can also check what works and what doesn't in our community, which is a great advantage apart from customizability.

We also have some telemitry, some wave statistics and tracking too.

## Weekly Activity Rewards

The bot tracks eligible weekly activity and can reward active members with a weekly request opportunity. After a lot of testing, the workflow more or less can be divided into three consecutive sections: skipping configured roles and channels, avoiding countability of obvious farming patterns, and storage of activity history towards weekly rewards.

When a member earns a weekly request, Avenue Guard can contact them privately, guide them through the claim process, and send the submitted level into the same review process used by normal request waves. It is external to request waves though.

The weekly system also tracks streaks, so members who stay active across multiple weeks can be recognized. This helps the server reward consistent participation instead of only sudden bursts of activity.

## Help, Reports, Appeals, And Tickets

Avenue Guard includes an in DMs help system. Members can open a private help menu, search FAQ entries, check basic status information, submit appeals, report users, report bot issues, request transcripts, or create a private staff ticket.

The help flow is designed to reduce confusion. Before opening a ticket, the bot can suggest relevant FAQ entries to avoid any unnecessary pings. If the member still needs help, the bot can create a private channel between the member and staff. Tickets have statuses such as waiting for user, waiting for staff, and resolved to help index and distribute for staff.

When a ticket closes, Avenue Guard saves a transcript before deleting the channel. It can also ask for satisfaction feedback, which helps measure whether support workflows are actually helping members. This makes tickets more accountable and easier to review later as well as any necessary proof of interactions.

## Forum And Message Organization

Big Discord servers can become messy quickly. Avenue Guard helps keep important instructions visible through sticky notices and forum reminders. In selected channels, the bot can repost a reminder so members do not miss important rules or formats.

For forum channels, Avenue Guard can send a first-message reminder when a new post is created. It can also enforce a required word or format marker in certain forum posts so that users are ensured to have read our channel guide. If a thread does not follow the required format, the bot can DM the author, delete the thread, and log what happened for staff.

This is especially useful for channels where structure matters, such as collabs. A lot of posts get removed daily, but the goal isn't really to punish people randomly but more to not drive staff crazy with all the posts that ignore our formats completely.

## Moderation checks

Avenue Guard includes small moderation automations for repeated issues. For example, it can watch specific channels where only certain users should post. If someone posts or reacts where they should not, the bot can remove the action and apply a configured restriction role (so is the case of our creator points channel).

It can also send configurable DMs when users receive certain roles like restrictions. This is very important for some bot interactions like requests, because members can benefit from understanding when/why they were restricted automatically.

These checks are intentionally really specific, so there aren't any or as little false positives as possible. They are not meant to replace staff judgment, but more to handle repetitive, predictable cases so staff can focus on situations that actually need human attention.

## Admin Tools And Reliability

Behind the scenes, Avenue Guard has several tools for keeping itself healthy. It can show an admin dashboard, check configuration issues, diagnose missing permissions, create database backups, restore from uploaded database copies, and generate impact reports.

This is important because our bot depends on many moving parts like channels, roles, permissions, messages, buttons, background tasks, and persistent data. If any of those drift, we can at least know what is missing and how to fix it.

The bot stores important workflow state persistently. That includes request waves, ticket data, transcripts, weekly tracking, help submissions, request reviews, validation cache, backups, restore history, and impact snapshots. So basically, the bot should not to forget the important parts of the server's operations.

Some of the reliability methods behind Avenue Guard include:

1. **Persistent storage for long workflows**: request waves, tickets, weekly rewards, scheduled openings, reviews, transcripts, backups, and impact snapshots are stored so they can survive restarts.
2. **Atomic counters and protected state updates**: ticket numbers, request counts, duplicate checks, and review transitions are handled carefully so two users or two reviewers cannot accidentally claim the same state at the same time.
3. **Persistent Discord components**: buttons and menus are registered again after restarts, so old request buttons, review buttons, ticket controls, and help-menu controls can still route to the correct workflow.
4. **Pre-action validation**: the bot checks roles, channels, permissions, level IDs, URLs, request state, and duplicate submissions before allowing important actions to continue.
5. **External validation with caching and fallbacks**: Geometry Dash level checks use external sources, cached results, cooldowns, and provider backoff so one failing service does not break the whole request system.
6. **Recovery and repair commands**: staff can refresh request buttons, rebuild summaries, relock reviewed requests, check storage, run diagnostics, create backups, and restore from uploaded database copies.
7. **Audit trails and logs**: request edits, reviewed levels, weekly reward events, ticket transcripts, forum deletions, admin actions, backups, restores, and impact reports all leave records.
8. **Safe backup/restore flow**: the bot can create zipped database backups, validate uploaded database copies, migrate restored data, and log the recovery.
9. **Rate limits and cooldowns**: activity tracking, help flows, validation checks, fun commands, and auto-responses use limits to reduce spam and accidental overload.
10. **Config checks and permission diagnostics**: the bot can scan for missing roles, missing channels, bad template variables, broken permissions, and unhealthy background tasks before they become bigger problems.

## Impact Reports

Avenue Guard through multiple analytics also generates a combined community impact report, which we may make public soon if requested so members can learn more about what our bot detects and can interact with.

## Fun

Not everything in the bot is automatization and QOL. Avenue Guard also includes small community features, like fun commands and server icon rotation. These features originated from community polls and weren't a planned part of the system, but they help the bot feel like part of the server rather than just a background rotating mechanical machine.

That balance is part of the idea behind Avenue Guard. It should be useful, dependable, and structured, but it should still feel like it belongs in GD Avenue.

The server icon changes following different themes in a 5 min rotation period, we also add server art made by members to the cicles! 

## How the bot is built (out for the curious)

The public version of this repository does not include the source code, but the architecture can still be explained pretty thoughrouly.

Avenue Guard is built as a modular Discord bot. Each major feature area is separated into its own internal component: requests, tracking, help and tickets, forum reminders, moderation, background summaries, admin tools, and shared utilities. Those components communicate with a persistent database and a central configuration system.

Configuration controls server specific behavior such as which channels are used, which roles are allowed to do certain actions, how embeds are worded, where logs are sent, and how request waves behave. This makes the bot adaptable without hardcoding every server decision directly into the logic.

Persistent storage is used because many workflows last longer than a single bot session. All of the information SHOULD survive a restart.

Real server architecture:

| Area | What it contains | Purpose |
|---|---|---|
| `main.py` | Bot startup, configuration loading, database connection, cog loading, keepalive startup, persistent view registration | Starts the bot and wires every major system together |
| `cogs/` | Feature modules such as requests, tracking, help/tickets, moderation checks, sticky/forum automation, background jobs, commands, and message responses | Keeps each major bot workflow separated instead of putting everything in one giant file |
| `utils/` | Shared helpers for config, database access, checks, safe mentions, time handling, transcripts, validation, persistent views, server icons, and error logging | Holds reusable logic used by multiple parts of the bot |
| `config.json` | Server-specific settings for roles, channels, request behavior, help text, embeds, backups, summaries, and permissions | Lets the bot be customized without changing source code every time |
| `responses.json` | Configurable message-triggered auto-responses | Controls simple automatic replies outside the main Python logic |
| `data/` | Local database fallback location | Used when no persistent production path is available |
| `docs/` | Private manuals and generated documentation | Used for understanding and maintaining the bot privately |
| `scripts/` | Documentation and maintenance scripts | Helps generate internal documentation and supporting files |
| `requirements.txt` | Runtime dependencies | Defines the packages needed to run the bot |
| `TEST_CHECKLIST.md` | Manual testing checklist | Helps verify important Discord workflows after changes |

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

## Why The Code Is Private

The code is private for security and server safety reasons. A bot like this contains stuff that could be abused if copied or studied in too much detail. Some features involve moderation behavior, staff review flows, server specific assumptions, validation rules, internal recovery paths, and configuration patterns that should not be exposed publicly.

Keeping the source private protects GD Avenue while still allowing the project to be explained honestly. This repository exists as a public overview of what the bot does, what problems it solves, and how it supports the server.

## Project Summary

Avenue Guard is a custom operations bot for GD Avenue. It helps the server manage level requests, reward active members, support users privately, keep forums organized, protect sensitive channels, generate useful logs, and measure community impact over time.

The code is private, but the project itself is a complete (pretty big) Discord bot: persistent workflows, staff tools, scheduled automation, review queues, support systems, backups, analytics, and community interactions all working together for one server.
