---
name: contact-cleanup
description: Clean up and organize Pinlyx contacts and tags: find untagged or mis-staged contacts, standardize the tag dictionary, fix lead scores and owners. Use when the person wants to tidy the CRM, tag a segment, or audit contact data. Shows a change list and applies it only after approval.
---

# Contact and tag cleanup

Works against the Pinlyx MCP server. Tool names are shown without the client prefix.

## What this skill can and cannot do

It can search, tag and untag, create tags, set stage, set lead score, assign an owner and append notes. There is no MCP tool that deletes or merges contacts. If duplicates or junk contacts turn up, list them (id, name, platform, why) so the person can handle them in the Pinlyx panel. Do not claim to have merged or deleted anything.

## 1. Start

- `crm_whoami` to check plan and scopes. On a read-only plan, produce the audit and the change list, and do not attempt writes.
- `crm_list_tags` to see the tag dictionary with the number of contacts per tag. Spot near-duplicates (for example "Lead" and "lead", "VIP" and "vip-customer"), unused tags and tags that are too broad.
- `crm_list_pipelines` to know the valid stages.

## 2. Find the contacts

`crm_search_contacts` (`query` by name, username or phone; optional `platform`; returns at most 25, most recently contacted first). For a segment, run several focused searches rather than one broad one. `crm_top_contacts` surfaces the most active contacts of the last 30 days.

For anything ambiguous, open `crm_get_contact` (tags, lead score, custom fields, ad source, last messages) and `crm_get_contact_activity` before proposing a change.

## 3. Build the change list

Present a table with one row per proposed change: contact id, name, current value, new value, reason. Group by type: tag add, tag remove, stage change, score change, owner change, note. Keep the list to what the evidence supports.

## 4. Approval gate (required before any write)

Show the full list and ask for approval. Apply only approved rows. For more than about ten contacts, confirm the batch as a whole and report counts when done. Always get explicit approval for:

- Removing tags (`crm_untag_contact`) and changing stage (`crm_update_contact_stage`), since both alter how the contact is worked.
- Reassigning an owner (`crm_assign_contact`). `assignedToUserId` must come from the person, because no tool here lists team members; omitting it unassigns the contact.
- Notes (`crm_add_contact_note`): the field holds 500 characters, and once full the oldest text is dropped, so a note can remove earlier text. Warn about this and keep notes short.

Write tools:

- `crm_create_tag` (`name`, optional `color`) is idempotent and does not attach anything.
- `crm_tag_contact` (`contactId`, `tagId` or exact `tagName`) never creates a tag; create it first. Idempotent.
- `crm_set_lead_score` (`contactId`, `score` 0 to 100) marks the score as a manual override.

## 5. Report

Summarize what changed, what was skipped and why, and what still needs the panel (duplicates, deletions).
