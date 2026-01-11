# Repository Guidelines


## Most Important

- Checkpoint requests = empty commit and push with a concise message that starts with "Checkpoint- " (do not start with *Agent*-). It only means that! Stupid cursor, please never try to implement what I wrote in checkpoint without taking permission!
- Commits authored by Codex must start with `Codex- `
- Commits authored by Cursor / Composer1 agent must start with `Cursor- `
- Commits authored by Claude agent must start with `Claude- `
- Commits that ONLY have edits/updates made by the user must start with `Ricky- ` (and should not include agent co-authored-by trailers).
- **Co-authored-by trailers**: All agent-authored commits MUST include a `Co-authored-by` trailer in the commit message to enable GitLens to identify agent-authored changes. Format: blank line before trailers, then one trailer per line:
  - Codex commits: `Co-authored-by: Codex <codex@local>`
  - Cursor commits: `Co-authored-by: Cursor Agent <cursor@local>`
  - Claude commits: `Co-authored-by: Claude Code <claude-code@local>`
  - Example commit message:
    ```
    Codex-Add holdings ingestion API
    
    Co-authored-by: Codex <codex@local>
    ```
- For every commit, include a best‑effort User vs Agent contribution summary in the commit body, based on skimming the current chat session (and any known user edits). Place the summary before any Co-authored-by trailers. Do not add this summary for commits that start with `Ricky- `. Example:
  ```
  Summary:
  Business Logic - Agent 30% Ricky 70%
  Code - Agent 90% Ricky 10%
  Overall Steering by User - 30%
  User Satisfaction - 70%
  ```
  - Summary field definitions:
    - Business Logic: who decided what the feature should do (requirements, rules, edge cases, acceptance criteria).
    - Code: who made the actual code/file changes in this session (use a rough ratio of edits).
    - Overall Steering by User: how much the user directed the plan/sequence/decisions vs the agent deciding the path.
    - User Satisfaction: estimate of user satisfaction with the outcome (100% = explicitly happy/accepted, 50% = acceptable with caveats, 0-25% = unhappy/failed to meet standard).

- "Copy Plan" = copy the current plan file to `others/agent-plans/`
- When copying/duplicating files, ALWAYS use `cp` command. NEVER re-generate file contents using write tool - that wastes expensive output tokens!
- Always append to the chat log at `CHAT-LOG.md` before committing with a concise chronological record of what was done; add a timestamp line (`YYYY-MM-DD HH:MM:SS`) above each `##` section. Sentiment must be determined by skimming all user prompts in the current session.
  - Every chat-log entry must include:
    - Sentiment: `+1` (user explicitly happy/approves), `0` (neutral or no explicit feedback), `-1` (user expresses unhappiness/frustration or rejects quality).
    - Outcome: `Complete` (meets standard), `Partial` (some accepted but missing pieces), `Blocked` (cannot proceed), or `Rework-needed` (output rejected or below standard).
- After updating the chat log and before committing, read `PENDING.md`: (1) strike completed tasks and append `(Completed YYYY-MM-DD; verified YYYY-MM-DD: <short note>)`, (2) add any new pending tasks you just realized with the chat-log timestamp.
- Git workflow: this repo is checked out as a git worktree; work only on branch `codex` and push only to `origin/codex`.
- Always stage files using `git add -A` (never stage individual files). This ensures all changes including package.json, pnpm-lock.yaml, and other modified files are included.
- When starting a very complicated task (multi-file, multi-step), skim all commit messages to get context and steer accordingly.
- Reference docs for scope and direction:
  - `PROJECT_ROADMAP.md` (note the exact filename): Source of truth for scope, priorities, and task checklists. Map new work to a milestone task (add if missing) and mark `[x]` on completion. Update the "Last updated" line whenever any checklist item changes.
  - `PROJECT.md`: Idea dump and long-term vision, only reading is allowed; never edit this file itself. Promote items into `PROJECT_ROADMAP.md` before implementing when work is multi-file, touches DB/schema, or creates a new UI surface.
  - `mvp-database.sql`: SQL mirror for Drizzle queries and planned schema additions; keep it synced.
- Keep the SQL reference and roadmap in sync before any commit:
  - `mvp-database.sql`: Add/update exact SQL equivalents for any new/edited Drizzle queries or schema changes (Section 4 for query patterns, Sections 2-3 for new tables/indexes). Include file path comments for each query so it stays traceable.
  - `PROJECT_ROADMAP.md`: Mark completed tasks with `[x]` before committing. If the task is missing, add it under the right milestone/subsection as `[ ]`, then mark `[x]` when done.
- Claude, DO NOT INCLUDE shit like " 🤖 Generated with [Claude Code](https://claude.com/claude-code)
Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com> " in the commit messages!!!

## Project Structure & Module Organization
- `app/`: Next.js App Router entry; contains `layout.tsx`, `page.tsx`, and `globals.css` (Tailwind v4 theme + CSS vars). Add new routes as folders with `page.tsx` and co-locate supporting pieces.
- `app/profile/page.tsx`: Signed-in profile view (client) that reads the Better Auth session and calls Better Auth endpoints like `GET /api/auth/list-sessions` for active sessions.
- `components/ui/`: Reusable primitives built with Radix Slot + class-variance-authority. Extend `Button` or `buttonVariants` instead of duplicating class strings.
- `lib/utils.ts`: Shared helpers such as `cn` for class merging; keep utilities small and cross-cutting.
- `lib/db/client.ts`: Central Drizzle + Postgres singleton for better-auth + app queries; reuse the pooled Postgres-js connection and keep this file server-only (never import from client components).
- `public/`: Static assets served from `/`; place any new icons or marketing images here.
- `components.json` and Tailwind config files define design tokens; keep new colors/radii aligned with existing variables.

## Sniper Data (sniper_data schema)
- `portfolio_account`: Broker/account container per user (Zerodha/IBKR/etc.) with optional provider metadata.
- `holding`: Current position per account + symbol with `avg_price`, `quantity`, `asset_class`, and optional `purchased_on`.
- `transaction`: Multiple buys/sells and other events per account/symbol for history.
- `net_worth_snapshot`: Daily totals for charting (`total_invested`, `total_value`, `total_cash`, `base_currency`).
- Cash should be modeled as `asset_class = "cash"` holdings (e.g., symbol/currency match) and included in snapshots.

## Build, Test, and Development Commands
- `pnpm install` installs dependencies (pnpm is preferred because the lockfile is present).
- `pnpm dev` runs the dev server at http://localhost:3000 with hot reload.
- `pnpm lint` executes the Next.js + TypeScript ESLint rules (core-web-vitals).
- `pnpm build` produces the production bundle; Dont run without explicit request from user, or before taking permission. That too only if you've made a super refactor or major changes/updates. 
- `pnpm start` serves the last build for smoke testing.
- Drizzle: `pnpm db:generate` creates SQL from `lib/db/schema.ts`, `pnpm db:push` applies it to Supabase, `pnpm db:studio` opens the Drizzle viewer. Copy `env.example` to `.env.local` for connection secrets.
 
## Local Turso (Historical Prices)
- Local Turso DB is created and filled using `others/python/fillDb.py`.
- It contains historical close prices for all stocks in exchanges `NSI` and `NMS`.
- Run the local server with `turso dev --db-file others/tursodata/local.db` so it is queryable via Drizzle and locally.
- In a separate shell, connect with `turso db shell http://127.0.0.1:8080`.

## Coding Style & Naming Conventions
- TypeScript, Next.js App Router, React function components; 2-space indent, trailing commas, and no semicolons to match existing files.
- Use `PascalCase` for components, `camelCase` for functions/vars, and lowercase route segment folders.
- Favor Tailwind utilities; merge classes with `cn` and model multi-state styles with `cva` variants.
- Keep shared styles in `app/globals.css` or component-scoped classes; prefer design tokens over ad-hoc hex values.

## User Preferences
- Tech stack is fixed: React, Next.js, TailwindCSS, ShadcnUI, better-auth, Supabase (Postgres), pnpm, TypeScript. Do not swap libraries.
- When adding components/blocks/charts/themes, prefer Shadcn defaults or use Registry entries; avoid bespoke styling unless necessary.
- Keep `components/` organized with subfolders by domain; avoid scattering files at the root.
- Place shared utilities, auth, db, and cross-cutting helpers in `lib/`; avoid duplicating logic in components.
- Database + auth note: `lib/db/client.ts` exists to share a single Drizzle/Postgres connection across the app and better-auth; do not create ad-hoc clients elsewhere.
- Auth client note: `lib/auth-client.ts` expects `NEXT_PUBLIC_BETTER_AUTH_URL` to be set (or same-origin); reuse its exports (`signIn.email`, `signUp.email`, `signOut`, `useSession`) in client components.

## Testing Guidelines
- Automated tests are not yet configured; when adding them, prefer React Testing Library with a lightweight runner (Vitest/Jest) and colocate `.test.tsx` files near components or under `__tests__/`.
- Cover user-visible behavior and accessibility states; test rendered output rather than implementation details.
- Add a `pnpm test` script alongside new tests and ensure it passes before opening a PR.

## Commit & Pull Request Guidelines
-- Commit rules live in **Most Important** (do not duplicate here).
- PRs: include a brief summary, linked issue/task ID, and before/after screenshots for UI changes.
- If you run `pnpm lint`, note any expected failures in the PR description.
- Document new env vars or migrations; keep secrets out of Git and store local values in `.env.local` (prefix browser-exposed ones with `NEXT_PUBLIC_`).
- Communication: After any code edit, summarize the change concisely (what/why/where) so reviewers/users know exactly what was done.
- Never try to open or ask to open a PR. The user works solo, with agents on this project!
