---
name: pipeline-review
description: "Review the Pinlyx sales pipeline and deals, find stalled or overdue opportunities, and propose next steps. Use for weekly pipeline reviews, \"what is in my pipeline\", \"which deals are stuck\" or \"what is overdue\". Proposes changes first and applies them only after approval."
---

# Pipeline and deal review

Works against the Pinlyx MCP server. Tool names are shown without the client prefix.

## 1. Orient

- `crm_whoami` for the plan and scopes. Free plans are read-only: deliver the review and list the proposed changes instead of applying them.
- `crm_list_pipelines` to see the boards and their ordered stages with per-stage contact counts. `crm_get_pipeline` (`pipelineId`) for one board.

## 2. Pull the deals

- `crm_list_deals` once per open stage (`lead`, `qualified`, `proposal`, `negotiation`), and with `stage: "won"` or `"lost"` when the person wants recent outcomes. Use `search` or `contactId` to narrow. Results are ordered by stage then value and include open-task counts.
- `crm_get_deal` (`dealId`) for deals that need a closer look: linked tasks and the contact name.
- `crm_list_tasks` with `overdue: true` (and `dealId` or `contactId` when focusing) to see what is late.

## 3. Add context for the important deals

For the largest or most at-risk deals: `crm_get_contact` and `crm_get_contact_activity` (last touch, stage changes, notes), and `crm_get_conversation` or `crm_list_social_conversations` to see whether the customer replied last or you did. `crm_top_contacts` shows the most-messaged contacts of the last 30 days, a useful warm-lead check.

## 4. Produce the review

Structure the output as:

1. Totals by stage (count and value, per currency; never add different currencies together).
2. Needs attention: deals with no open task, overdue tasks, no recent activity, or an unanswered customer message.
3. Likely to close soon: late-stage deals with recent two-way activity.
4. Proposed actions, each tied to a deal or contact, with the exact tool call it would need.

Say plainly when data is missing (for example no expected close date) instead of guessing.

## 5. Approval gate for changes

Nothing is changed during the review. After presenting the proposals, apply only the ones the person approves, listing each change first (deal, from, to):

- `crm_update_deal_stage` (`dealId`, `stage`): allowed stages are lead, qualified, proposal, negotiation, lost. Moving a deal to won is not supported through MCP because it books revenue; tell the person to do that in the Pinlyx panel. Treat `lost` as a significant change and confirm it explicitly.
- `crm_create_deal` (`title`, optional `contactId`, `value`, `currency`, `stage`, `probability`, `expectedCloseAt`, `notes`).
- `crm_create_task` (`title`, optional `dueAt`, `priority`, `contactId`, `dealId`) and `crm_complete_task` (`taskId`).
- `crm_update_contact_stage`, `crm_set_lead_score` (0 to 100), `crm_add_contact_note`.

Do not message customers from this skill. If a follow-up message is needed, hand over to the inbox-triage skill.
