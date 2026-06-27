# Avenue Guard

Avenue Guard is the Discord utility bot for GD Avenue. It handles moderation guardrails, live level request waves, weekly activity rewards, staff tickets, DM help flows, forum reminders, sticky notices, configurable auto-responses, server icon rotations, and a few community fun commands.

Now, you probably are wondering: **"Where is the code?"** Totally fair question. Due to recent events and for server security reasons, mainly to prevent the bot from being abused, the **code is on a private repository**.

Even though I cannot publish the source code here, this page explains what Avenue Guard does, how it is organized, and why it matters to the server. Think of this as a public walkthrough of the bot as a real community system, without exposing private implementation details, secrets, role IDs, channel IDs, or abuse-sensitive logic.

## What Avenue Guard Is

GD Avenue is an active Discord community, and a server that size creates a lot of repeated operational work. Members need help, staff need logs, requests need structure, tickets need transcripts, forum posts need formatting, and activity rewards need to be tracked fairly. Avenue Guard was built to make those recurring workflows smoother and more reliable.

The bot is not just a single-purpose moderation tool. It is closer to a community operations assistant. It connects several systems together: request management, activity tracking, support tickets, private help menus, forum reminders, moderation guardrails, analytics, backups, and small interactive features that make the server feel more alive.

The most important design goal is consistency. If a workflow starts, Avenue Guard tries to remember it, log it, and keep it recoverable. A level request should not disappear because the bot restarted. A ticket should not close without a transcript. A staff review should not be processed twice. A weekly reward should have a visible history. The bot is built around that kind of practical reliability.

## Level Requests

One of Avenue Guard's biggest jobs is managing level requests. Instead of relying on messy manual messages, the bot can open and close request waves. When requests are open, members press a request button, fill out a form, and submit their level information. When requests are closed, the bot simply tells them requests are unavailable.

Request waves can be opened with different limits. Staff can open requests for a certain number of successful submissions, for a certain amount of time, indefinitely, or through a scheduled opening. The bot only counts a request after the form is successfully submitted, so simply clicking a button does not waste a slot.

The request system also prevents common problems. A user can only submit once per wave, repeated level IDs are blocked during the same wave, and users can edit their request for a limited time if they notice a mistake. Reviewers see submitted levels in a staff queue, where they can mark each one as sent, rejected, or handled with another specific reason.

Avenue Guard also checks Geometry Dash level information before a request reaches staff. It validates level IDs, checks whether a level appears to exist, detects whether a showcase may be required, and can warn reviewers when a level seems rated or suspicious. This does not replace human judgment, but it saves staff time and catches obvious issues early.

## Weekly Activity Rewards

The bot tracks eligible weekly activity and can reward active members with a weekly request opportunity. It does this carefully: it skips configured roles and channels, avoids counting obvious low-effort farming patterns, and stores activity history so weekly rewards are not based only on memory or screenshots.

When a member earns a weekly request, Avenue Guard can contact them privately, guide them through the claim process, and send the submitted level into the same review workflow used by normal request waves. That means staff do not need to learn two separate review systems.

The weekly system also tracks streaks, so members who stay active across multiple weeks can be recognized. This helps the server reward consistent participation instead of only one-time bursts of activity.

## Help, Reports, Appeals, And Tickets

Avenue Guard includes an in-DMs help system. Members can open a private help menu, search FAQ entries, check basic status information, submit appeals, report users, report bot issues, request transcripts, or create a private staff ticket.

The help flow is designed to reduce confusion. Before opening a ticket, the bot can suggest relevant FAQ entries. If the member still needs help, the bot can create a private channel between the member and staff. Tickets have statuses such as waiting for user, waiting for staff, and resolved, so staff can understand what is happening at a glance.

When a ticket closes, Avenue Guard saves a transcript before deleting the channel. It can also ask for satisfaction feedback, which helps measure whether support workflows are actually helping members. This makes tickets more accountable and easier to review later.

## Forum And Message Organization

Busy Discord servers can become messy quickly. Avenue Guard helps keep important instructions visible through sticky notices and forum reminders. In selected channels, the bot can repost a reminder so members do not miss important rules or formats.

For forum channels, Avenue Guard can send a first-message reminder when a new post is created. It can also enforce a required word or format marker in certain forum posts. If a thread does not follow the required format, the bot can DM the author, delete the thread, and log what happened for staff.

This is especially useful for channels where structure matters, such as collabs, feedback, or request-related spaces. The goal is not to punish people randomly; it is to keep high-traffic areas understandable and useful.

## Moderation Guardrails

Avenue Guard includes small moderation automations for repeated issues. For example, it can watch specific channels where only certain users should post. If someone posts or reacts where they should not, the bot can remove the action and apply a configured restriction role.

It can also send configurable DMs when users receive certain roles. That matters because moderation actions are clearer when users understand what changed and what to do next.

These guardrails are intentionally narrow. They are not meant to replace staff judgment. They handle repetitive, predictable cases so staff can focus on situations that actually need human attention.

## Admin Tools And Reliability

Behind the scenes, Avenue Guard has several tools for keeping itself healthy. It can show an admin dashboard, check configuration issues, diagnose missing permissions, create database backups, restore from uploaded database copies, and generate impact reports.

This is important because Discord bots depend on many moving parts: channels, roles, permissions, messages, buttons, background tasks, and persistent data. If any of those drift, staff need a way to understand what broke and how to recover.

The bot stores important workflow state persistently. That includes request waves, ticket data, transcripts, weekly tracking, help submissions, request reviews, validation cache, backups, restore history, and impact snapshots. In normal language: the bot tries very hard not to forget the important parts of the server's operations.

## Community Impact Reports

Avenue Guard can generate reports that summarize the bot's community impact. These reports can include activity trends, support volume, request throughput, ticket history, review progress, command usage, and other operational signals.

The point is not just to say "the bot is useful." The point is to make its usefulness measurable. For a community project, that matters. It shows how many workflows the bot has helped coordinate and gives staff a clearer picture of where the server is active, where help is needed, and how the community is changing over time.

## Server Personality

Not everything in the bot is serious. Avenue Guard also includes small community features, like fun commands and server icon rotation. These features are not the core of the system, but they help the bot feel like part of the server rather than just a silent machine that deletes messages and posts logs.

That balance is part of the idea behind Avenue Guard. It should be useful, dependable, and structured, but it should still feel like it belongs in GD Avenue.

## How The Bot Is Built, Without Revealing The Private Code

The public version of this repository does not include the source code, but the architecture can still be explained safely.

Avenue Guard is built as a modular Discord bot. Each major feature area is separated into its own internal component: requests, tracking, help and tickets, forum reminders, moderation, background summaries, admin tools, and shared utilities. Those components communicate with a persistent database and a central configuration system.

Configuration controls server-specific behavior such as which channels are used, which roles are allowed to do certain actions, how embeds are worded, where logs are sent, and how request waves behave. This makes the bot adaptable without hardcoding every server decision directly into the logic.

Persistent storage is used because many workflows last longer than a single bot session. A scheduled request opening, an active ticket, an unreviewed level, a weekly reward claim, or an impact report should survive a restart. Avenue Guard is designed around that expectation.

## Why The Code Is Private

The code is private for security and server-safety reasons. A bot like this contains workflows that could be abused if copied or studied in too much detail. Some features involve moderation behavior, staff-only review flows, server-specific assumptions, validation rules, internal recovery paths, and configuration patterns that should not be exposed publicly.

Keeping the source private protects GD Avenue while still allowing the project to be explained honestly. This repository exists as a public overview of what the bot does, what problems it solves, and how it supports the server.

## Project Summary

Avenue Guard is a custom-built operations bot for GD Avenue. It helps the server manage level requests, reward active members, support users privately, keep forums organized, protect sensitive channels, generate useful logs, and measure community impact over time.

The code is private, but the project itself represents a complete production-style Discord bot: persistent workflows, staff tools, scheduled automation, review queues, support systems, backups, analytics, and community-facing interactions all working together for one server.

In short: Avenue Guard is the quiet infrastructure behind a lot of GD Avenue's day-to-day organization.
