# 0002 — Reusing the ImpactTracker Gemini Account for Pallet Lead Agents

**Date:** 2026-06-01
**Status:** Decision pending (memo to lead developer)
**Question raised by:** Jesse Morgan

---

## The question

In ImpactTracker we set up agents on Gemini (Gemini 2.5 Flash via the `GEMINI_API_KEY` env var, used by `functions/_lib/gemini.ts` and the writer/validator persona pipeline). How do we use that same account for the Pallet Lead Agents stack?

## Short answer

**Yes — reuse the same Google account and the same Google Cloud project, but generate a separate API key for Pallet Lead Agents. Do not copy the ImpactTracker key string into this build.**

The account is shared. The key is not.

---

## Why a separate key (same account)

| Concern | What sharing the same key costs you | What a separate key buys you |
|---|---|---|
| Cost attribution | Single bill, no per-app split — hard to justify the 15% markup invoice line to the client | Per-key usage visible in Google Cloud Console, exportable for the monthly Third-Party Pass-Through invoice |
| Security / blast radius | One key leak compromises both apps; rotation forces redeploy of both | Rotate one key, the other keeps working |
| Per-app rate limits | Both apps draw from one rate budget; an outreach burst in Pallet starves ImpactTracker reporting | Restrict each key independently (IP, referrer, daily quota cap) |
| Audit trail | "Which app made this call?" is unanswerable from logs | Each call carries its own key identity in Cloud logs |
| Operational hygiene | One env var serves two services — easy to misroute traffic | Clear separation of concerns; matches the rest of your env structure |

## What the two apps DO share

- Same Google account
- Same Google Cloud project
- Same monthly quota pool (combined free-tier RPM and RPD; combined paid-tier billing)
- Same billing account (unless you choose to split, see below)

## How to set it up

1. Sign in to **https://aistudio.google.com** with the same Google account that owns the ImpactTracker key.
2. **API Keys → Create API Key.** When prompted for a Google Cloud project, **pick the existing project ImpactTracker uses — do not create a new project** (unless you want hard separation; see "Optional: hard cost separation" below).
3. Name the key something like `pallet-lead-agents-prod` and `pallet-lead-agents-dev` (one per environment if you want them split). The display name only matters in Cloud Console, not in the app.
4. Copy the key value once. Store it in:
   - `C:\Dev\Pallets_Project\.env.local` (local dev — gitignored)
   - Cloudflare Pages secret: `wrangler secret put GEMINI_API_KEY` (production)
5. In `package.json`, the env var name stays `GEMINI_API_KEY` (matches `.env.example` already in the scaffold).
6. **Never** commit the key value. The PAT-scope guardrails on the public repo will catch some of this, but treat env values as out-of-band always.

## Quota and rate-limit considerations

- **Free tier on Gemini 2.5 Pro:** ~5 RPM, ~25 RPD as of model release. Two apps sharing is fine for development, painful for production.
- **Free tier on Gemini 2.5 Flash** (what ImpactTracker uses): ~10 RPM, ~250 RPD. Friendlier, but still combined.
- **Paid tier (pay-as-you-go):** soft RPM/TPM ceilings, no hard daily cap. Both apps draw from the same combined ceiling on the project.
- **Practical implication:** if Pallet Lead Agents kicks off a large Lead Discovery or Enrichment batch while ImpactTracker is generating an AI report, one or both will hit 429s. Mitigate by running heavy Pallet batches off-peak, or splitting projects (below).

## Model selection — what to actually call

ImpactTracker uses **Gemini 2.5 Flash** for its writer/validator persona pipeline. The Pallet Lead Agents pilot doesn't have to use the same model. Suggested per-agent allocation:

| Agent | Suggested model | Reason |
|---|---|---|
| Territory Agent | Gemini 2.5 Flash | Mostly structured output, cheap, fast |
| Lead Discovery Agent | Flash | High-volume, structured |
| Enrichment Agent | Flash | Structured, repetitive |
| Lead Scoring Agent | Flash or Pro | Borderline — Pro if scoring quality drives owner-approval rate |
| Compliance Agent | Gemini 2.5 Pro | Reasoning over rules; cost is worth it |
| Outreach Drafting Agent | Pro | Style + brand voice + compliance footer awareness |
| Routing Agent | Flash | Numeric/structured |
| CRM Follow-Up Agent | Flash | Pattern-matching, cheap |

The `GEMINI_MODEL` env var in `.env.example` can stay as a default; per-agent override at call time when you build the agent layer.

## Cost attribution for the 15% markup billback

Cloud Console per-key usage is **daily granular** with no clean CSV export. For accurate monthly Third-Party Pass-Through billing under MOU §4.2, instrument in-app metering:

The pilot spec §14 already provides an `agent_runs` table. When each agent invocation runs, record:

- `agent_name` (Territory / Discovery / Enrichment / etc.)
- `model` (e.g., `gemini-2.5-pro`)
- `input_tokens` and `output_tokens` (Gemini returns these on every response in `usageMetadata`)
- `started_at`, `finished_at`, `status`
- the call's input/output JSON for audit

Multiply `input_tokens × input price/M` + `output_tokens × output price/M` (current pricing at https://ai.google.dev/pricing) → exact cost per call. Sum monthly for the TP invoice line item. This is more reliable than guessing from Cloud Console and is a defensible audit trail if Anthony ever questions a TP invoice.

## Budget guardrail (MOU §4.3 — $500/month notice threshold)

Set a Google Cloud budget alert at $500/month on the project's billing account so a runaway agent loop in either app trips an alarm before it eats into your margin. Cloud Console → Billing → Budgets & Alerts → New Budget → set 50%, 80%, and 100% alert thresholds.

If the budget is for the combined project (Pallet + ImpactTracker), tune the threshold to account for ImpactTracker's normal baseline. Or split projects (below).

## Optional: hard cost separation via a separate Google Cloud project

If billback accuracy matters more than setup convenience, create a **new Google Cloud project** for Pallet Lead Agents (separate from ImpactTracker's). This gives you:

- Independent quotas (no cross-app rate competition)
- Independent billing (clean attribution without app-side metering)
- Independent IAM, audit logs, and budget alerts
- Independent API keys (still)

Trade-off: more setup (new project, billing link, API enablement), and you can't share infra with ImpactTracker. Worth it if Pallet's monthly Gemini spend is projected over ~$200, or if you ever want to hand the project to a successor service provider cleanly at end of pilot.

For a 6–8 week pilot at low volume, sharing the project is fine. Promote to a separate project at production scale-up if the build expands.

## Recommended action (minimum viable setup)

1. Same Google account ✓
2. Same Google Cloud project ✓
3. New API key labeled `pallet-lead-agents` ✓
4. Same billing account ✓
5. Budget alert at $500/month combined ✓
6. In-app `agent_runs` metering (built in Phase 4 when SendGrid + agent flow lands) ✓

## What NOT to do

- Don't copy the literal `GEMINI_API_KEY` value from ImpactTracker into this build's env.
- Don't deploy the Pallet Lead Agents build into the ImpactTracker Cloudflare Pages project — they get their own CF project for deploy isolation, even if they share a Gemini key project.
- Don't commit the API key. `.env.local` only. Cloudflare secret only.
- Don't disable the budget alert "temporarily" during a large agent batch. The alert is what stops runaway loops.

---

## Status of this memo

This is **not yet an accepted ADR**. It is the answer to a question Jesse raised on 2026-06-01. If Jesse wants this in the repo as a binding decision, rename it and commit. Otherwise it can be emailed to the intern as setup guidance or kept locally for reference.

## Cross-references

- Pilot spec §14 — `agent_runs` table for metering
- MOU §4.2 — 15% admin markup on Third-Party Pass-Through
- MOU §4.3 — $500/month notice threshold
- `CLAUDE.md` — Stack section (Gemini 2.5 as the agent provider)
- `.env.example` — `GEMINI_API_KEY` and `GEMINI_MODEL`
- ImpactTracker: `functions/_lib/gemini.ts`, `functions/_lib/personas/*` (pattern reference)
