---
name: ads-weekly-report
description: Weekly ad performance report across Google Ads, Meta (Facebook and Instagram) and TikTok through Pinlyx, with week-over-week comparison, budget pacing, account alerts and CRM lead attribution. Use for "how did our ads do this week", ad health checks or budget questions. Reporting is read-only; any budget, bid or status change needs explicit approval.
---

# Weekly ads report

Works against the Pinlyx MCP server. Tool names are shown without the client prefix. The report only reads data. Do not change anything unless the person asks.

## 1. Find the accounts

- `crm_whoami` for plan and scopes (ads reading needs the ads read scope).
- Google Ads: `crm_list_google_ads_accounts`. Use a `customerId` from this list. A manager (MCC) account has no metrics; use one of its client accounts.
- Meta: `crm_list_meta_ad_accounts`. TikTok: `crm_list_tiktok_ad_accounts`. Note any account the list flags as unusable (expired authorization, disabled, spending limit) and report it instead of silently skipping it.

## 2. Pull the numbers

Use a seven day window and compare with the week before.

Google Ads, per `customerId`:
- `crm_google_ads_summary` with `range: "LAST_7_DAYS"` and `compareTo: "previous_period"`.
- `crm_google_ads_campaigns` for the per-campaign split, including status reasons such as budget constrained.
- `crm_google_ads_budget_pacing` for spend against budget and projected month spend.
- `crm_google_ads_account_alerts` for anything that stops or throttles ads.
- `crm_google_ads_breakdown` with `level: "search_terms"` for wasted or promising search terms.
- `crm_google_ads_recommendations` for Google's suggestions (report them; do not apply).

Meta, per `accountId`: `crm_meta_ads_summary` (`range: "LAST_7_DAYS"`, `compareTo: "previous_period"`), `crm_meta_ads_campaigns`, and `crm_meta_pixel_status` before trusting the conversion numbers.

TikTok, per `accountId`: `crm_tiktok_ads_summary` and `crm_tiktok_ads_campaigns` with `range: "LAST_7_DAYS"`. TikTok only accepts preset ranges and counts days in UTC.

Across networks: `crm_ads_attribution_report` joins spend with CRM leads and deals won, attributed from the ad source recorded on each contact. Meta and TikTok are included only for the preset ranges (LAST_7_DAYS works).

Notes on windows: LAST_N_DAYS ranges end yesterday; use TODAY or `startDate` and `endDate` to include today. Reports state their window and freshness; quote them.

## 3. Write the report

1. Headline per network: spend, clicks, conversions, cost per conversion, ROAS, each with the change against the previous week.
2. Best and worst campaigns by cost per result, with the reason when the data shows one.
3. Budget and delivery: pacing, campaigns limited by budget, disapproved ads, pixel problems, account alerts.
4. CRM view: leads and deals attributed to ads, cost per lead, and how much is unattributed.
5. Three to five recommended actions, each marked as "needs your approval".

Never add or compare different currencies as if they were the same; show each currency separately. Say when a number is missing or a network is not connected. Do not invent benchmarks.

## 4. Optional changes (approval gate)

Only after the person approves a specific change, and after stating the account, the campaign, the current value and the new value:

- Google Ads: `crm_update_google_ads_budget` (`customerId`, `campaignId`, `dailyBudget`) and `crm_set_google_ads_status` (`customerId`, `level`, `status`). Run with `validateOnly: true` first and show the result. Removing anything needs `confirm: true` and is permanent. `crm_google_ads_undo` (`changeId`) reverts a change made through Pinlyx.
- Meta: `crm_update_meta_ads_budget` and `crm_set_meta_ads_status`. Archiving or deleting needs `confirm: true` and cannot be reactivated.
- TikTok: `crm_update_tiktok_ads_budget` and `crm_set_tiktok_ads_status`. TikTok has no validate-only mode.

Anything that spends money or removes something must be confirmed with the person every time, even if a similar change was approved earlier.
