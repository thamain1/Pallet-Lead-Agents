# Contributing — Pallet Lead Agents

Two developers work on this repo, both using Claude Code. This document lays down the workflow conventions so we don't step on each other.

---

## Roles

| Role | Person | Responsibility |
|---|---|---|
| Lead developer / reviewer | Jesse Morgan (`thamain1`) | Architecture, PR review, merging to `main`, commercial decisions |
| Build developer | Collaborator | Implementing features per phase, opening PRs for review |

All PRs are reviewed and merged by Jesse. Direct pushes to `main` are reserved for the lead developer.

---

## Branches

- **`main`** — protected. Production-ready code only. Direct pushes by the lead developer only.
- **`feature/<short-description>`** — new features (e.g., `feature/lead-search-google-places`)
- **`fix/<short-description>`** — bug fixes
- **`chore/<short-description>`** — refactors, dependency bumps, tooling
- **`docs/<short-description>`** — documentation-only changes

Branch off `main`. Rebase onto `main` before opening a PR if main has moved.

---

## Pull requests

### Opening a PR

1. Push your branch: `git push -u origin feature/<name>`.
2. Open a PR via `gh pr create` or the GitHub UI.
3. Target branch: `main`.
4. PR title: short, conventional-commit style — e.g., `feat: lead search via Google Places`, `fix: bounce events not creating suppressions`.
5. PR description must include:
   - **What** the PR does (1–2 sentences).
   - **Which phase** it belongs to (refer to pilot spec §21 or `CLAUDE.md` phasing).
   - **How to test** locally — at minimum, the steps a reviewer can take to verify.
   - **Out-of-scope items** intentionally not addressed.

### Review

- The lead developer reviews every PR.
- Approving a PR means the change is consistent with the pilot scope, follows house conventions, and is testable.
- Squash-merge by default. Use merge commits only when preserving multiple meaningful commits.

### Hooking up CI

CI is not configured at Phase 0. As features land, add lint + typecheck + test workflows. PR description should call out the addition.

---

## Commit messages

Conventional Commits, lower-cased subject:

```
feat: lead search via Google Places
fix: bounce events not creating suppressions
refactor: extract owner-approval gate into shared function
docs: add ADR for choosing Next.js
chore: bump @supabase/ssr to 0.5.1
```

Body: optional. Use it for the *why*, not the *what* — the diff already shows the what.

When Claude Code authors a commit, include the trailer:

```
Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
```

(Or whichever Claude model the developer is using.)

---

## Architectural decisions (ADRs)

When you make a non-obvious architectural decision — choosing a library, picking a pattern, deciding to deviate from the pilot spec — record it in `decisions/`.

Format: `decisions/NNNN-short-kebab-title.md`. See `decisions/0001-phase-0-scaffolding.md` for the template.

ADRs are how the OTHER developer's Claude session learns about your decision. If you don't write it down, it doesn't exist for them.

---

## Mid-task handoffs

If one developer needs to hand off an in-progress feature to the other:

1. Push the WIP branch with a meaningful commit (don't squash WIP into the final commit).
2. Open a draft PR with the description filled in describing where you stopped and what's next.
3. Add a comment on the PR pinging the other developer.

The receiving developer's Claude can read the PR description and pick up where the previous one stopped. The PR description IS the handoff document.

---

## Scope guard

Two recurring scenarios to watch for:

1. **"While we're here, let's also..."** — If the proposed work is not in the pilot spec §1–16 / §19–23, **do not implement it**. Open a `chore/` or `docs/` PR proposing it as a future-phase note in `decisions/`, or surface to Jesse for a scope decision.

2. **"The v1.0 spec says..."** — The v1.0 spec (`pallet_lead_generation_platform_build (1).md`) is broader than the pilot. If something in v1.0 is NOT in the pilot spec, it's out of scope. Use v1.0 only when the pilot spec is genuinely ambiguous.

---

## Multi-Claude coordination

Both developers use Claude Code. To keep our two Claude sessions from getting out of sync:

- **Each Claude is sandboxed to one developer's machine.** Anything one Claude knows, the other one doesn't, unless it's committed to the repo.
- **If you decide something with your Claude, commit it.** Either as code, as an ADR in `decisions/`, or as an update to `CLAUDE.md` or `CONTRIBUTING.md`.
- **`CLAUDE.md` is the shared brain.** Update it when conventions shift.
- **Local memory does not sync.** Don't assume the other side knows context from your local `~/.claude/` memory.

---

## Local environment

See `README.md` for setup. Required environment variables are in `.env.example`. Never commit `.env.local` or any file with real credentials.

API keys are provided by Jesse. Until you have them, you can run the app shell (`npm run dev`) but Supabase, SendGrid, and the agent flows won't function.

---

## Pre-launch infrastructure rule

Until the engagement deposit has cleared, **do not provision any paid third-party service** under a billable plan. This includes:

- Supabase project on a paid tier
- SendGrid paid plan or domain authentication on a paid plan
- Production domain purchase
- Production hosting plan
- Any Google API on a billing account

You can develop locally against Supabase's free tier and use sandbox/free tiers of the other services until then. Coordinate with Jesse before flipping anything to paid.
