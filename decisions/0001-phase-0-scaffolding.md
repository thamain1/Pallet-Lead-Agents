# 0001 — Phase 0 Scaffolding: Next.js 15, Supabase SSR, Tailwind, shadcn/ui

**Date:** 2026-06-01
**Status:** Accepted
**Deciders:** Jesse Morgan (lead)
**Phase:** Pilot Phase 0 — Project Setup (per `pallet_platform_pilot_build_mou_invoice_source.md` §21)

---

## Context

The Pallet Lead Agents pilot is greenfield. The pilot spec (`pallet_platform_pilot_build_mou_invoice_source.md`) suggests Next.js as a frontend framework but leaves the choice to the build team. The full v1.0 spec recommends Next.js explicitly. The rest of the lead developer's portfolio runs React + Vite + Cloudflare Pages on Supabase.

We need a stack that:

- supports the pilot's agent-heavy server-side workload (Gemini calls, SendGrid webhook ingestion, Google Places/Routes lookups) without a separate backend
- handles authenticated server-rendered routes for the owner approval queue (cookie-based session refresh)
- pairs cleanly with Supabase Row Level Security
- doesn't force a CDN/edge migration mid-build
- works for a two-developer team where one developer is junior

## Decision

Next.js 15 with the App Router on the frontend. Supabase for database, auth, RLS, storage, and edge functions. Tailwind CSS for styling. shadcn/ui for components. TypeScript everywhere. Cloudflare Pages for hosting.

Specifically:

- **Next.js 15 + React 19 + App Router** — server components handle the agent calls and SendGrid webhook routes natively; no separate backend needed. Typed routes enabled.
- **Supabase SSR (`@supabase/ssr`)** — three client factories (`lib/supabase/client.ts` for browser, `lib/supabase/server.ts` for server components/actions, `lib/supabase/middleware.ts` for session refresh in `middleware.ts`).
- **Tailwind CSS 3 + shadcn/ui** — component primitives copied into `components/ui` as we need them, configured via `components.json`. CSS variables for theming (light/dark via `globals.css`).
- **TanStack Query (`@tanstack/react-query`)** — server-state caching on the client side for dashboard widgets.
- **Recharts** — dashboard charts.
- **Lucide React** — icons.
- **TypeScript strict mode + `noUnusedLocals` + `noUnusedParameters`** — catches dead code early; matches the lead developer's standard for other repos.

Out-of-scope at Phase 0 (deferred to later phases or out of scope entirely):

- testing framework (Jest / Vitest / Playwright) — to be added at Phase 1 or 2 when the first real logic exists to test
- CI workflows — to be added once tests exist
- Mapbox / Google Maps JavaScript wrapper — install when the map page is built (Phase 5)
- shadcn/ui components — install with `npx shadcn@latest add <component>` as each feature needs them
- Supabase migrations — added in Phase 1 (schema design)
- Supabase edge functions — added when the SendGrid webhook handler is needed (Phase 4)

## Consequences

### Positive

- **One framework for frontend + server-side agents.** Next.js API routes / server actions remove the need for a separate Express layer or a swarm of Supabase Edge Functions for things that can live in the Next.js app.
- **Built-in auth pattern.** Supabase SSR's middleware refresh + cookies model is the recommended pattern and aligns with App Router's session model.
- **shadcn/ui = own your components.** Components are copied into the repo, not bundled — easy to customize, no library versioning anxiety.
- **Strict TypeScript catches problems early.** Especially important for a junior-led implementation.

### Negative / trade-offs

- **Different from the lead developer's other 25 builds.** Most other 4ward repos are React + Vite + Cloudflare Pages with Supabase Edge Functions. The Pallet Lead Agents codebase will look different.
- **Cloudflare Pages + Next.js requires either the `@cloudflare/next-on-pages` adapter or Cloudflare Workers Sites pattern.** Hosting nuance to revisit before production launch.
- **App Router learning curve.** Server vs client components, server actions, streaming — more concepts to learn than Vite's static SPA model.

### Mitigations

- Document the patterns that diverge from other 4ward repos here and in `CLAUDE.md`.
- Defer Cloudflare deployment specifics to Phase 6 (production prep). Local dev runs on `next dev` regardless.

## Alternatives considered

### React + Vite + Supabase Edge Functions (matches lead developer's other builds)

Faster onboarding for the lead developer; same patterns as ImpactTracker / IntelliService / MentorApp / etc. Friction is that every server-side agent call needs to live in a separate Supabase Edge Function, which is operationally heavier (separate deploy, separate auth/cookie wiring, no shared type definitions with the frontend).

Rejected because the agent-heavy workload of this build benefits substantially from co-located server actions and the SendGrid webhook is much cleaner as a Next.js API route than an Edge Function.

### Remix or SvelteKit

Both are capable but not in use at 4ward. Adds unfamiliar runtime + ecosystem for marginal benefit.

Rejected — Next.js is already familiar enough.

## References

- `pallet_platform_pilot_build_mou_invoice_source.md` §5 (Technology Stack)
- `pallet_lead_generation_platform_build (1).md` §4 (Recommended Tech Stack — informational only; pilot spec is binding)
- `CLAUDE.md` — Stack section reflects this decision
