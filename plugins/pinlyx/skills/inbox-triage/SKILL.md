---
name: inbox-triage
description: "Prioritize the Pinlyx inbox (social DMs, Telegram/X conversations and email threads) and draft replies. Use when the person asks who needs a reply, wants a morning inbox sweep, or asks for a draft answer to a conversation. Drafts first, sends only after explicit approval."
---

# Inbox triage and reply drafts

Works against the Pinlyx MCP server. Tool names below are shown without the client prefix.

## 1. Check the connection

Call `crm_whoami` once. It shows the plan and the scopes this connection holds. If a write tool is refused (the Free plan is read-only), still deliver the triage and the drafts, and say that sending needs Pro or above.

## 2. Collect what is waiting

- Social DMs (Instagram, Facebook, WhatsApp and the other connected networks): call `crm_social_inbox_summary` first. It lists unread totals per network and the threads still waiting on a reply, oldest first. Then `crm_list_social_conversations` with `unreadOnly: true` (optionally `platform`).
- Telegram and other contact conversations: `crm_list_recent_conversations` with `unreadOnly: true`.
- Email: `crm_search_email_threads` with `status: "open"` and `unreadOnly: true`.
- X (Twitter) DMs on demand: `crm_search_twitter_messages` with a free-text `query`.

## 3. Read before ranking

For the top candidates only, read the thread:

- Social: `crm_get_social_conversation` (participant, linked contact, last 10 messages), then `crm_list_social_messages` to page further back if needed.
- Contact conversation: `crm_get_conversation` with `contactId`.
- Email: `crm_get_email_thread` with `threadId`.
- Context on the person: `crm_get_contact` (tags, lead score, ad source) and `crm_get_contact_activity`.

## 4. Rank

Give each thread a priority (urgent, today, can wait) using: how long it has waited, buying signals (pricing, demo, "ready to start"), lead score and open deals, complaints or deadlines. Output a short table: contact, channel, waiting since, priority, one-line reason.

## 5. Draft replies

Write each draft in the language and tone of the thread. Keep it short and specific to what the person asked. Show the draft together with the channel, the recipient and the conversation it belongs to. Never invent prices, dates or commitments that are not in the thread or in the CRM data.

## 6. Approval gate (required before any send)

Sending reaches a real person immediately. For every message, show the exact text, the recipient and the channel, then ask the person to approve. Send only after a clear yes for that message. Do not batch-send on a general "go ahead" for the whole list unless the person approved each draft.

Send tools, by channel:

- Social DM: `crm_send_social_message` (`conversationId`, `text`, optional `mediaUrl`). It pauses the AI auto-reply for the linked contact. Platform rules still apply, for example WhatsApp only allows free-form replies inside its reply window.
- Telegram: `crm_send_telegram_message` (`accountId`, `text`, and `contactId` or `username`). It is queued and sent asynchronously; check `crm_list_jobs` afterwards.
- X DM: `crm_send_twitter_dm` (`xAccountId`, `contactId`, `text`).
- Email: the MCP server cannot send mail. Give the person the draft to send from their mail client or the Pinlyx panel.

## 7. Housekeeping (ask first)

After a reply is handled, offer these and run them only when the person agrees: `crm_mark_social_conversation_read`, `crm_set_email_thread_status` (open, pending, closed), `crm_create_task` for a follow-up, `crm_add_contact_note` for a key fact.
