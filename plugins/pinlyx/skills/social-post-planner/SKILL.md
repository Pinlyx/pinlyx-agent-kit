---
name: social-post-planner
description: Plan, schedule and review social posts through Pinlyx (Instagram, Facebook, X, LinkedIn, TikTok, YouTube, Threads and more). Use when the person wants a weekly content calendar, wants to schedule or reschedule posts, or asks what is going out this week. Schedules only after approval and never publishes immediately without an explicit request.
---

# Social post planning

Works against the Pinlyx MCP server. Tool names are shown without the client prefix.

## 1. Know the accounts and the current queue

- `crm_whoami` for the plan. Scheduling is a write: on the Free plan, produce the calendar and the post copy but do not attempt to schedule.
- `crm_list_social_accounts` (optional `platform`, `includeInactive`) gives the connected accounts, handles and posting limits. The `accountId` to post from comes from this list.
- `crm_list_social_posts` (`status`: pending, processing, published, failed, cancelled; `platform`; `fromDate`, `toDate`; `limit`) shows what is already scheduled or published, with published URL and any failure reason.
- `crm_social_post_stats` (`days`) shows publishing volume and outcome per network, plus the last publish time. Use it to spot a network that has gone quiet or has failures.

## 2. Plan

Propose a calendar (date, time, account, format, copy, media) that fills gaps and avoids clashing with posts already pending. Respect the per-platform rules the server enforces:

- X is capped at 280 characters.
- Instagram needs media.
- TikTok and YouTube need a video.
- `mediaUrls` must be publicly reachable and stay reachable until the post publishes.
- One post is created per target account, so two platforms means two posts.
- YouTube: the description is the post content and the title comes only from `youtube.title` (up to 100 characters). Instagram Reels and YouTube Shorts accept the optional `instagram` and `youtube` option objects (for example `firstComment`, cover options, tags). Unknown option names are rejected.

Ask the person for the time zone if it is not known and pass it as `timeZone` (an IANA name such as `Europe/Istanbul`) when `scheduledAt` has no offset.

## 3. Approval gate (required before scheduling)

Show the full plan as a table: platform, account, scheduled time with time zone, exact text, media. Schedule only the posts the person approves.

- `crm_schedule_social_post` (`platforms`, `content`, `accountIds`, `scheduledAt`, `mediaUrls`, `timeZone`). `scheduledAt` is required unless `publishNow` is true.
- Never set `publishNow: true` unless the person explicitly asks to publish right now. It goes to a real audience immediately.
- Do not invent claims, statistics, prices or links. Use only what the person provided.

## 4. Changes and cancellations

Edit a post that has not gone out with `crm_update_social_post` (`postId`, and any of `content`, `scheduledAt`, `mediaUrls`, `timeZone`); only pending posts can be edited. Cancel with `crm_cancel_social_post` (`postId`); a post that already went out cannot be withdrawn from here. Show the before and after (or the post being cancelled) and get approval first. `crm_get_social_post` (`postId`) fetches the full current state.

## 5. Follow up

After scheduling, run `crm_list_social_posts` with `status: "pending"` for the window and show the person the resulting queue. A few days later, `status: "failed"` and `crm_social_post_stats` catch problems early.
