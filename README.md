# heyreach-claude-skill

A [Claude Code](https://claude.com/claude-code) skill to run **[HeyReach](https://heyreach.io/?via=ivanrx)**
LinkedIn outreach end-to-end from your terminal — the full research-to-send loop, with no browser bots and
no VM queues to babysit.

## Why
Doing LinkedIn outreach in a Claude + CRM loop usually means building your own automation: virtual machines,
a send queue, and constant patching so LinkedIn does not break it. That is a never-ending side job.

HeyReach is **API-first** — accounts, campaigns, leads, inbox, replies, webhooks, all over the API. So the
infrastructure you were about to build for months already exists. This skill teaches Claude to drive it.

## What it does
With this skill, Claude can, from one terminal:
- research who to contact in a niche,
- write a personalized first line per person (not a template blast),
- push leads into a HeyReach campaign over the API,
- let HeyReach send from the cloud (your laptop can be off),
- log each contact to your CRM,
- pull replies back in.

You review the ready batch and say "send." HeyReach handles warmup and rate limits so the account stays safe.

## Install
1. Sign up for HeyReach: **https://heyreach.io/?via=ivanrx**
2. Connect your LinkedIn account in HeyReach.
3. Pick a transport (MCP and REST use **different** keys — don't mix them):
   - **MCP (recommended):** Settings → Integrations → **MCP Server** → *Get MCP key* → copy the
     **MCP Connection URL**, then
     `claude mcp add --scope user --transport http heyreach '<MCP Connection URL>'` and restart Claude.
   - **REST (fallback):** Settings → Integrations → **API** → copy the API key (a separate key).
4. Drop `SKILL.md` into your Claude skills (`~/.claude/skills/heyreach/SKILL.md`).

> If the MCP server shows `connected` in `claude mcp list`, let Claude use the `heyreach.*` tools — do
> not point it at REST. Calling REST without an `X-API-KEY` returns `401`, which looks like a broken
> service but just means "wrong transport / missing key."

Full how-to, the working campaign recipe, and the gotchas are in [`SKILL.md`](./SKILL.md).

## The gotcha that cost a day
The `CONNECTION_REQUEST` payload field is **`messages`** (an array of strings), **not** `message`. The
singular form silently returns HTTP 500. See `SKILL.md` for the exact working `sequenceJson`.

## License
MIT.

---
Built at [WeLabelData](https://welabeldata.com). The HeyReach links are referral links
(https://heyreach.io/?via=ivanrx) — if the skill helps, signing up through them supports the work.
