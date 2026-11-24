---
name: "pinlyx"
displayName: "Pinlyx CRM, Social and Ads"
description: "Work with a Pinlyx workspace: unified inbox, contacts, deals and tasks, social post scheduling, outbound sequences and Google, Meta and TikTok ad reporting."
version: "1.0.0"
keywords: ["pinlyx", "crm", "inbox", "contacts", "deals", "pipeline", "social media", "scheduling", "google ads", "meta ads", "tiktok ads", "sequences"]
author: "Pinlyx"
---

# Pinlyx

Pinlyx is a CRM with a unified inbox (social DMs, Telegram, X, email), deals and tasks, social publishing, outbound sequences and ad account reporting for Google Ads, Meta and TikTok. This power connects Kiro to the Pinlyx remote MCP server. One connection sees exactly one workspace: the one the sign-in or API key belongs to.

## Onboarding

1. The server uses OAuth with dynamic client registration. Kiro opens a browser sign-in the first time a tool is used. Sign in to Pinlyx and approve access.
2. Alternatively use an API key from the Pinlyx panel. Add this to the `pinlyx` entry in `mcp.json` and approve the variable in Kiro's "Mcp Approved Env Vars" setting:

```json
"headers": { "X-Api-Key": "${PINLYX_API_KEY}" }
```

3. Call `crm_whoami` once. It shows the plan, the scopes of this connection and any tool it cannot call.

The MCP server is available on every plan. Free is read-only. Writing needs Pro or above.

## Available MCP server

### pinlyx

**Connection:** remote HTTPS, Streamable HTTP, `https://api.pinlyx.com/mcp`

To show fewer tools, add query parameters to the URL: `?toolsets=ads,contacts` limits the groups and `?readonly=true` hides every tool that writes. Group names include contacts, conversations, deals, tasks, email, social, posts, sequences, pipelines, jobs, ads, finance, analytics and webhooks.

Main tools by area:

- Inbox: `crm_social_inbox_summary`, `crm_list_social_conversations`, `crm_get_social_conversation`, `crm_list_recent_conversations`, `crm_get_conversation`, `crm_search_email_threads`, `crm_get_email_thread`
- Contacts and tags: `crm_search_contacts`, `crm_get_contact`, `crm_get_contact_activity`, `crm_list_tags`, `crm_tag_contact`, `crm_set_lead_score`
- Pipeline: `crm_list_pipelines`, `crm_list_deals`, `crm_get_deal`, `crm_update_deal_stage`, `crm_list_tasks`, `crm_create_task`
- Social publishing: `crm_list_social_accounts`, `crm_list_social_posts`, `crm_schedule_social_post`, `crm_update_social_post`, `crm_cancel_social_post`
- Sequences and jobs: `crm_list_sequences`, `crm_get_sequence_status`, `crm_pause_sequence`, `crm_resume_sequence`, `crm_list_jobs`
- Ads: `crm_list_google_ads_accounts`, `crm_google_ads_summary`, `crm_meta_ads_summary`, `crm_tiktok_ads_summary`, `crm_ads_attribution_report`

## Rules

- Reading is safe. Anything that sends a message, publishes a post, spends money or removes something needs the person's explicit approval first. Show the exact text, recipient, budget or target before calling the tool.
- Social DMs, Telegram and X messages reach real people immediately. The server cannot send email.
- Moving a deal to won is not available over MCP (it books revenue); the person does that in the Pinlyx panel.
- Google Ads writes accept `validateOnly: true`; use it first. Removals need `confirm: true`. `crm_google_ads_undo` reverts a change by its `changeId`.
- Never add different currencies together. Reports state their window and freshness; quote them.
- Do not invent tool names, prices, dates or customer details.

## Workflows

### Inbox triage
1. `crm_social_inbox_summary`, then `crm_list_social_conversations` with `unreadOnly: true`, `crm_list_recent_conversations` and `crm_search_email_threads`.
2. Read the top threads (`crm_get_social_conversation`, `crm_get_conversation`, `crm_get_email_thread`) and rank them.
3. Draft replies. Send with `crm_send_social_message`, `crm_send_telegram_message` or `crm_send_twitter_dm` only after approval of the exact text.

### Pipeline review
1. `crm_list_pipelines`, `crm_list_deals` per open stage, `crm_list_tasks` with `overdue: true`.
2. Summarize by stage and flag deals with no task or no recent activity.
3. Apply changes (`crm_update_deal_stage`, `crm_create_task`) only after approval.

### Weekly ads report
1. List accounts with `crm_list_google_ads_accounts`, `crm_list_meta_ad_accounts`, `crm_list_tiktok_ad_accounts`.
2. Pull `LAST_7_DAYS` summaries with a previous-period comparison, plus `crm_google_ads_budget_pacing` and `crm_google_ads_account_alerts`.
3. Join spend with CRM results using `crm_ads_attribution_report`. Budget or status changes need approval.

### Social post planning
1. `crm_list_social_accounts` and `crm_list_social_posts` for the current queue.
2. Propose a calendar. Schedule approved posts with `crm_schedule_social_post`. Never use `publishNow` unless the person asks for it.

## Troubleshooting

- "requires scope" or a plan message: call `crm_whoami` and tell the person which scope or plan is missing.
- Structured errors look like `{ error: { code, message, hint, retryable, retryAfterSec } }`. Follow the hint and retry only when `retryable` is true.

Docs: https://docs.pinlyx.com. Support: support@pinlyx.com.
