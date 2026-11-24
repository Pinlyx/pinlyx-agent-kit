---
name: sequence-status
description: Report on Pinlyx outbound message sequences and Telegram send jobs: what is running, progress, failures and why messages did not go out. Use for "what campaigns are running", "did my messages go out" or "why did this sequence fail". Pausing or resuming a sequence needs the person's approval.
---

# Sequence and send-job status

Works against the Pinlyx MCP server. Tool names are shown without the client prefix.

## 1. Overview

- `crm_whoami` for plan and scopes. Pause and resume are writes; on a read-only plan, report and recommend only.
- `crm_list_sequences` (optional `status`: active, paused, completed, cancelled; `limit`) lists sequences with status, target counts and progress. Start with `active`, then `paused`.

## 2. Drill into one sequence

`crm_get_sequence_status` (`sequenceId`) returns name, status, target, processed, successful and failed counts, the message steps and recent job activity. Summarize:

- progress (processed out of targets) and success rate,
- the failure count and any pattern in recent jobs,
- whether it is moving or stuck (no recent job activity while active).

## 3. Check the jobs behind it

`crm_list_jobs` lists outbound Telegram send jobs (from sequences and from `crm_send_telegram_message`), newest first. Filter with `status` (queued, sent, failed), `accountId` and `windowDays`. For a specific failure, `crm_get_job` (`jobId`) shows the full text, target, status and `lastError`. Group failures by error and by account: one account failing everywhere points to the account (session, limits), one error across accounts points to the content or the target.

Only Telegram jobs appear here. Do not claim anything about other channels from this data.

## 4. Report

Give a compact table (sequence, status, progress, failed, last activity) and a short diagnosis per problem sequence with the most likely cause and a suggested next step. Quote error text exactly rather than paraphrasing it. If the evidence is thin, say so.

## 5. Pause and resume (approval gate)

Changing a sequence affects real outbound messages, so ask first and name the sequence, its current status and what will happen.

- `crm_pause_sequence` (`sequenceId`): no more messages are queued. Jobs already in flight still run. Idempotent.
- `crm_resume_sequence` (`sequenceId`): the worker starts queueing messages again. Before resuming a sequence that was paused because of failures, confirm that the cause was addressed.

After the change, call `crm_get_sequence_status` again and show the new state. Editing sequence content or targets is not available through MCP; point the person to the Pinlyx panel.
