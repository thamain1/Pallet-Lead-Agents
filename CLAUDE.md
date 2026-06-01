# Pallet Lead Agents — Build Context for Claude Code

> Lead generation, lead approval, outreach, routing, and lightweight CRM platform for an Atlanta-area pallet business. Pilot build.

This file exists so any Claude Code session on this repo immediately sees the binding scope, stack decisions, and non-negotiables — without re-asking. If you're a human reader, this is also a fast onboarding doc.

---

## Scope is the pilot spec, not the v1.0 spec

**Source of truth for what to build:** `pallet_platform_pilot_build_mou_invoice_source.md`, sections **§1–16** and **§19–23**. This is the contractually binding scope.

The other root-level spec, `pallet_lead_generation_platform_build (1).md`, is the v1.0 reference — broader than the pilot. **Do not use it as the source of truth for what to build.** Use it only when a pilot-spec section is ambiguous and the v1.0 spec clarifies intent.

If a user request or design proposal falls outside the pilot spec, flag it and surface that it needs a separate scope agreement before implementation.

---

## Single-tenant

This is a **single-tenant build**. One pallet company is the only end user of this deployment.

- **No** `tenants` or `organizations` table.
- **No** `tenant_id` / `organization_id` foreign key on any table.
- RLS is **role-based only** (e.g., `owner` / `admin` / `sales` / `ops` / `viewer`), not tenant-scoped.
- No org-selector UI, no `/org/[slug]` routes.

Any multi-tenant pattern that creeps into the schema, RLS, or UI should be flagged and removed — it is out of pilot scope, not missing work.

---

## Stack

| Layer | Choice |
|---|---|
| Frontend | Next.js + TypeScript + Tailwind CSS + shadcn/ui |
| Backend | Supabase (Postgres + Auth + RLS + Edge Functions + Storage) |
| Email | SendGrid (transactional + suppression + webhooks + ASM groups) |
| Maps / location | Google Places API + Google Maps |
| AI agents | Gemini 2.5 (advisory only — never auto-sends anything) |
| Hosting | Cloudflare Pages |

**PostGIS:** Add the extension early. Lead spatial queries (drive-time bands, route clustering) need a `geog geography(Point, 4326)` column with a GIST index. Retrofitting is painful — do it in the first migration.

---

## Owner-approval gate (non-negotiable)

No outbound email may be sent until **all** of the following are true:

- Lead is owner-approved
- Contact is owner-approved
- Campaign is owner-approved
- Email template has passed compliance check
- Contact is **not** unsubscribed, suppressed, bounced, or marked do-not-contact
- Sending domain is configured (SPF / DKIM / DMARC verified)

Implement this as a **single shared function** (Postgres function or shared TS module) called from every send path. Do not duplicate the check inline — duplicate checks drift, and one bypass = compliance liability.

---

## Out of scope for the pilot (DO NOT IMPLEMENT)

These are explicitly deferred. If a request comes in for any of these mid-build, flag it and note that it requires a separate scope agreement.

- Paid lead-data enrichment providers (Apollo, ZoomInfo, Clearbit, People Data Labs, Data Axle, Clay, Hunter, etc.)
- Third-party email-verification services
- Automated email-reply parsing or classification
- Advanced multi-truck route optimization
- SMS outreach (consent capture, STOP keyword handling, etc.)
- Inbound web lead-capture forms or SEO landing pages
- Customer portal (pickup requests, account self-service)
- Driver-facing mobile app or route view
- Invoicing / payment / accounting-software integrations
- Automated weekly owner reports or advanced analytics dashboards
- Pallet inventory forecasting, photo-based pallet condition classification, or quote generation
- Service-area expansion beyond the configured three-hour drive-time radius

---

## Pre-launch checks before paid services are provisioned

- **Verify the engagement deposit has cleared before provisioning any paid third-party services, domains, or production accounts.** Infrastructure costs are billed back to the client with a fixed admin markup; do not start incurring charges until the deposit milestone is confirmed paid.
- SendGrid domain authentication (SPF / DKIM / DMARC) must be configured and verified before any production send.
- Plan a **~2-week SendGrid sender warmup** before the first large outreach batch. Starting at full volume on a fresh domain will tank sender reputation.
- Set per-month cost ceilings and alert thresholds on every paid third-party account (Supabase, SendGrid, Google Cloud, Gemini API, hosting). Runaway agent loops or misconfigured campaigns are the realistic failure modes.

---

## Build phasing

Per pilot spec §21:

1. Confirm scope + create Supabase project
2. Schema + RLS + auth + role structure
3. Lead table + lead detail views
4. Google Places lead discovery workflow
5. CSV import workflow
6. Gemini lead classification workflow
7. Owner approval queue
8. Outreach draft generation
9. SendGrid send-after-approval
10. SendGrid event webhook ingestion
11. CRM stages + notes + follow-up tasks
12. Routing / territory grouping
13. End-to-end testing
14. Acceptance and handoff

Each phase should produce a thing the owner can see in the dashboard. Don't ship a phase that depends on the next phase to be useful.

---

## Acceptance criteria

See pilot spec §14. The build is not done until each acceptance item demonstrates working end-to-end. Acceptance is a 10-business-day review window with a deemed-acceptance fallback if no written response is received.

---

## Multi-Claude collaboration convention

This repo has more than one developer, each using Claude Code. Each developer's local `~/.claude/` memory is **private**.

Anything that needs to outlive a conversation or be visible to the other developer's Claude session must live in the repo:

- **Architectural decisions** → ADR-style notes in `decisions/NNNN-short-title.md` (at the repo root, NOT under `docs/` — `docs/` is gitignored on this public repo because it holds commercial documents)
- **Conventions, gotchas, scope clarifications** → update this CLAUDE.md or `CONTRIBUTING.md`
- **Test strategies, env setup, runbooks** → standard repo docs (`README.md`, etc.)

If you make a decision in conversation that affects how the other developer should code, commit a note. The other Claude is not telepathic.

**Note on `docs/`:** The `docs/` folder is gitignored on this public repo because it holds the commercial MOU and invoices (with personally identifying info and banking details). If the repo is later made private, that gitignore can be removed. For now, commercial documents are shared out-of-band.

**Note on `decisions/`:** ADRs live at the repo root in `decisions/`, not under `docs/`. They are tracked in git. Use `decisions/NNNN-short-title.md` numbering. See `decisions/0001-phase-0-scaffolding.md` for the first one.

---

## Quick reference

- **Pilot scope (binding):** `pallet_platform_pilot_build_mou_invoice_source.md` §1–16, §19–23
- **v1.0 reference (broader):** `pallet_lead_generation_platform_build (1).md`
- **Build is single-tenant** (no orgs/tenants)
- **All agents are advisory** — owner approves before any email send
- **Verify deposit before provisioning** paid services
