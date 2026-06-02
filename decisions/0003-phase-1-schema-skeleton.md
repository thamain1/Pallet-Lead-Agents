# 0003 — Phase 1 Schema Skeleton

**Date:** 2026-06-01
**Status:** Skeleton — intern fills in implementation under lead-developer review
**Phase:** Pilot Phase 1 — CRM Foundation (per pilot spec §21)
**Owner:** Intern (build), lead developer (review + merge)

---

## Goal of Phase 1

Stand up the Supabase Postgres schema, Row Level Security policies, and database-side compliance helpers that every subsequent phase depends on. After Phase 1 the application can store companies, contacts, notes, tasks, opportunities, and audit log entries — but does **not** yet need Google Places, Gemini, or SendGrid integration. Those land in Phases 2–5.

Phase 1 is the foundation: get the data model right and the gates correct before any agent or external API touches the database.

## Reference materials

| Doc | Section | Use |
|---|---|---|
| `pallet_platform_pilot_build_mou_invoice_source.md` | §8 (Database Design) | Pilot's 12-table starting list |
| `pallet_lead_generation_platform_build (1).md` | §14 (Supabase Data Model) | Fuller v1.0 schema — use as enum + index reference, NOT as a maximal target |
| `pallet_lead_generation_platform_build (1).md` | §15 (RLS Requirements) | RLS policy patterns |
| `pallet_platform_pilot_build_mou_invoice_source.md` | §12 (Compliance Requirements) | What the compliance gate must enforce |
| `CLAUDE.md` | "Single-tenant" + "Owner-approval gate" | Hard rules for Phase 1 |

When the pilot spec and the v1.0 spec disagree, **pilot spec wins**. Use v1.0 only for enum values, index suggestions, and patterns the pilot spec omits.

## Pre-decided constraints (do not relitigate)

1. **Single-tenant.** No `tenants` or `organizations` table. No `tenant_id` / `organization_id` foreign key on any table. RLS is role-based, not tenant-scoped. See `CLAUDE.md`.
2. **PostGIS is in.** Spatial queries on leads (drive-time bands, route clusters, "leads within 50 miles of base") need it. Enable in the very first migration. Retrofitting hurts.
3. **Owner-approval gate is a Postgres function**, not application-level checks scattered across send paths. One function, called from every code path that could result in an outbound email.
4. **Enums vs TEXT:** prefer enums for closed sets that genuinely never extend (`company_status`, `lead_type`, `task_status`, `opportunity_type`). TEXT with a CHECK constraint for sets that may need additions (industry categories, route corridor names). Memory note from IntelliService: enum-to-TEXT migration is painful — be conservative about adding new enums.
5. **`updated_at` triggers** on every mutable table.
6. **Audit log captures every state change** on companies, contacts, campaigns, email messages, and route plans — by trigger, not by application code.

## Migration plan — order and naming

Use Supabase CLI migration files (or hand-write SQL files in `supabase/migrations/`). Sequential numbering:

```
supabase/migrations/
  0001_extensions_and_enums.sql
  0002_core_tables.sql           -- profiles, companies, contacts
  0003_lead_intake.sql            -- company_sources, lead_scores, owner_reviews
  0004_pipeline.sql               -- opportunities, tasks, notes
  0005_outreach.sql               -- campaigns, email_templates, email_messages, email_events
  0006_suppressions.sql           -- suppressions table
  0007_routing.sql                -- route_plans, route_stops (with PostGIS geom)
  0008_agent_observability.sql    -- agent_runs, audit_logs
  0009_rls_policies.sql           -- all RLS in one file for review clarity
  0010_compliance_gate_function.sql  -- the shared send-gate function
  0011_triggers.sql               -- updated_at, audit_log, owner-approval propagation
```

Adjust ordering if FK dependencies force it. Keep migrations atomic — each file should be reviewable in one PR.

## Per-migration intent

### 0001 — Extensions and Enums

```sql
create extension if not exists "uuid-ossp";
create extension if not exists pgcrypto;
create extension if not exists postgis;

-- Enums per pilot spec §8 + v1.0 §14:
-- lead_type, company_status, contact_status, opportunity_type, task_status, task_type
```

**Decisions to make:**
- Final enum names and values. Use v1.0 §14 as the source unless the pilot spec contradicts.
- Whether to add `agent_run_status` as an enum or leave as TEXT.

### 0002 — Core Tables

`profiles` (links to `auth.users`), `companies`, `contacts`.

- `profiles.role`: CHECK constraint values `('owner', 'admin', 'sales', 'ops', 'viewer')`.
- `companies` includes `lat`, `lng`, AND a generated `geog geography(Point, 4326)` column with a GIST index. Spatial queries hit the `geog` column, not `lat`/`lng`.
- `companies.duplicate_of` self-reference for the dedup workflow.
- All boolean approval/suppression flags default `false`.
- `contacts.email_verified_status` as TEXT with CHECK, not enum.

### 0003 — Lead Intake

`company_sources` (provenance), `lead_scores` (one-to-many — keep history of score changes), `owner_reviews` (audit of owner decisions).

- `lead_scores` rows are append-only — never UPDATE, only INSERT. Latest score is `ORDER BY created_at DESC LIMIT 1`.
- `owner_reviews.decision` CHECK against the closed set from pilot spec §7.2.

### 0004 — Pipeline

`opportunities`, `tasks`, `notes`.

- `opportunities.recurring boolean` for recurring pickup / recurring delivery accounts.
- `tasks.task_type` enum covers the eight types from v1.0 §14.

### 0005 — Outreach

`campaigns`, `email_templates`, `email_messages`, `email_events`.

- `campaigns.status` CHECK constraint matches v1.0 §14: `('draft', 'owner_review', 'approved', 'sending', 'paused', 'completed', 'archived')`.
- `email_templates.compliance_passed boolean` — gate field.
- `email_messages.sendgrid_message_id` indexed (webhook lookups).
- `email_messages.blocked_reason TEXT` — populated when the compliance gate refuses the send.
- `email_events.event_payload jsonb` for the raw SendGrid event blob.

### 0006 — Suppressions

`suppressions` table with `unique(email, suppression_type)`. Used by the compliance gate.

### 0007 — Routing

`route_plans`, `route_stops`. Both reference `companies` for stop locations.

- Add a `geog geography(Point, 4326)` to `route_stops` (snapshot of stop location at time of route generation — companies move addresses, the route shouldn't change retroactively).

### 0008 — Agent Observability

`agent_runs`, `audit_logs`.

- `agent_runs` per v1.0 §14 — records every agent invocation with input/output JSON.
- **Add `model TEXT`, `input_tokens INT`, `output_tokens INT` columns** beyond v1.0 §14 — required for the 15% TP markup billback math per `decisions/0002-gemini-account-reuse.md`.
- `audit_logs` captures actor (user/agent/system), action verb, entity type + id, before/after JSON.

### 0009 — RLS Policies

Single file for review clarity. **All tables enable RLS.** Policy pattern:

- **SELECT:** any authenticated `profiles` row with an allowed `role` for that table.
- **INSERT / UPDATE / DELETE:** restricted by role per the matrix in v1.0 §15.
- **`auth.uid()` is the join key**, NOT a `tenant_id`. Single-tenant rule.
- **`suppressions` table is INSERT-only for `admin` / `owner`** — never DELETE except by the daily cleanup job (if any).

**Watch for:** RLS cross-table policy cycles (when policy on table A queries table B, whose policy queries A) — see Jesse's memory note `feedback_supabase_rls_recursion.md`. Use SECURITY DEFINER helper functions to break cycles.

### 0010 — Compliance Gate Function

The non-negotiable. One function, called from every send path:

```sql
create or replace function pal_can_send_email(p_email_message_id uuid)
  returns table (ok boolean, blockers text[])
  language plpgsql security definer set search_path = public
as $$
declare
  v_msg record;
  v_company record;
  v_contact record;
  v_campaign record;
  v_template record;
  v_blockers text[] := '{}';
begin
  -- Load all the records we need; if any row is missing, block.
  -- Then check, in order:
  --   1. company.owner_approved
  --   2. contact.owner_approved
  --   3. campaign.owner_approved AND campaign.status = 'approved'
  --   4. template.compliance_passed
  --   5. contact.unsubscribed = false
  --   6. contact.suppressed = false
  --   7. contact.do_not_contact = false
  --   8. company.do_not_contact = false
  --   9. NOT EXISTS (suppressions for contact.email)
  --  10. contact.email_verified_status IN ('verified', 'generic_company_email')
  -- Return (false, v_blockers) if any check fails.
  -- Return (true, '{}') only if every check passes.
end;
$$;
```

**Rule:** the Edge Function or server action that actually calls SendGrid must `SELECT * FROM pal_can_send_email(:id)` and refuse to call SendGrid if `ok = false`. The blocker array is written to `email_messages.blocked_reason` for the audit trail.

**Why a single function:** application-side checks drift. Multiple send paths (initial campaign send, follow-up, manual resend) each forget one check. Centralizing in Postgres means the gate is enforced by the database itself, not by every caller remembering.

### 0011 — Triggers

- `updated_at` trigger on every mutable table.
- Audit-log trigger on `companies`, `contacts`, `opportunities`, `campaigns`, `email_messages`, `route_plans`. Captures old vs new JSON into `audit_logs`.
- `bounce → suppression` trigger: when an `email_events` row is inserted with `event_type = 'bounce'` (hard) or `'spamreport'` or `'unsubscribe'`, INSERT into `suppressions`.

## Seed data

`supabase/seed.sql` (or `supabase/migrations/9999_seed_dev.sql` if you'd rather keep it in migrations):

- One `profiles` row for the lead developer with `role = 'owner'`.
- A handful of `companies` rows with realistic Atlanta addresses for local dev (no real Pallets-customer data).
- Three `email_templates` (pickup source, buyer, partner) marked `compliance_passed = false` until reviewed.

Do **not** seed Anthony's real business or any real lead data in seed files — those live in production only.

## Acceptance criteria for Phase 1

The phase is complete when:

- [ ] All 11 migration files apply cleanly to a fresh Supabase project via `supabase db push` (or the Management API).
- [ ] PostGIS extension is enabled and `companies.geog` populates correctly from `lat`/`lng`.
- [ ] RLS is enabled on every tenant-owned table; an unauthenticated request returns zero rows.
- [ ] `pal_can_send_email(uuid)` returns `(false, blockers[])` when any approval/suppression check fails, and `(true, '{}')` only when all pass — verified with at least four unit-style SQL tests in `supabase/tests/` or a `pgTAP`-style assertion file.
- [ ] Audit log captures inserts and updates on `companies` and `contacts` automatically.
- [ ] Bounce + unsubscribe + spamreport `email_events` rows trigger suppression inserts.
- [ ] `database.types.ts` regenerated via `supabase gen types typescript` and committed.
- [ ] Schema reviewed and merged via PR (no direct pushes to `main`).
- [ ] ADR `0003-phase-1-schema-final.md` (rename of this skeleton) committed reflecting the actual decisions made.

## Out of scope for Phase 1

- Google Places integration — Phase 2
- CSV import — Phase 2
- Lead classification / Gemini agent calls — Phase 3
- Owner approval UI — Phase 3
- Outreach drafting — Phase 4
- SendGrid integration and webhook — Phase 4
- Map UI and route generation — Phase 5
- Weekly automated reports — Phase 6

If a Phase 1 task requires touching anything in that list, stop and surface it — it's a phase boundary.

## How to work this phase

1. Read the pilot spec §8 carefully. Then v1.0 §14 as a reference.
2. Open a PR per migration file, or a single PR with all migrations if smaller. Either is fine — the reviewer will say which they prefer on the first PR.
3. Include test SQL (or `pgTAP` assertions) for the compliance gate function specifically. That is the most consequential piece of Phase 1.
4. Update this ADR with the actual decisions made — rename to `0003-phase-1-schema-final.md` and commit alongside the final PR.
5. Generate `database.types.ts` and commit.

## When in doubt

- Bias toward simpler. The pilot spec's 12 tables are enough; don't add tables that v1.0 includes but pilot doesn't.
- When the schema conflicts with the pilot spec, the pilot spec wins.
- When the schema can't comply with the single-tenant rule cleanly, surface to the lead developer — don't quietly add a `tenant_id`.
