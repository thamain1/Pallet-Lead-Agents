# Pallet Lead Agents

Lead generation, owner approval, outreach, routing, and lightweight CRM platform. Pilot build.

> **First time on this repo?** Read [`CLAUDE.md`](./CLAUDE.md) first — it has the build rules, stack decisions, and out-of-scope fence. Then read [`CONTRIBUTING.md`](./CONTRIBUTING.md) for how the two-developer workflow works.

---

## Stack

- **Frontend:** Next.js 15 (App Router) + TypeScript + Tailwind CSS + shadcn/ui
- **Backend:** Supabase (Postgres + Auth + RLS + Edge Functions + Storage)
- **Email:** SendGrid
- **Maps / location:** Google Places + Maps + Routes APIs
- **AI agents:** Gemini 2.5 (advisory only — never auto-sends)
- **Hosting:** Cloudflare Pages

## Requirements

- Node.js 20+
- npm (or pnpm / yarn if you prefer)
- A Supabase project (created separately — credentials supplied via `.env.local`)
- API keys for SendGrid, Google, Gemini

## Local setup

```bash
git clone https://github.com/thamain1/Pallet-Lead-Agents.git
cd Pallet-Lead-Agents
npm install
cp .env.example .env.local
# fill in real values in .env.local
npm run dev
```

The app will be at `http://localhost:3000`.

## Scripts

| Command | Purpose |
|---|---|
| `npm run dev` | Start the local dev server |
| `npm run build` | Production build |
| `npm run start` | Run the production build locally |
| `npm run lint` | Run Next.js ESLint |
| `npm run typecheck` | Type-check without emitting |

## Repository layout

```
.
├── app/                  Next.js App Router pages and layouts
├── components/           React components (shadcn/ui lives under components/ui)
├── lib/                  Shared utilities and Supabase clients
│   └── supabase/         Browser, server, and middleware client factories
├── supabase/
│   └── migrations/       SQL migrations (Phase 1 onward)
├── decisions/            ADRs — architectural decisions, in repo
├── docs/                 (gitignored on public repo — commercial documents)
├── CLAUDE.md             Build context for Claude Code sessions
├── CONTRIBUTING.md       Two-developer workflow conventions
└── README.md             This file
```

## Pilot scope

The binding scope for this build is **`pallet_platform_pilot_build_mou_invoice_source.md`** at the repo root, **sections §1–16 and §19–23**. The other root-level spec (`pallet_lead_generation_platform_build (1).md`) is a v1.0 reference — broader than the pilot — and is **not** the build target.

When in doubt, default to the pilot spec.

## Important rules

- **Single-tenant build.** No `tenants` or `organizations` table. RLS is role-based only.
- **Owner-approval gate is non-negotiable.** No email sends without lead + contact + campaign approval + compliance check.
- **No paid third-party services provisioned before deposit clears.** See `CLAUDE.md` § "Pre-launch checks."

Full out-of-scope list in `CLAUDE.md`.

## Multi-developer workflow

This repo has two developers, both using Claude Code. See [`CONTRIBUTING.md`](./CONTRIBUTING.md) for branch / PR / handoff conventions.
