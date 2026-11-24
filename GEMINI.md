# Pinlyx extension

This extension connects Gemini CLI to a Pinlyx workspace through the remote MCP server at https://api.pinlyx.com/mcp. Pinlyx is a CRM with a unified inbox, deals and tasks, social publishing, outbound sequences and Google, Meta and TikTok ad reporting. One connection sees exactly one workspace: the one the sign-in or API key belongs to.

## First steps

- Call `crm_whoami` first when unsure what the connection can do. It shows the plan, the scopes and any tool the connection cannot call.
- The MCP server is available on every plan. The Free plan is read-only; writing (sending messages, creating or moving deals, scheduling posts, changing ads) needs Pro or above.

## Rules for using the tools

- Reading is safe. Anything that sends a message, publishes a post, spends money or removes something needs the person's explicit approval first. Show the exact text, recipient, budget or target before calling the tool.
- Social DMs, Telegram and X messages reach real people immediately. The server cannot send email.
- Moving a deal to won is not available over MCP (it books revenue); the person does that in the Pinlyx panel.
- Google Ads writes accept `validateOnly: true`; use it first. Removals need `confirm: true`. `crm_google_ads_undo` reverts a change by its `changeId`.
- Never mix currencies in one total. Reports state their window and freshness; quote them.
- Do not invent tool names, prices, dates or customer details. If a tool is missing or misleading, offer to report it with `crm_submit_feedback` (only with the person's agreement, and without passwords, keys or personal data).

## Useful starting tools

- Inbox: `crm_social_inbox_summary`, `crm_list_social_conversations`, `crm_list_recent_conversations`, `crm_search_email_threads`
- Pipeline: `crm_list_pipelines`, `crm_list_deals`, `crm_list_tasks`
- Contacts: `crm_search_contacts`, `crm_get_contact`, `crm_list_tags`
- Social: `crm_list_social_accounts`, `crm_list_social_posts`, `crm_schedule_social_post`
- Sequences: `crm_list_sequences`, `crm_get_sequence_status`, `crm_list_jobs`
- Ads: `crm_list_google_ads_accounts`, `crm_google_ads_summary`, `crm_meta_ads_summary`, `crm_tiktok_ads_summary`, `crm_ads_attribution_report`

## Narrowing the connection

The URL accepts `?toolsets=ads,contacts` and `?readonly=true` to limit which tools are shown, for example `https://api.pinlyx.com/mcp?toolsets=ads&readonly=true` for read-only ad reporting.

Docs: https://docs.pinlyx.com. Support: support@pinlyx.com.
