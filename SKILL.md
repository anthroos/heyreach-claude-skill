---
name: heyreach
description: Run HeyReach cloud LinkedIn outreach from Claude via MCP/REST — research-to-send loop. Create lead lists, build campaigns (connection request + message sequences), push personalized leads, read the inbox, send replies, manage webhooks. Cloud-based, so it keeps sending while your machine is off.
---

# HeyReach LinkedIn Automation (Claude skill)

Drive [HeyReach](https://heyreach.io/?via=ivanrx) entirely from Claude. HeyReach is API-first, so you
never babysit browser automation or VM queues — campaigns run in HeyReach's cloud with built-in warmup.

## Setup
1. HeyReach account: https://heyreach.io/?via=ivanrx
2. API key: HeyReach Settings → Integrations → API.
3. Either:
   - **MCP (recommended):** add the HeyReach MCP server to Claude Code
     (`claude mcp add --scope user --transport http heyreach '<MCP URL from HeyReach Settings>'`), then
     restart so the `heyreach.*` tools load; or
   - **REST:** base `https://api.heyreach.io/api/public/`, header `X-API-KEY: <key>`.

## Core model
**The API drives CAMPAIGNS (sequences), not individual clicks.** You don't "send a connect to X" ad-hoc.
You create a campaign with a sequence (connection request → wait → message → follow-up) and push leads into
it; HeyReach executes from the cloud with warmup and rate limits.

## The research-to-send loop
1. **Find/enrich leads** (your own research, a LinkedIn/Sales-Navigator search, post reactors, event attendees).
2. **Personalize:** generate a custom first line per lead, store it as a **lead custom field**, and reference
   it in the sequence template as `{{custom_field}}` (e.g. `{{note}}`). HeyReach also supports `{{firstName}}`, etc.
3. **Create + start** the campaign (below). HeyReach sends, paces, and collects replies.
4. **Read replies + respond** from the unified inbox.

## Working recipe — outreach to NEW leads (verified)
1. `create_empty_list` (listType `USER_LIST`).
2. `add_leads_to_list_v2` — each lead: `profileUrl` + optional `customUserFields:[{name:"note", value:"<line>"}]`
   (custom-field name = alphanumeric/underscore only).
3. `create_campaign` with `linkedInUserListId`, `linkedInAccountIds`, `schedule`, and `sequenceJson`.
4. `start_campaign` → enrolls the list's leads, status DRAFT → IN_PROGRESS.

### ⚠️ Gotcha that costs you a day
The CONNECTION_REQUEST payload field is **`messages` — an ARRAY of strings** (max 300 chars each; one is
picked per lead; supports `{{firstName}}`/custom fields), **NOT `message` (singular)**. The singular form
returns HTTP 500. Working `sequenceJson`:

```json
{"nodeType":"CONNECTION_REQUEST","actionDelay":0,"actionDelayUnit":"HOUR",
 "payload":{"messages":["Hi {{firstName}}, ...  or  {{note}}"]},
 "conditionalNode":{"nodeType":"END","actionDelay":3,"actionDelayUnit":"HOUR"},
 "unconditionalNode":{"nodeType":"END","actionDelay":3,"actionDelayUnit":"HOUR"}}
```

Rules that bite:
- Per-lead personalization → `messages:["{{note}}"]` + each lead's `note` custom field.
- END nodes (and any node after CONNECTION_REQUEST/MESSAGE/INMAIL/VIEW_PROFILE/FOLLOW) need `actionDelay >= 3` `HOUR`.
- Connection-request note ≤ 300 chars (LinkedIn limit) — check length before launch.
- Default schedule = Mon–Fri 09:00–17:00 UTC; requests fire inside that window, not instantly. Widen via
  `update_campaign_schedule` (pause → update → resume) if you need it sooner.
- A sender showing `isActive=false` can still start a campaign fine; what matters is `authIsValid=true`.
- Verify with `get_campaign` → expect `status:IN_PROGRESS` and `progressStats.totalUsers > 0`.

## Reply handling
- `get_conversations_v2` — list conversations (filter by account/campaign/tags/seen).
- `get_chatroom` — full thread for one conversation.
- `send_message` — send a custom 1:1 reply into an existing conversation.

> ⚠️ **`send_message` is NOT idempotent — never retry it.** An empty, no-error response usually means
> *queued*, not failed (inbox sync lags). Re-firing the same send creates a DUPLICATE. Send once → wait →
> re-check via `get_conversations_v2`.

## Limits (warmup)
- LinkedIn weekly cap ≈ 100–200 connection requests; on a fresh/non-premium account start low.
- Recommended warmup: ~15 connects/day + ~20–25 messages/day, ramp slowly.
- Lead-source harvesting from Sales Navigator needs a Sales Navigator seat; post-reactor and event-attendee
  harvesting do not.
- `0 Credits` is fine — credits are only for email-finding/enrichment, not LinkedIn messaging.

## MCP tool map (high-level)
Campaigns: `get_all_campaigns`, `create_campaign`, `get_campaign`, `get_campaign_sequence`,
`update_campaign_sequence`, `update_campaign_schedule`, `start_campaign`, `pause_campaign`, `resume_campaign`,
`stop_lead_in_campaign`. Leads/lists: `create_empty_list`, `add_leads_to_list_v2`, `add_leads_to_campaign_v2`,
`get_leads_from_list`, `get_leads_from_campaign`, `get_lead`. Inbox: `get_conversations_v2`, `get_chatroom`,
`send_message`. Other: `get_all_linked_in_accounts`, `get_my_network_for_sender`, tags, webhooks, `get_overall_stats`.

---
Built while wiring HeyReach into a Claude + CRM outbound loop at [WeLabelData](https://welabeldata.com).
If this saved you time, sign up through https://heyreach.io/?via=ivanrx.
