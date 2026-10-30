# PROMPT 5 of 5 — `admin.aptohire.com` (Admin console)

> Paste everything below this line into your AI coding tool inside an **empty folder** named `aptohire-admin/`.
> This project is **independent**. It does not import from, or share code with, the other portals. Where this prompt says "copy", copy the file; never create a cross-project import.

---

## 0. Your role and how to work

You are a senior frontend engineer and product designer. Build the **Admin console of AptoHire** (`admin.aptohire.com`) for AptoHire staff, as a complete, production-quality Vite + React + TypeScript app that talks to the **shared Supabase backend**.

This is the **most privileged surface in the system**. It approves universities and organizations, reviews borderline AI results, decides appeals, watches the AI agents, and holds the kill switch. Everything below is designed to make that power **safe, auditable and fair**: MFA is mandatory, dangerous actions need step-up confirmation, reviewers work **blind to student identity**, and there is **no impersonation**.

Work in this order:

1. **Plan (short).** Write `docs/PLAN.md` (max 1 page): route list, feature modules, the `Backend` interface (4.1), what you will mock. Then build.
2. **Build** in the phases of section 15. After each phase run `npm run typecheck && npm run lint && npm run test` and fix failures before moving on.
3. **Verify** against section 16 at 360, 390, 768, 1024 and 1440 px.
4. **Report** in the exact format of section 17.

Rules for decisions:

- Do not ask me questions. If something is unclear, pick the safe default, leave `// TODO(decision): …`, and list it in the report.
- Never invent product behaviour or numbers. Anything shown that isn't real data must be labelled **"Sample"**.
- **All people, universities and organizations are fictional.**
- The backend (`aptohire-backend/`) may not exist yet. **The app must run completely in mock mode** (`VITE_USE_MOCKS=true`) and switch to the real backend by changing one env value.
- **Exact backend object names come from `docs/CONTRACT.md`** (the user copies it here from `aptohire-backend`). If it differs from the names in this prompt, **the contract wins**; adapt only the `supabase/` implementation and mappers, never the UI.

---

## 1. Product context (admin view)

**AptoHire — "Hired for what you can do."** Students prove skills by building real projects; an AI review issues a **verified scorecard**; organizations find students **without seeing names** until the student says yes. **AI proposes, rules decide.** The admin console is where humans stay in the loop.

What staff do here (features F-AD-01 → F-AD-11):

1. **Approve or reject university applications** (F-AD-01). Approving a university **unlocks its roster**: students can register against it and pending roster matches are created.
2. **Approve organizations** (F-AD-02), grant the **Verified organization** badge, and **toggle the subscription** (prototype billing switch).
3. **Review borderline results** (F-AD-03): audits scoring **40–60** are held as "Under human review" and are **not shared with organizations** until a human **confirms or adjusts** the score.
4. **Decide appeals** (F-AD-04): acknowledge, **assign a *different* reviewer than the one who confirmed the original result**, decide (**Upheld / Overturned / Adjusted**) and record the reason. Outcomes feed the **rubric improvement log**.
5. **Monitor agent runs** (F-AD-05): status, stage, latency, mode (`mock` / `direct` / `n8n`), errors, estimated tokens; **retry** failed runs.
6. **Kill switch and configuration** (F-AD-06): agent mode, `PASS_MARK`, borderline range, `REQUIRE_VIDEO`, upload limits, daily AI caps, and the **AI kill switch**.
7. **Skill taxonomy and project template library** (F-AD-07, P1).
8. **Audit log viewer** (F-AD-08, P1): sensitive reads, identity reveals, grants, admin actions. **Append-only; no edit or delete.**
9. **Users and roles** (F-AD-09, P1): **read-mostly. Impersonation is not allowed, ever.**
10. **Seed/reset demo data** (F-AD-10): **non-production only.**
11. **AI-vs-human agreement report** (F-AD-11, P2), feeding the ≥ 80% agreement target.

Levels: **Emerging** (0–50, not verified) · **Competent** (51–69) · **Proficient** (70–84) · **Advanced** (85–100). `verified = score > PASS_MARK` (config, default 50). **Borderline** = score within the config range (default 40–60).

**Principles this console must respect (never relax):**

- **Fairness in review:** review and appeal screens are **identity-blind** (no student name, email or university). Reviewers judge the work, evidence and rationale.
- **Least exposure:** show contact details only where a decision needs them (applications); never show raw student ID numbers (they don't exist) or any secret.
- **Every consequential action is explained before it happens** (what will change, who is affected) and **recorded** (the backend writes `audit_log`).
- **The UI never enforces permissions**; RLS and Edge Functions do, and the backend requires MFA (`aal2`). The UI mirrors that for usability.

---

## 2. Where this project fits

AptoHire is **five separate web projects sharing one Supabase database**:

| Project | Host | Port (local) |
|---|---|---|
| `aptohire-root/` | `aptohire.com` | 5173 |
| `aptohire-student/` | `student.aptohire.com` | 5174 |
| `aptohire-university/` | `university.aptohire.com` | 5175 |
| `aptohire-organization/` | `organization.aptohire.com` | 5176 |
| **`aptohire-admin/` ← YOU ARE BUILDING THIS** | `admin.aptohire.com` | **5177** |

`aptohire-backend/` owns migrations, RLS, seed data and Edge Functions. **This project never edits any of that.** It consumes the database with the anon key, the admin's JWT (**`aal2`**), RLS, Storage (signed URLs), Realtime and the `admin-*` Edge Functions in section 4.

If `../aptohire-university` or `../aptohire-organization` exists you may **read** it and **copy** (never import): `src/styles/tokens.css`, `src/components/proof/ProofRing.tsx`, `StatusChip.tsx`, `src/components/brand/*`, `src/components/shell/*` patterns, `DataTable`, `src/lib/supabase/client.ts` pattern. If not, build from sections 6–7.

**Hard boundaries**

- **Admin accounts are created manually, never through sign-up.** This app has **no sign-up route**. Role assignment is server-side; the client can never set a role.
- **The admin portal is never linked from any other project**, and this project must not advertise itself. Security rests on authentication + MFA, **not** on the URL being secret.
- **Impersonation is not built, not stubbed, not hinted at.** Nothing in the UI lets an admin act as another user.
- **The service-role key must never exist in this repo.** Only `VITE_*` values.
- A signed-in user whose role is not `admin` sees a plain refusal and **no admin data is fetched** (section 5). **No admin data hook may run before the role guard and the MFA gate have both passed.**

---

## 3. Tech stack and project setup

| Layer | Choice |
|---|---|
| Build | **Vite + React 18 + TypeScript (strict, no `any`; `unknown` + Zod)** |
| Routing | React Router (data router); every route `React.lazy` except shell and login |
| Data | **TanStack Query** + `@supabase/supabase-js` (used **only** in `src/lib/backend/supabase/`) |
| Tables | `@tanstack/react-table` (headless), our own styling |
| Forms | React Hook Form + **Zod** |
| Styling | **Tailwind CSS 3.4 (pinned)** + CSS variables (section 6) + **shadcn/ui** (Radix); install only primitives you use |
| Icons | `lucide-react`, stroke 1.75, 20 px |
| Fonts | Self-host via Fontsource: `@fontsource-variable/bricolage-grotesque`, `@fontsource-variable/instrument-sans`, `@fontsource/jetbrains-mono` (400) |
| Charts | Plain SVG/CSS only (stat bars, latency strip). **No chart library** in this app |
| Motion | No animation library. CSS only; Proof Ring reveal on the review page |
| Head | `react-helmet-async` (`noindex`, `Cache-Control: no-store` via headers) |
| Tests | Vitest + React Testing Library, Playwright + `@axe-core/playwright` |
| Lint | ESLint (typescript-eslint, react-hooks, jsx-a11y) + Prettier |

### Scripts

`dev` (port **5177**, `--strictPort`), `build`, `preview`, `typecheck`, `lint`, `test`, `e2e`, `db:types` (`supabase gen types typescript --project-id $SUPABASE_PROJECT_REF > src/lib/database.types.ts`; committed; until it exists use hand-written domain types).

### Environment (`.env.example` committed; `.env.local` ignored)

```
VITE_APP_ENV=local                  # local | demo | production
VITE_DEMO_MODE=true                 # demo panel + prototype notice + demo data tab
VITE_USE_MOCKS=true                 # true = in-memory backend, no network
VITE_SITE_URL=http://localhost:5177
VITE_ROOT_URL=http://localhost:5173
VITE_ROOT_DOMAIN=localhost
VITE_STUDENT_URL=http://localhost:5174
VITE_UNIVERSITY_URL=http://localhost:5175
VITE_ORGANIZATION_URL=http://localhost:5176
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=             # public anon/publishable key ONLY
VITE_DEMO_PASSWORD=                 # demo/local only; fictional accounts
VITE_IDLE_TIMEOUT_MINUTES=30
VITE_SUPPORT_EMAIL=support@aptohire.test
```

`src/config/env.ts` parses with Zod at startup and throws a readable error. **Build fails** if `VITE_APP_ENV=production` and `VITE_USE_MOCKS` or `VITE_DEMO_MODE` is true, or `VITE_DEMO_PASSWORD` is set.

`src/config/links.ts` exports `portalUrl('student' | 'university' | 'organization', path)` (explicit env first, else `https://{portal}.${VITE_ROOT_DOMAIN}`) and `rootUrl(path)`. Used **only** by the wrong-portal screen. **No other code builds cross-site URLs.**

### Folder structure

```
aptohire-admin/
├─ README.md  .env.example  index.html  package.json  vite.config.ts  tailwind.config.ts
├─ vercel.json               # rewrites + strict security headers (section 12)
├─ public/                   # favicon.svg, robots.txt (Disallow: /)
├─ docs/PLAN.md
├─ src/
│  ├─ main.tsx
│  ├─ app/                   # router.tsx, providers.tsx, AuthGate.tsx, RoleGuard.tsx, MfaGate.tsx, IdleGuard.tsx, ErrorBoundary.tsx
│  ├─ config/                # env.ts, links.ts
│  ├─ components/
│  │  ├─ shell/              # AppShell, SideRail, TopBar, BottomTabBar, MoreSheet, AccountMenu, NotificationBell, SkipLink, PrototypeNotice, KillSwitchBanner, CountBadge
│  │  ├─ data/               # DataTable, FilterBar, StatRow, SlaChip, JsonViewer, SampleDataBadge
│  │  ├─ dialogs/            # StepUpDialog, TypedConfirmDialog, ReasonDialog, ExternalLinkDialog, IdleTimeoutDialog
│  │  ├─ approvals/          # ApplicationSheet, ChecksPanel, ApprovalActions, SubscriptionToggle
│  │  ├─ review/             # ReviewWorkbench, AiAssessment, CriterionRow, EvidencePanel, VideoTranscript, CodeExcerpt, DecisionBar, ConsequencePreview
│  │  ├─ tickets/            # TicketTimeline, AssignReviewer, DecideTicketDialog
│  │  ├─ runs/               # AgentStats, RunSheet, RunStatusChip
│  │  ├─ settings/           # ConfigField, KillSwitchPanel, LevelsPreview
│  │  ├─ proof/              # ProofRing, StatusChip
│  │  ├─ brand/              # Logo, SealMark
│  │  └─ ui/                 # Button, Field, Input, Select, Textarea, Checkbox, Switch, Tabs, Dialog, Sheet, Toast, Skeleton, EmptyState, SegmentedControl, ...
│  ├─ features/              # api hooks (use*.ts), components, schemas per area
│  │  ├─ auth/ mfa/ approvals/ reviews/ tickets/ runs/ config/ library/ auditlog/ users/ reports/ notifications/ counts/ account/
│  ├─ pages/                 # thin route components
│  ├─ lib/
│  │  ├─ backend/            # types.ts (Backend interface), index.ts (factory), supabase/, mock/
│  │  ├─ supabase/client.ts  # the ONLY createClient call
│  │  ├─ domain/             # types.ts, mappers.ts
│  │  ├─ schemas/            # zod for every response, form and config key
│  │  ├─ sla.ts  levels.ts  consequence.ts  agreement.ts  configRules.ts  csvExport.ts  mfa.ts  idle.ts  format.ts  cn.ts
│  └─ styles/                # tokens.css, globals.css
├─ tests/                    # vitest
└─ e2e/                      # playwright
```

`levels.ts`: `levelFor(score)`, `isVerified(score, passMark)`, `isBorderline(score, [low, high])`. **The pass mark and borderline range always come from `app_config`** (via `public_config`/admin config); constants are fallbacks only, commented *"Mirror of app_config. Keep in sync."*

---

## 4. Backend contract (what this app consumes)

### 4.1 One interface, two implementations

Define `Backend` in `src/lib/backend/types.ts`, grouped (`auth`, `mfa`, `approvals`, `reviews`, `tickets`, `runs`, `config`, `library`, `auditLog`, `users`, `reports`, `notifications`, `counts`, `account`, `demo`) with **Zod-validated** inputs and outputs. Two implementations: `supabase/` (real) and `mock/` (in-memory, 4.6). The factory picks by `VITE_USE_MOCKS`, provided through a `BackendProvider`. **Components never import supabase or call fetch**; they use feature hooks on TanStack Query. **Decisions use pessimistic updates** (wait for the server, then invalidate); never optimistic for approvals, reviews, tickets, config or retries.

### 4.2 Data access (RLS enforces `is_admin()` and, per the backend, `aal2`)

| Object | Admin can | Used for |
|---|---|---|
| `profiles` | read own (role) and list staff | role guard, Users |
| `applications` (admin-only) | read | Approvals: `id, reference, type, subject_id, contact_name, contact_role, contact_email, contact_phone, payload, status, created_at` |
| `universities`, `organizations` | read in full | Approvals detail: `status, kind, tier, email_domains, subscription_status, verified_at…` |
| `roster_entries` | **count only** (head queries filtered by university) | "N students waiting" on a pending university. Never fetch rows |
| `audits`, `submissions` | read | Reviews and Tickets (identity columns are never rendered) |
| `tickets` | read | Tickets |
| `agent_runs` | read | Agent runs |
| `audit_log` | read (append-only) | Audit log |
| `app_config` | read (writes only via `admin-update-config`) | Settings |
| `skills`, `skill_edges`, `project_templates`, `rubrics` | read/write per RLS (rubrics **read-only** in this phase) | Library |
| `notifications` | read own; update `read_at` | bell and centre |
| `public_config` | read | pass mark and borderline range for previews |
| Storage `videos`, `documents` | **signed URLs on demand** (10 minutes, never stored) | Review evidence |

Admin helper views (names per `docs/CONTRACT.md`) may exist for convenience (`admin_borderline_v`, `admin_tickets_v`, `admin_agent_runs_v`, `admin_audit_log_v`, `admin_counts_v`, `admin_agent_stats_v`, `admin_agreement_v`, `admin_users_v`). **Use them when present; otherwise compute client-side from the fetched window and mark `TODO(backend)`.** Never select `profile_sources.token_secret_id`.

### 4.3 Edge Functions (admin JWT with `aal2`; base `${VITE_SUPABASE_URL}/functions/v1`)

Errors: `{ error: { code, message, fields? } }`. Map every code to calm copy (section 9); never show raw messages. `403 MFA_REQUIRED` → send the user to the MFA challenge and retry once after success.

| Function | Request → Response |
|---|---|
| `admin-approve-university` | `{ university_id, decision: 'approve' \| 'reject', note? }` → `{ status }`. Approve sets `approved`, runs the deferred roster matching, notifies members. Reject requires a note |
| `admin-approve-org` | `{ organization_id, decision: 'approve' \| 'reject', note? }` → `{ status, verified_at? }` |
| `admin-set-subscription` | `{ organization_id, status: 'active' \| 'inactive' }` → `{ subscription_status }` |
| `admin-review-audit` | `{ audit_id, decision: 'confirm' \| 'adjust', adjusted_scores?: { skill_id, score }[], note }` → `{ review_status }`. Re-applies scoring; clears the hold; notifies the student |
| `admin-decide-ticket` | `{ ticket_id, action: 'ack' \| 'assign' \| 'decide', assigned_to?, decision?: 'upheld' \| 'overturned' \| 'adjusted', note?, adjusted_scores?, criterion? }` → `{ status }`. **The backend rejects a decision by the reviewer who confirmed the original review (`FORBIDDEN`/`CONFLICT`).** `criterion` is optional (rubric log) |
| `admin-update-config` | `{ key, value }` → `{ key, value, updated_at }` for whitelisted keys: `AGENT_MODE`, `PASS_MARK`, `REQUIRE_VIDEO`, `AI_KILL_SWITCH`, `AI_DAILY_CAP_PER_USER`, `VIDEO_MAX_MB`, `VIDEO_MAX_SECONDS`, `BORDERLINE` (`[low, high]`) |
| `admin-retry-run` | `{ run_id }` → `{ run_id, status }` (failed/timeout runs only) |
| `admin-seed-reset` | `{}` → `204`. **`403` when `APP_ENV=production`** |
| `admin-view-student` (P2) | `{ student_id }` → detail; **audit-logged sensitive read**. Build the UI only if the contract provides it (feature flag `FEATURE_STUDENT_LOOKUP`) |

Domain types (`src/lib/schemas/`; mappers adapt to the contract):

```ts
type ApplicationRow = { id: string; reference: string; type: 'university' | 'organization'; subject_id: string;
  subject_name: string; status: 'received' | 'approved' | 'rejected'; subject_status: 'pending' | 'approved' | 'rejected';
  created_at: string; contact: { name: string; role: string; email: string; phone?: string };
  payload: Record<string, unknown>; locked_roster_count?: number; subscription_status?: 'active' | 'inactive'; verified_at?: string | null };
type ReviewItem = { audit_id: string; submission_id: string; submission_ref: string; title: string;
  skills: { skill_id: string; skill_name: string }[]; overall: number; confidence: number; created_at: string;
  review_status: 'pending' | 'confirmed' | 'adjusted'; reviewed_by?: string | null; reviewed_at?: string | null; flags: string[] };
type TicketRow = { id: string; ref: string; submission_id: string; audit_id: string; skill_names: string[];
  original_score: number; reason: string; status: 'open' | 'ack' | 'in_review' | 'decided';
  assigned_to?: { id: string; name: string } | null; original_reviewer_id?: string | null;
  decision?: 'upheld' | 'overturned' | 'adjusted' | null; decision_note?: string | null; created_at: string; decided_at?: string | null };
type AgentRunRow = { id: string; kind: 'audit' | 'assign' | 'match' | 'jd_extract' | 'profile' | 'embed';
  mode: 'mock' | 'direct' | 'n8n'; status: 'queued' | 'running' | 'succeeded' | 'failed' | 'timeout';
  stage?: string | null; attempt: number; latency_ms?: number | null; tokens_in?: number | null; tokens_out?: number | null;
  error?: string | null; model?: string | null; prompt_version?: string | null; submission_ref?: string | null;
  created_at: string; finished_at?: string | null };
type AuditLogRow = { id: string; created_at: string; actor_role: string; actor_id: string; actor_label?: string | null;
  action: string; target_type: string; target_id: string; details: Record<string, unknown> };
```

The review workbench also needs the `Audit` type (summary, `per_skill[]` with criteria `{name, weight, score|null, rationale, evidence_refs, insufficient_evidence?}`, `evidence[]` `{ref, kind: 'video'|'file'|'repo'|'doc'|'live', label, pointer, excerpt?}`, `transcript?: {t, text}[]`, `confidence`, `flags`, `next_steps`, `borderline`, `review_status`, `prompt_version`) and submission fields (title, description, `live_url`, `github_url`, file names, `ai_use` disclosure, `originality_answers`, video path or `video_url`). **Never map or render** the student's name, email, university or `student_id`.

### 4.4 Realtime and counts

Subscribe (RLS-filtered) to `notifications` (own) and invalidate affected queries. **Queue counts** for the rail badges (pending approvals, borderline reviews pending, open tickets, failed/timeout runs in the last 24 h) come from `admin_counts_v` or head-count queries, **polled every 60 s** and on window focus. The Agent runs page offers an **auto-refresh (10 s) toggle**, off by default. Always unsubscribe on unmount.

### 4.5 Auth and MFA (Supabase Auth TOTP)

Use `auth.mfa.listFactors()`, `enroll({ factorType: 'totp' })`, `challenge`, `verify`, and `getAuthenticatorAssuranceLevel()`. State machine (`lib/mfa.ts`, unit-tested): `no-session → wrong-role | needs-enroll | needs-challenge | ready (aal2)`. Step-up confirmation re-runs `challenge` + `verify` for dangerous actions (section 8.9).

### 4.6 Mock backend (required, in-memory, deterministic)

- State in memory and `sessionStorage` (try/catch). Latency 300–800 ms. **Reset demo data** in demo mode (Settings → Demo data). The mock MFA accepts the code **`123456`** for enroll, challenge and step-up (**mock mode only; never in production code paths**).
- **Accounts:** `admin@demo.aptohire.test` ("Admin 1", the signed-in reviewer), plus mock-only admins `reviewer2@demo.aptohire.test` ("Reviewer 2") and `reviewer3@demo.aptohire.test` ("Reviewer 3") so the **different-reviewer rule** can be demonstrated. Non-admin mock logins (`aarav@unia.edu.test` student, `tpo@unia.edu.test` university, `recruiter@orga.demo.test` organization) exist to show the wrong-role screens.
- **Applications:** pending **Uni C** (`AH-7K2Q9F`, Tier 3, 6 locked roster entries), pending **Orbit Analytics** (`AH-4M8Z2D`, organization), pending **Riverbend College** (email domain `gmail.com`, flagged by the checks), pending **Quick Hire Co** (contact on a free-mail address, website domain mismatch, flagged). Approved: Uni A, Uni B, **Northwind Labs**, **Brightlane**, **Cedar Works** (subscription inactive). One **rejected** application with a note.
- **Reviews:** 5 borderline items (**Naveen's React 52** among them) with varying ages (one **overdue**), full `Audit` with transcripts, evidence and flags (one with `possible_copied_work`, one with `prompt_injection_attempt`), plus ~30 human-decided reviews for the agreement report.
- **Tickets:** one `open` (on a seeded scored submission), one `ack`, one `in_review` assigned to Reviewer 2, one `decided` (upheld), and one whose **original review was confirmed by Admin 1** (so Admin 1 cannot decide it).
- **Agent runs:** ~120 across kinds/modes with a mix of `succeeded`, `failed`, `timeout`, `running`; latencies and token estimates; a few `[fail]`-style errors.
- **Config** defaults: `AGENT_MODE='mock'`, `PASS_MARK=50`, `BORDERLINE=[40,60]`, `REQUIRE_VIDEO=true`, `VIDEO_MAX_MB=50`, `VIDEO_MAX_SECONDS=190`, `AI_DAILY_CAP_PER_USER=5`, `AI_KILL_SWITCH=false`.
- **Audit log:** ~200 entries across reveals, grants, withdrawals, approvals, config changes, ticket decisions and role settings (ids and labels only).
- **Library:** the starter taxonomy (Web/Frontend, Backend, Data, Management/Commerce) with edges and project templates for Web/Frontend only.
- Mock functions mutate the store, **append to the mock audit log**, push notifications, and apply the same validation and rules as the real backend (including the different-reviewer rule and `FORBIDDEN` when `APP_ENV=production` for seed-reset). `admin-retry-run` moves a failed run to `queued` → `running` → `succeeded` over ~6 s.
- Times are relative to *now* so demos stay fresh.

### 4.7 What the backend project must provide (list in your report; treat each as optional at runtime)

`is_admin()` policies that **also require `aal2`** · an admin-readable list of staff for reviewer assignment (`admin_users_v` with `user_id, display_name, role`) · optional `criterion` on `admin-decide-ticket` · applicant-visible rejection `status_note` · `admin_counts_v`, `admin_agent_stats_v`, `admin_agreement_v` (else the client computes from a bounded window) · `admin_audit_log_v` with actor labels · admin signed-URL access to `videos` and `documents` · admin write access to `skills`, `skill_edges`, `project_templates` with the exact columns · who last changed each `app_config` key · `admin-view-student` (P2) · user-management functions (disable user, change role, invite) if wanted (**not built in this phase; the Users page is read-only**) · demo logins for extra reviewers (`reviewer2@…`, `reviewer3@…`) in the seed · the exact shapes of the admin views. If something is missing: hide the dependent UI, keep the rest working, and mark `TODO(backend)`.

---

## 5. Auth, MFA, sessions and gates

- **Supabase Auth, email + password, then TOTP.** Single client in `src/lib/supabase/client.ts`: `storageKey: 'aptohire-admin'`, `persistSession`, `autoRefreshToken`, `detectSessionInUrl`, `flowType: 'pkce'`. **No cookie shared across subdomains.** There is **no sign-up route**.
- **Public routes:** `/login`, `/forgot-password`, `/reset-password`, `/auth/callback`. Everything else is behind gates **in this order: `AuthGate` (session) → `RoleGuard` → `MfaGate` → app**. The role check comes **before** any MFA prompt so a non-admin is never asked to enrol.
- **`RoleGuard`:** fetch **only** the user's own `profiles` row. `admin` → continue. `student`, `university` or `organization` → "This account can't use the admin site." with a **Go to your portal** button (`portalUrl(role)`) and **Sign out**. Any other/unknown role → the same message with no link. **No admin data hook may run before this passes** (tested).
- **`MfaGate`:** if the admin has **no verified TOTP factor** → forced **`/mfa/setup`**: show the QR (the `qr_code` data URI in an `<img>`), the secret in mono with a copy button, a 6-digit field; verify → `aal2`. Copy: "Two-step verification is required for admin accounts. If you lose your authenticator, another admin must reset it. Save the secret somewhere safe." If a factor exists and the session is `aal1` → **`/mfa/verify`**: a 6-digit field (`inputMode="numeric"`, `autocomplete="one-time-code"`, auto-submits on the 6th digit), clear error and cool-down messages, **Use a different account** (sign out). Only at `aal2` does the app mount. A `403 MFA_REQUIRED` from any function re-routes here and retries once after success.
- **Idle timeout (`IdleGuard`, `lib/idle.ts`):** sign out after `VITE_IDLE_TIMEOUT_MINUTES` (default 30) without input; at 2 minutes before, an `IdleTimeoutDialog` ("You'll be signed out in 2 minutes. **Stay signed in**" / **Sign out now**). Sign-out clears the Query cache, Realtime channels and any signed URLs held in memory.
- **Sessions:** account page offers **Sign out of all devices** (`signOut({scope:'global'})`).
- **Login page:** email + password, "Forgot password", and **no link to any other portal or sign-up**. In demo mode a **Demo account** panel with a button for `admin@demo.aptohire.test` (mock mode signs in directly; **in real mode it only prefills the email**; MFA is never bypassed). The panel and its code are excluded from production builds.
- Never reveal whether an email exists; failed logins show the same message ("Email or password is incorrect.").

---

## 6. Design system (identical across all five portals)

**Concept: "Proof you can see."** Quiet, white, spacious; evidence is the hero. Green = verified, trusted, done. Orange = attention, in progress. For the admin console the character is **utilitarian**: dense, calm, predictable, built for people who make decisions all day. **Flat `mist` strip, green only; orange appears solely for warnings and pending states; red solely for destructive actions.**

### 6.1 Tokens → `src/styles/tokens.css` (mirror in `tailwind.config.ts`)

```css
:root {
  --canvas:#ffffff; --mist:#f6f8f7; --hairline:#e3e8e5; --ink:#17261e; --moss:#5b6a62;
  --verdant-50:#ecfaf2; --verdant-100:#d2f3e1; --verdant-300:#8fe0b2;
  --verdant-600:#0e9f5b; --verdant-700:#0b7f49; --verdant-800:#0a6139;
  --ember-50:#fff3e8; --ember-100:#ffe2c7; --ember-300:#ffb470; --ember-500:#ff7a1a; --ember-700:#b94a06;
  --danger-600:#c4311f; --danger-50:#fdece9;
  --grad-verified: linear-gradient(135deg,#0e9f5b,#34c98a);
  --r-control:10px; --r-panel:16px; --r-feature:28px;
  --shadow-1: 0 1px 2px rgba(23,38,30,.06), 0 8px 24px -10px rgba(23,38,30,.14);
  --shadow-2: 0 2px 4px rgba(23,38,30,.06), 0 24px 48px -16px rgba(23,38,30,.22);
  --font-display:"Bricolage Grotesque Variable","Bricolage Grotesque","Segoe UI",system-ui,sans-serif;
  --font-body:"Instrument Sans Variable","Instrument Sans","Segoe UI",system-ui,sans-serif;
  --font-mono:"JetBrains Mono",ui-monospace,monospace;
  --ease-out:cubic-bezier(.2,.8,.2,1); --dur-fast:150ms; --dur-base:220ms; --dur-reveal:900ms;
  --portal-strip: var(--mist);   /* admin identity: flat, no gradient */
}
@media (prefers-reduced-motion: reduce){ *{animation-duration:.001ms!important;transition-duration:.001ms!important;scroll-behavior:auto!important} }
```

Tailwind names: colors `canvas, mist, hairline, ink, moss, verdant.{50,100,300,600,700,800}, ember.{50,100,300,500,700}, danger.{50,600}`; radius `control, panel, feature`; shadow `float` (shadow-1), `modal` (shadow-2); fonts `display, body, mono`. **No hard-coded hex in components.** Light theme only. **This portal uses no gradient surface at all** (the only gradient is the Proof Ring arc and the verified seal).

### 6.2 Rules

- **Colour:** primary button = **white on `verdant-700`**. **No accent (orange) buttons in this portal.** Destructive buttons (Reject, Pause AI, Reset) = white on `danger-600` (contrast ≥ 4.5:1; verify) in a confirm dialog only, never as a first-click action. `verdant-600` / `ember-500` for graphics and large elements only; small green text `verdant-700`, small orange text `ember-700`. Colour is never the only signal (icon or word alongside).
- **Type:** Bricolage Grotesque (headings 600–700; large numbers 600), Instrument Sans (UI), JetBrains Mono for references, ids, JSON, code, timestamps. Sentence case; no tracked all-caps labels; no eyebrow labels unless they carry information; `tabular-nums` everywhere numbers appear. Lines ≤ 68 chars in prose. Left-aligned.

| Token | Size | Line-height |
|---|---|---|
| `h1` | `clamp(26px, 3.4vw, 32px)` | 1.2, −0.015em |
| `h2` | `clamp(20px, 2.4vw, 24px)` | 1.3 |
| `h3` | `clamp(17px, 1.8vw, 18px)` | 1.35 |
| `body` | **16px minimum** | 1.6 |
| `small` | **14px minimum** (table cells, helper text) | 1.5 |

- **Density:** table rows **40 px at ≥ 1024**, 48 px on tablets, **56 px as stacked lists on phones**; filter bars compact; but **touch targets stay ≥ 44×44 px on touch widths** and text never drops below 14 px.
- **Spacing** (4 px base): 4 · 8 · 12 · 16 · 24 · 32 · 48. Content max **1440 px**, page padding 24 px desktop / 16 px mobile.
- **Radius:** `control` 10 (inputs, buttons; chips full-pill), `panel` 16 (dialogs, sheets), `feature` unused here.
- **Flat by default** with hairline borders. Shadow only for floating things (`float`: menus, popovers; `modal`: dialogs, side sheets). **No cards**; data lives in tables, lists and split panes with hairlines.
- **Motion:** none except the Proof Ring reveal on the review page and dialogs/sheets opening. Reduced motion → final states instantly.
- **Icons:** Lucide. Empty states: palette circles/arcs on `mist`; no stock photography, no photos of people (the console never shows student photos).

### 6.3 App shell

```
┌────────────┬──────────────────────────────────────────────────────┐
│ ◖ AptoHire │ [☰ at 768–1023]       🔔   Admin 1 · aal2 ▾          │ ← 56px top bar, flat mist
│ admin      ├──────────────────────────────────────────────────────┤
│ Approvals 3│  ▲ AI is paused. New projects and reviews are blocked.│ ← KillSwitchBanner (only when on)
│ Reviews   5│   Page title (h1)                                    │
│ Tickets   2│   One-line description (moss)                        │
│ Agent runs │   ───────────────────────────────────────────────    │
│ Settings   │   content (dense tables / split panes)               │
│ ───────    │                                                      │
│ Library    │                                                      │
│ Audit log  │                                                      │
│ Users      │                                                      │
│ Reports    │                                                      │
└────────────┴──────────────────────────────────────────────────────┘
```

| Width | Behaviour |
|---|---|
| ≥ 1280 | Full 232 px rail; list + detail **split panes** for Approvals, Reviews, Tickets (detail on the right) |
| 1024–1279 | Rail collapses to 72 px icons with tooltips; split panes stack to list → detail page |
| 768–1023 | Rail becomes a **drawer** opened by a hamburger (☰/✕, 44×44, `aria-expanded`, focus trap, Escape closes and returns focus, closes on navigation and on resize ≥ 1024) |
| < 768 | **Bottom tab bar**: Approvals · Reviews · Tickets · Agent runs · **More** (sheet: Settings, Library, Audit log, Users, Reports, Notifications, Account & security, Help, Sign out). Sheets become full-screen |

Primary nav items are **exactly**: Approvals · Reviews · Tickets · Agent runs · Settings; a secondary "System" group holds Library · Audit log · Users · Reports. Each of the first four shows a **`CountBadge`** (with a text alternative, e.g., "3 pending"). Skip-to-content link first. Minimum width **360 px**. Respect `env(safe-area-inset-*)`. Keyboard shortcuts (optional, `?` opens a help dialog): `j`/`k` move through queue rows, `Enter` opens, `Esc` closes. **No shortcut ever triggers an approval, rejection, decision or config change.**

---

## 7. Signature components

### 7.1 `ReviewWorkbench` (the console's signature: side-by-side AI rationale and evidence)

Desktop (≥ 1280): two panes with a sticky **`DecisionBar`** at the bottom. Below 1280 the panes stack (AI assessment, then Evidence), and the bar stays sticky.

```
┌─ AI assessment ───────────────────────┬─ Evidence ─────────────────────────┐
│ Submission #A1B2 · React · 52  ◯ ring │ Submission: title, description,     │
│ Confidence: Medium · Flags: ⚑ …       │ links (confirm before opening),     │
│ Summary (≤ 600 chars)                 │ AI-use disclosure, originality Q&A  │
│ Functionality   ▇▇▇▇▇▁▁ 58  rationale │ ──────────────────────────────────  │
│   [Video 01:42] [src/cart.tsx]        │ Video player ⇄ transcript (synced)  │
│ Code quality …                        │ Code/doc excerpt for the selected   │
│ Next steps (≤ 3)                      │ evidence chip                       │
└───────────────────────────────────────┴─────────────────────────────────────┘
 [ Consequence preview ]   [ Adjust score… ]   [ Confirm score ]
```

- **Identity-blind:** the header shows a **submission reference** (short id) and the title, **never a student name, email, university, photo or `student_id`**. A visible note: "Reviews are blind to the student's identity to reduce bias."
- **`AiAssessment`:** per skill, a 96 px `ProofRing` (state `under-review`), criterion rows (name, weight, score bar, number, rationale, evidence chips). `insufficient_evidence` shows "Not enough evidence to score this". Overall, confidence (High/Medium/Low from `confidence`), flags as **reviewer-only** chips with plain explanations: `possible_copied_work`, `inconsistent_claims`, `prompt_injection_attempt`, `video_unusable`, `repo_unreachable` (never worded as accusations; "Check this").
- **`EvidencePanel`:** chips open the matching item. **Video** → player (signed URL created on demand, 10 minutes) that **seeks to the timestamp**, with the **transcript as selectable text** beside it (current line highlighted, click a line to seek); if no file is available, transcript only. **Repo/file/doc** → path, line range and `excerpt` in JetBrains Mono with line numbers (rendered as text in `<pre>`, never HTML). **Live/GitHub/other links** → shown as text with an **Open** button that triggers the **`ExternalLinkDialog`** ("You're opening a link provided by a student. Open in a new tab?" once per session per host), `rel="noopener noreferrer"`.
- **`DecisionBar`:** **Confirm score** (primary) and **Adjust score…**. Adjust opens a panel: per skill, a numeric field 0–100 (initialised to the AI score) and a required **reason** (≥ 20 characters, ≤ 600). The **`ConsequencePreview`** (always visible, pure `lib/consequence.ts` using config) states in words what will happen: "New score 58 → Competent · Verified. It will become visible to organizations." / "New score 49 → Emerging · Not verified." and whether the result stays in the borderline range (it won't be held again: it becomes final once decided). A **confirm dialog** repeats the consequence; submit calls `admin-review-audit` (pessimistic). On success: toast "Review confirmed." / "Score adjusted." and **auto-advance to the next pending item** (focus moves to the new heading). `409/CONFLICT` ("This review was already decided.") refreshes the queue.

### 7.2 `DataTable` (dense variant)

Built on `@tanstack/react-table`: sticky header on `mist`, 1 px hairlines, 40 px rows at ≥ 1024, sortable columns with `aria-sort`, selectable rows (only where bulk actions exist; there are **no bulk decisions** in this console), keyboard-navigable rows (`j`/`k`, Enter), server-side pagination (50 per page, "Showing 1–50 of 312"), loading skeleton rows, empty and error states with **Retry**, URL-synced filters/sort. **Scrolls inside its own container**; **below 640 px it becomes a labelled list** without losing actions.

### 7.3 `StatusChip` and domain chips

| Chip | Look |
|---|---|
| Approved · Verified organization · Verified | `verdant-100` / `verdant-800`, seal icon |
| Pending · Received · Open · Under review · Running · Queued | `ember-50` / `ember-700` (Running adds a pulsing `ember-500` dot, pulse disabled under reduced motion) |
| Rejected · Failed · Timeout | `danger-50` / `danger-600`, icon (never colour alone) |
| Decided · Succeeded | `verdant-100` / `verdant-800` |
| Acknowledged · In review | `mist` / `ink` |
| Overdue | `danger-50` / `danger-600`, clock icon |
| Due soon (≤ 1 working day) | `ember-100` / `ember-700`, clock icon |
| Mode: mock · direct · n8n | `mist` / `ink`, mono text |
| Subscription: Active / Inactive | Active = `verdant-50`; Inactive = `ember-50` |

### 7.4 Dialogs for dangerous actions

- **`ReasonDialog`:** title, consequence text, required reason textarea (min/max length shown), **Cancel** / **Confirm** (loading keeps label and fixed width).
- **`TypedConfirmDialog`:** the user must type an exact word (e.g., `PAUSE`, `RESET`) before the confirm button enables; the word is shown and announced.
- **`StepUpDialog`:** re-challenges TOTP (6-digit field) immediately before dangerous actions (8.9). Its success is valid for **5 minutes in memory only**, never stored.

### 7.5 `ProofRing`

Used on the review page (96 px per skill). Props: `score`, `skill`, `state: 'under-review'|'verified'|'not-verified'|'loading'`, `size: 56|96`, `level`, `animate?`. Track `hairline`; SVG arc with round caps, gradient `#FF7A1A → #FFB470 → #8FE0B2 → #0E9F5B`, pass-mark tick; `role="img"` with `aria-label="React, score 52 of 100, Competent, under human review"`. Reveal 900 ms once (reduced motion → final). Unit-test the aria-label builder.

### 7.6 Other UI

Buttons (primary / secondary / ghost / danger; height 44 on touch, 40 on desktop; label 15/600; loading keeps label and fixed width), Field (label **always above**, helper 14 px, error with icon, `aria-describedby`), Tabs (underline, 2 px `verdant-700`), `SegmentedControl`, Sheet (side sheet `modal` shadow, full-screen on mobile), Toast (bottom-centre, 4 s, reuses the action's verb), Skeleton (matches layout), EmptyState (one sentence, one button), `StatRow` (plain figure + label inline, hairline dividers), `SlaChip`, `JsonViewer` (read-only, collapsible, **text rendering only**, long values truncated with "Show more", keys sorted), `SampleDataBadge`.

---

## 8. Screens and flows

Routes: `/login` · `/forgot-password` · `/reset-password` · `/auth/callback` · `/mfa/setup` · `/mfa/verify` · `/` → redirect to `/approvals` · `/approvals` (`?tab=universities|organizations&status=pending|approved|rejected`) · `/approvals/:type/:id` · `/reviews` · `/reviews/:auditId` · `/tickets` · `/tickets/:ticketId` · `/runs` · `/runs/:runId` · `/settings` (`?tab=config|demo`) · `/library` (`?tab=skills|templates`) · `/audit-log` · `/users` · `/reports` · `/notifications` · `/account` · `/help` · `*` 404. Every page: `h1`, document title, loading skeleton, empty state, error state with **Retry**, `noindex`. **Document titles and URLs never contain student names** (use refs and ids).

### 8.1 Approvals `/approvals` (F-AD-01, F-AD-02)

Tabs **Universities** · **Organizations**; status filter (default **Pending**). Table: **Reference (mono) · Name · Type details · Contact · Submitted · Age · Status**. Universities: kind, tier, city/state, **locked roster count**. Organizations: kind, size, website, **subscription chip**. Selecting a row opens the **`ApplicationSheet`** (split pane ≥ 1280, page below):

- **Application:** every field from `payload` in a labelled description list (render as text), **contact name, role, email, phone** (shown here because the decision needs them), submitted date, reference with copy.
- **Checks panel (`ChecksPanel`, informational, never auto-decides):** university: email domains syntactically valid; **free-mail domain warning** (e.g., gmail.com in `email_domains`); **possible duplicate** (same name or domain as an existing university); roster size waiting. Organization: **contact email domain vs website domain** match (informational; mismatch shown as "Check this"); **free-mail contact warning**; **possible duplicate** name/website. Each check is a row with an icon and a plain sentence, with "Check this" rather than "Suspicious".
- **Actions** (only for `pending`): **Approve** → confirm dialog with consequence copy: universities "Approving unlocks registration for N rostered students and creates matches with existing profiles." · organizations "Approving gives the Verified organization badge. Turn on the subscription separately." **Reject** → `ReasonDialog` with a required note (≥ 10 chars): "This note may be shown to the applicant." Calls the function pessimistically; toast "University approved." / "Application rejected."; the list refreshes and the next pending item opens.
- **Approved organizations:** a **`SubscriptionToggle`** (Active/Inactive) with confirm ("Turning this off blocks candidate matching and new access requests for Northwind Labs.") → `admin-set-subscription`.
- **Rejected:** shows the note and a **Reconsider** button (opens the Approve confirm). **Approved** applications can't be rejected here (`// TODO(backend)`: revocation).
- Empty (pending): "No applications waiting."

### 8.2 Reviews `/reviews` and `/reviews/:auditId` (F-AD-03)

**Queue:** table of pending borderline audits: **Submission ref · Title · Skills · Score · Confidence · Flags · Submitted · Due**. **`SlaChip`**: due = submitted + **3 working days** (`lib/sla.ts`: Monday–Friday, skips Saturday/Sunday, unit-tested incl. weekend boundaries), shown as "Due 14 Oct", **Due soon** or **Overdue**. Default sort: **most overdue first**. Filters: skill, flags present, overdue only. Tabs: **Pending** (default) · **Decided** (last 30 days, with who decided, confirmed/adjusted, old → new score). Row opens the **`ReviewWorkbench`** (7.1). Keyboard: `j`/`k`, Enter.

Empty: "No reviews waiting."

### 8.3 Tickets `/tickets` and `/tickets/:ticketId` (F-AD-04)

**List:** **Ref · Skill(s) · Original score · Reason (first 80 characters) · Status · Assignee · Age**; filters by status and assignee ("Mine", "Unassigned"). **Detail:** header (ref, status chip, timeline), the **student's reason** (rendered as text), the **original review** read-only (reuse `AiAssessment` + `EvidencePanel`, identity-blind), and a note **"Original review: confirmed by Reviewer 2"** (or "by you").

- **Timeline (`TicketTimeline`):** Ticket received → Acknowledged → Assigned → Decision, with timestamps and actors (labels only).
- **Actions by state:** `open` → **Acknowledge** (`action:'ack'`). `ack` → **Assign reviewer** (`AssignReviewer`: select from admins, **excluding the reviewer who confirmed the original review**; if none are available: "No other reviewer is available. Add another admin first."; you may assign yourself **only if** you weren't the original reviewer). `in_review` → **Decide** (`DecideTicketDialog`): decision **Upheld** (original stands) / **Overturned** / **Adjusted** (per-skill new scores with the same `ConsequencePreview`), required **reason** (≥ 20, ≤ 600), optional **Which criterion needs improvement?** (select from the rubric criteria; sent as `criterion` only if the contract supports it) and a plain note "This is recorded in the rubric improvement log and the student is notified."
- **Different-reviewer rule in the UI:** if the signed-in admin was the original reviewer, **disable Decide** and explain: "You confirmed the original review, so a different reviewer must decide this appeal." The backend enforces it too; map `FORBIDDEN`/`CONFLICT` to the same message.
- Decided tickets are read-only with the decision, reason and decider. Empty: "No appeals waiting."

### 8.4 Agent runs `/runs` (F-AD-05)

- **`AgentStats` strip** (plain figures, hairline-divided; "Based on the last 500 runs" when client-computed): **Runs (24 h) · Failure rate · Median latency · 95th-percentile latency · Estimated tokens today** (vs the daily caps from config when available).
- **Mode and kill-switch state** at the top: current `AGENT_MODE` chip and the kill switch state with a link to Settings.
- **Table:** **Run (short id) · Kind · Mode · Status · Stage · Attempt · Latency · Tokens (in/out) · Submission ref · Started**. Filters: status, kind, mode, date range; **auto-refresh toggle** (10 s, off by default; live region announces "Updated").
- **Run sheet (`RunSheet`):** timeline of stage changes if available, full **error text** (monospace, text only), model, prompt version, input hash, attempts. **Retry** (only `failed` or `timeout`): confirm dialog "Retry this run? It uses the current mode, AI limits and kill switch." → `admin-retry-run` → toast "Run queued." Retrying is disabled with an explanation while the kill switch is on.
- Never display `user_id` or any identity; show the submission reference only.
- Empty: "No runs yet."

### 8.5 Settings → Configuration `/settings?tab=config` (F-AD-06)

One calm page of labelled sections, each with current value, **last changed by/at** (if available), and a **Save** per section. Validation lives in `lib/configRules.ts` (unit-tested) and the server re-validates.

| Key | Control | Rules and copy |
|---|---|---|
| **`AI_KILL_SWITCH`** | `KillSwitchPanel` (danger-bordered) | Off → On: "Pause all AI reviews and project assignments. Students keep their drafts." **Typed confirm `PAUSE` + step-up**. On → Off: confirm + step-up. While on, a persistent **`KillSwitchBanner`** shows on every page |
| `AGENT_MODE` | `SegmentedControl` mock / direct / n8n | Helper: "Real students' reviews never fall back to mock silently." Changing it needs a `ReasonDialog` and **step-up** |
| `PASS_MARK` | number 1–99 | Shows a **`LevelsPreview`** (Emerging ≤ N, Competent N+1–69…) and "Existing skill scores keep their current Verified status until their next review." (`// TODO(backend)`: confirm). **Step-up** + reason |
| `BORDERLINE` | two numbers low < high within 0–100 | Preview "Scores from 40 to 60 are held for human review and not shared with organizations." Step-up + reason |
| `REQUIRE_VIDEO` | switch | Helper "Students must add a demo video or a video link before submitting." |
| `VIDEO_MAX_MB` | number 1–500 | Helper "Check your storage plan's per-file limit before raising this." |
| `VIDEO_MAX_SECONDS` | number 30–600 | Display as m:ss (190 → 3:10) |
| `AI_DAILY_CAP_PER_USER` | number 0–50 | "Reviews and project assignments per student per day." 0 = none allowed |

Each change shows **old → new**, requires a reason where marked, calls `admin-update-config`, then toasts "Setting saved." and shows who/when. Invalid combinations are blocked client-side with plain messages.

### 8.6 Settings → Demo data `/settings?tab=demo` (F-AD-10)

Only when `VITE_APP_ENV !== 'production'`. **Reset demo data** → `TypedConfirmDialog` (`RESET`) + step-up: "This replaces all demo data with the original fictional data. Real data is never touched." Calls `admin-seed-reset`; `FORBIDDEN` → "Reset is turned off in this environment." In production the tab is not rendered.

### 8.7 Library `/library` (F-AD-07, P1)

Tabs **Skills** · **Project templates**. Build after the P0 areas.

- **Skills:** table (**Name · Slug · Family · Related skills · Active**), add/edit in a side sheet: name (2–60), slug (kebab-case, **immutable after create**), family (select), description (≤ 280), **related skills** (edges; pick skills, no self-edge). Deactivate instead of delete when referenced ("This skill is used by 14 scores. Deactivate it instead."). Validation in Zod.
- **Project templates:** table (**Title · Skill · Level · Estimated hours · Active**), editor: title, skill, level (Competent/Proficient/Advanced), scenario seed text, deliverables (list, 3–8), constraints (list), suggested stack (chips), estimated hours, active toggle. A read-only **rubric preview** (criteria and weights; check "weights sum to 100"). **Rubrics are read-only in this phase** (`// TODO(decision)`: editing is risky).
- All writes go through RLS (admin). Warn: "Changes affect new projects and reviews only."

### 8.8 Audit log `/audit-log` (F-AD-08, P1)

Append-only viewer. Banner: "This log can't be edited or deleted." Table: **Time · Actor (role + label) · Action · Target (type + id) · Details**. Filters: action (reveal, grant, withdraw, decline, approval, rejection, config change, ticket decision, review decision, role set, admin view), actor role, date range, target id (exact), free search on labels. Server-side pagination (50), newest first. A row opens a sheet with `JsonViewer` of `details` (text only). **Export CSV** of the filtered view with the formula-injection guard (prefix `'` for cells starting with `=`, `+`, `-`, `@`, tab, CR), UTF-8 BOM, and a confirmation ("This file contains administrative activity. Store it securely."). No mutation controls anywhere on this page.

### 8.9 Step-up and dangerous actions

Step-up (`StepUpDialog`) is required for: **kill switch on/off, `AGENT_MODE`, `PASS_MARK`, `BORDERLINE`, seed reset**. Valid 5 minutes in memory (cleared on sign-out). Typed confirmation is additionally required for the kill switch and seed reset. Never use step-up for ordinary decisions (approvals, reviews, tickets).

### 8.10 Users `/users` (F-AD-09, P1) — read-mostly

Tabs **Staff** (admins, university and organization members) and, only if `FEATURE_STUDENT_LOOKUP`, **Students**. Staff table: **Name/label · Role · University or organization · Created · Last sign-in (if available)**; filters by role. Banner: **"AptoHire doesn't allow impersonation."** All management actions (invite, disable, change role) are **absent in this phase** with a quiet note `TODO(backend)` (do not render dead buttons). Students tab (only with the function): search by **credential ID or email**; results show credential id and status only; **Open** calls `admin-view-student` (audit-logged) with a notice "This view is recorded in the audit log."

### 8.11 Reports `/reports` (F-AD-11, P2)

**AI-vs-human agreement**, computed over the last N (default 100) human-decided reviews (view or client-side, `TODO(backend)`): plain `StatRow` (**Agreement rate · Reviews counted · False-pass rate · Average adjustment**), a plain progress bar toward the **80% target** (label with words), and a table of recent decisions (ref, AI score, human score, outcome; **no identities**). `lib/agreement.ts`: a review **agrees** when it was **confirmed**, or **adjusted by ≤ 5 points** (constant, `// TODO(decision)`); **false pass** = AI said verified but the human result is below the pass mark. Under 30 reviews show "Based on a small sample. Treat this as indicative." Export CSV with the guard.

### 8.12 Notifications, account, help, system screens

- **Notifications:** bell with unread count (`aria-label="Notifications, 3 unread"`) opening a popover (latest 8) with **Mark all as read** and **View all**. Kinds and deep links: `application_received` → Approvals · `ticket_opened` → Tickets · `review_needed` → Reviews · `run_failed` → Agent runs. Realtime inserts show a toast. Empty: "You're all caught up."
- **Account `/account`:** email (read-only), role, **MFA status** (factor enrolled, assurance level) with **Sign out of all devices**; note "To reset your authenticator, ask another admin." No password-change shortcuts beyond the standard reset flow.
- **Help `/help`:** short operational guide: how reviews and appeals work, the different-reviewer rule, what the kill switch does, what each config key affects, how to read the audit log, security reminders (never share your authenticator, lock your screen), contact support (mailto).
- **System screens:** wrong-role (5), 404 ("That page doesn't exist." + **Go home**), `ErrorBoundary` (**Reload page**; never a stack trace or payload), offline banner, and the **PrototypeNotice** (slim, dismissible per session) when `VITE_APP_ENV !== 'production'`: "Prototype. All people, universities and organizations shown are fictional."

---

## 9. Copy and voice

Plain, calm, precise. Sentence case. Active verbs. **No hype, no jargon in visible copy** (avoid "RLS", "aal2", "JWT": say "two-step verification"). State the consequence of an action **before** the action. Errors say what happened and the fix and **don't apologize**. Vocabulary (use consistently): **Application · Approve · Reject · Review · Confirm · Adjust · Appeal · Ticket · Decision · Run · Retry · Setting.** A button's verb is reused in its toast.

| Moment | Copy |
|---|---|
| Blind review | Reviews are blind to the student's identity to reduce bias. |
| Approve university | Approving unlocks registration for 6 rostered students and creates matches with existing profiles. |
| Approve organization | Approving gives the Verified organization badge. Turn on the subscription separately. |
| Reject note | This note may be shown to the applicant. |
| Review confirmed / adjusted | Review confirmed. · Score adjusted. |
| Consequence (verified) | New score 58 → Competent · Verified. It will become visible to organizations. |
| Different reviewer | You confirmed the original review, so a different reviewer must decide this appeal. |
| Kill switch | Pause all AI reviews and project assignments. Students keep their drafts. |
| Kill switch banner | AI is paused. New projects and reviews are blocked. |
| Audit log | This log can't be edited or deleted. |
| Impersonation | AptoHire doesn't allow impersonation. |
| Already decided | This review was already decided. Refreshing the queue. |
| MFA required | Two-step verification is required for admin accounts. |
| Idle | You'll be signed out in 2 minutes. |
| Rate limited | Too many attempts. Try again in a few minutes. |
| Network error | We couldn't reach AptoHire. Check your connection and try again. |
| Session expired | Your session ended. Sign in again to continue. |
| Login failure | Email or password is incorrect. |

Error-code mapping: `VALIDATION_FAILED` map `fields` · `MFA_REQUIRED` re-route to the challenge · `FORBIDDEN` "You don't have access to that." (or the different-reviewer copy in ticket context; or "Reset is turned off in this environment." for seed-reset) · `CONFLICT` "This was already decided. Refreshing." · `NOT_FOUND` "That item no longer exists." · `RATE_LIMITED` rate copy · unknown → network copy. All strings live in `src/content/*.ts` or per-feature `copy.ts`.

---

## 10. Quality bars

### 10.1 Responsiveness (built in, not deferred)

Breakpoints: **Mobile < 768 (base) · Tablet 768–1023 · Desktop ≥ 1024**. Tailwind `md` 768, `lg` 1024, `xl` 1280.

- `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">`; safe-area insets on fixed bars and the sticky `DecisionBar`.
- **No horizontal scroll from 360 px up.** Fix root causes; never `overflow-x:hidden` on `body` as a patch. Tables scroll inside their own container at ≥ 640 px and become labelled lists below.
- Multi-column layouts collapse below 768; panes stack; mobile section padding ≥ 40 px vertical; nothing touches the screen edge.
- Body ≥ 16 px, small ≥ 14 px, line-height ≥ 1.4; headings use the fluid tokens.
- **Touch targets ≥ 44×44 px** on touch widths (buttons, icon buttons, tab-bar items, row actions, menu items, transcript lines).
- **Forms:** inputs 100% width on mobile, **font-size ≥ 16 px** (the 6-digit MFA field especially), labels above inputs, form buttons full-width on mobile, correct `type`/`inputMode`/`autocomplete`.
- The review workbench, decision bar, ticket dialogs and config forms must be fully usable on a phone/tablet.
- Test at **360, 375, 390, 768, 1024, 1440**.

### 10.2 Accessibility (WCAG 2.2 AA)

Landmarks, one `h1` per page, skip link, **visible 2 px `verdant-700` focus outline with 2 px offset plus a 4 px white halo**, all flows keyboard-operable, focus trapped/restored in dialogs/sheets/drawers/menus, `aria-live="polite"` for queue updates, auto-refresh, toasts and filter counts, errors announced and focus moved to the first invalid field, **tables have headers, `scope`, `aria-label`, `aria-sort`**, the transcript is real text with the current line announced politely only on user action, video has controls and transcript, colour never the only signal (status chips, flags, SLA), reduced motion respected, the MFA field supports password managers/OTP autofill. `@axe-core/playwright` zero serious/critical on every route.

### 10.3 Performance

Mobile Lighthouse ≥ 90 (Performance), ≥ 95 (Accessibility, Best Practices). Route-level code splitting; initial JS for login and Approvals ≤ **180 KB gzipped** (no chart library). Server-side pagination (50 per page); debounce search (300 ms); skeletons match layouts; do not load the video player or transcript until a review/evidence is opened.

---

## 11. Security and privacy rules (never break)

1. **No secrets in repo or bundle.** Only `VITE_*`. Never the service-role/secret key. `.env*` git-ignored except `.env.example`. A static script fails the build on `service_role`, `sb_secret`, or hard-coded hex colours in components.
2. **MFA is mandatory.** No admin data hook runs before `AuthGate → RoleGuard → MfaGate` pass (tested). The mock code `123456` exists only in the mock implementation and is excluded from production builds.
3. **No impersonation, anywhere.** No "view as", no session switching, no token display or copying. A static test fails the build if the strings `impersonate`, `view as user`, or `login as` appear in `src/` outside the explanatory banner.
4. **Identity-blind review.** Domain types for reviews and tickets **do not contain** student name, email, university, photo or `student_id`; mappers drop them even if the backend returns them. A DOM test renders the review and ticket screens with the mock store containing identities and asserts none appear (including titles, aria-labels and tooltips).
5. **Never trust the client for authorization.** The UI hides things for convenience; **RLS and Edge Functions decide**. Never solve a permission error by moving a check into the frontend or widening a query. The different-reviewer rule is enforced by the backend; the UI mirrors it.
6. **Pessimistic mutations.** Approvals, reviews, tickets, config and retries wait for the server; no optimistic UI; every success shows who/when where available.
7. **Dangerous actions** (kill switch, `AGENT_MODE`, `PASS_MARK`, `BORDERLINE`, seed reset) need **step-up** (and typed confirmation for kill switch and reset). Step-up state is memory-only and expires in 5 minutes.
8. **Idle timeout** signs the admin out (default 30 minutes) and clears caches, channels and signed URLs.
9. **Untrusted content is data.** Student titles, descriptions, transcripts, READMEs, excerpts, flags, error messages, audit `details` and application payloads render as **text** only (React escaping; `<pre>` for code). **No `dangerouslySetInnerHTML`.** No HTML/Markdown rendering of student content.
10. **Student-provided links** open only after the **`ExternalLinkDialog`**, with `rel="noopener noreferrer"`. Signed URLs (video, documents) are short-lived, created on demand, never stored or logged.
11. **No PII in logs, analytics, error messages, URLs or storage.** Allowed storage: the Supabase session (`aptohire-admin`), notice dismissals, mock state (mock mode only). No third-party scripts, no analytics.
12. **CSV exports** use the formula-injection guard and a confirmation dialog; audit log and reports export only the visible, filtered columns.
13. **Demo-only code** (demo panel, mocks, reset tab, prototype notice) is excluded or disabled in production; the build fails if mocks/demo are on in production.
14. **The admin URL is never published or linked** by this project; README states that security comes from authentication and MFA, not obscurity.
15. Pin exact dependency versions; run `npm audit` and report results.

---

## 12. Deployment (its own Vercel project; strictest headers)

`vercel.json`: rewrite everything not a file to `/index.html`. Headers on all routes: `Content-Security-Policy` (`default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob: https://*.supabase.co; media-src 'self' blob: https://*.supabase.co; font-src 'self'; connect-src 'self' https://*.supabase.co wss://*.supabase.co; frame-ancestors 'none'; base-uri 'self'; form-action 'self'; object-src 'none'; worker-src 'self' blob:`), `X-Content-Type-Options: nosniff`, `Referrer-Policy: no-referrer`, `Permissions-Policy: camera=(), microphone=(), geolocation=()`, `X-Frame-Options: DENY`, `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`, `Cross-Origin-Opener-Policy: same-origin`, `Cross-Origin-Resource-Policy: same-origin`, `X-Robots-Tag: noindex, nofollow`, **`Cache-Control: no-store`** on HTML routes (hashed assets stay `immutable`).

README must explain: deploy as **its own Vercel project** with domain `admin.aptohire.com`; set the `VITE_*` variables; in Supabase **Authentication → URL Configuration** add `https://admin.aptohire.com/**` (and `http://localhost:5177/**`); **create the first admin manually** with the backend's `scripts/create-admin.sql` (there is no sign-up), then enrol TOTP at first sign-in; the other four portals are separate projects; `// TODO(decision)` mention optional extra protection (Vercel Deployment Protection or an IP allow-list in front of this project) as a human step.

---

## 13. Testing

| Level | Cover |
|---|---|
| Unit (Vitest) | `levels.ts` (levels at 0, 40, 50, 51, 60, 61, 69, 70, 84, 85, 100; verified only **above** pass mark; borderline from config), `consequence.ts` (verified/not verified/held, uses config), `sla.ts` (3 working days across weekends, due-soon, overdue), `configRules.ts` (low < high, ranges, MB/seconds bounds, cap bounds), `agreement.ts` (agree on confirm or ≤ 5 points, false-pass, small-sample flag), reviewer-exclusion helper, `mfa.ts` state machine, `idle.ts` (fake timers: warning at 2 minutes, sign-out), `csvExport.ts` injection guard, typed-confirm logic, Zod schemas (valid and invalid), mappers **drop identity fields**, `links.ts`, `env.ts` |
| Component (RTL) | **Gate order** (non-admin never sees MFA enrol; admin without factor → enrol; with factor → challenge; **no data hooks fire before `aal2`**), wrong-role screens, Approvals (confirm dialogs show consequences; reject requires a note ≥ 10 chars; subscription toggle confirm; checks panel rows; pending-only actions), Review workbench (**identity-free DOM**, confirm and adjust validation, consequence preview, conflict handling, auto-advance, evidence chips open the right panel, links go through `ExternalLinkDialog`), Tickets (original reviewer excluded from assignment, Decide disabled for the original reviewer, reason required, `criterion` sent only when supported), Agent runs (Retry only for failed/timeout, disabled while kill switch on, no identities shown), Settings (step-up required for danger keys, typed confirm for kill switch/reset, banner appears and persists, old → new displayed), Audit log (no mutation controls; export guard + confirmation), Users (no management buttons, no impersonation text beyond the banner), keyboard shortcuts never perform decisions |
| E2E (Playwright, mock mode) | Admin signs in (password, **MFA enrol with `123456`**, then challenge on the next sign-in) → Approvals: approve **Uni C** (consequence shown, 6 rostered students) → toast; reject **Riverbend College** with a note; approve **Orbit Analytics** and toggle subscription → Reviews: open **Naveen's React 52** → watch the video seek via an evidence chip → **Confirm** (or adjust to 58 and see the consequence) → auto-advance → Tickets: acknowledge, assign to Reviewer 2 (Admin 1 is excluded on the "confirmed by you" ticket), decide as Reviewer 2 via the mock account switch → Agent runs: filter failed → retry → Settings: **kill switch** with typed confirm + step-up → banner everywhere → turn off → change `PASS_MARK` and read the levels preview → Audit log: filter and export → Demo data reset. Axe on every route; **no horizontal overflow at 360, 390, 768, 1024, 1440**; keyboard-only runs of the review decision, the ticket decision and the kill switch; reduced-motion run; idle-timeout dialog (fake clock) |
| Static | `typecheck`, `lint`, secret/hex grep over `src/` and `dist/`, **impersonation-string grep** |

---

## 14. README

Beginner-friendly (short sentences, numbered steps): what this project is; how it relates to the other four portals and the shared database; **why this console is locked down** (MFA, blind reviews, no impersonation); prerequisites (Node LTS, Git); install; copy `.env.example`; **run in mock mode** (use code `123456` for MFA in mock mode only); connect to a real Supabase project (URL + anon key, set `VITE_USE_MOCKS=false`, copy `docs/CONTRACT.md` from `aptohire-backend`, run `npm run db:types`); **creating the first admin** (backend script) and enrolling an authenticator app; commands table; folder tour; deployment to Vercel and optional extra protection; how to read the queues, the different-reviewer rule and the kill switch; common problems (white page → env typo; "two-step verification required" → finish enrolment; "permission denied" → role or MFA; lost authenticator → another admin resets it; Supabase project paused → Restore); glossary (kill switch, borderline, appeal, step-up, audit log, mock mode). State clearly: **this app never needs the service-role key, has no sign-up, and has no impersonation.**

---

## 15. Build order (finish each phase, run the checks, then continue)

1. **Foundations:** Vite/TS/Tailwind, tokens, fonts, `env.ts`, `links.ts`, router, providers, `Backend` interface, **mock backend skeleton**, Supabase client module, lint/test/e2e setup.
2. **Design primitives and shell:** Button, Field family, Dialog/Sheet/Toast/Tabs/SegmentedControl/Skeleton/EmptyState, chips, Logo/SealMark, `AppShell` (rail, top bar, drawer, bottom tab bar, More sheet), `CountBadge`, `KillSwitchBanner`, PrototypeNotice.
3. **Auth, MFA and gates:** login, password reset, callback, `AuthGate` → `RoleGuard` → `MfaGate` (enrol, challenge), `IdleGuard`, `StepUpDialog`, `TypedConfirmDialog`, `ReasonDialog`, wrong-role screens, account, tests (gate order and MFA state machine first).
4. **Data components:** dense `DataTable` with list mode, `FilterBar` (URL-synced), `StatRow`, `SlaChip`, `JsonViewer`, split-pane layout, counts polling, `lib/sla.ts`, `lib/consequence.ts`, `lib/configRules.ts` with unit tests.
5. **Approvals:** lists, `ApplicationSheet`, `ChecksPanel`, approve/reject/reconsider, subscription toggle, tests.
6. **Reviews:** queue, **`ReviewWorkbench`** (AI assessment, evidence panel with video/transcript/code, link interstitial), `DecisionBar`, adjust flow, auto-advance, **identity-blind DOM test**.
7. **Tickets:** list, detail, timeline, assign (different-reviewer rule), decide dialog, tests.
8. **Agent runs and Settings:** stats, table, run sheet, retry; configuration forms, kill switch panel and banner, step-up flows, demo data tab.
9. **Audit log, Users, Reports, notifications, help, system screens.**
10. **Library** (skills and templates; P1) and remaining polish.
11. **Hardening:** real `supabase/` implementation of every `Backend` method (adapting mappers to `docs/CONTRACT.md`), axe, Lighthouse, overflow checks, keyboard and reduced-motion passes, privacy and impersonation greps, secret/hex greps, `npm audit`, README.

---

## 16. Acceptance checklist

- [ ] `typecheck`, `lint`, `test`, `build`, `e2e` pass; no `any`; no leftover `console.log`
- [ ] Entire app works with `VITE_USE_MOCKS=true` and no network; switching to the real backend needs only env changes plus the items in 4.7
- [ ] Gates run in order `AuthGate → RoleGuard → MfaGate`; non-admins are refused (never asked to enrol MFA); no admin data is fetched before `aal2`; no sign-up route exists
- [ ] MFA enrolment (QR + secret + code) and challenge work; idle timeout warns and signs out; step-up works and expires in 5 minutes
- [ ] Approvals: universities and organizations lists, checks panel, approve/reject with consequences and required rejection note, subscription toggle with confirm; no bulk decisions
- [ ] Reviews: queue with SLA chips (3 working days, overdue first), side-by-side workbench, video seek + transcript, code/doc excerpts, link interstitial, confirm/adjust with consequence preview, auto-advance, conflict handling
- [ ] **Review and ticket screens contain no student identity** (DOM test with identities present in the store)
- [ ] Tickets: acknowledge → assign → decide; the original reviewer is excluded from assignment and from deciding; reason required; recorded and student notified (by the backend)
- [ ] Agent runs: stats, filters, auto-refresh toggle, run sheet, retry only for failed/timeout, disabled under kill switch; no identities shown
- [ ] Settings: all config keys validated; old → new shown; step-up on danger keys; typed confirm for kill switch and reset; persistent kill-switch banner; demo reset hidden in production
- [ ] Audit log is read-only with filters and guarded CSV export; Users is read-only with the impersonation banner; **no impersonation anywhere** (string grep passes)
- [ ] Pass mark and borderline range read from config, never hard-coded in UI logic
- [ ] No gradient surfaces; sentence case; no tracked all-caps labels; contrast rules respected (white on `verdant-700`; white on `danger-600` only in confirm dialogs); no orange buttons
- [ ] Bottom tab bar < 768, drawer 768–1023, icon rail 1024–1279, full rail ≥ 1280; touch targets ≥ 44 px on touch widths; inputs ≥ 16 px; no horizontal scroll at 360–1440; tables become labelled lists below 640 px
- [ ] Axe: zero serious/critical on every route; Lighthouse targets met; initial JS ≤ 180 KB gzipped
- [ ] No secrets or PII in repo/bundle/logs/URLs/storage; student content only rendered as text; demo code excluded from production builds; production build fails if mocks/demo are on
- [ ] README and `docs/PLAN.md` complete; every `TODO(decision)` and `TODO(backend)` listed in the report

---

## 17. Final report (output exactly this at the end)

### ✅ Admin console build report

**Stack used:** [versions]

**Routes built:** [list]

**Backend interface:** [methods implemented in `mock/` and in `supabase/`; any method still mock-only]

**Backend items required (4.7) and their runtime fallbacks:** [list; mark which are blocking]

**Decisions made (`TODO(decision)` / `TODO(backend)`):** [each item and the default chosen]

**Quality results:**
- Lighthouse (mobile): Performance [x] · Accessibility [x] · Best Practices [x]
- Initial JS (gzipped): login [x KB] · Approvals [x KB]
- Axe violations (serious/critical): [0 per route]
- Overflow check at 360 / 375 / 390 / 768 / 1024 / 1440: [pass/fail]
- Security tests: gate order [pass/fail] · identity-blind DOM [pass/fail] · different-reviewer rule [pass/fail] · impersonation grep [pass/fail] · step-up and idle [pass/fail]

**Navigation:** [bottom tab bar, drawer and rail behaviours verified]

**Files created:** [grouped list]

**What to check manually:**
- Chrome DevTools → Device Toolbar (Ctrl+Shift+M) at 375, 390, 768, 1024 and 1440 px
- Sign in as the demo admin, enrol MFA with `123456` (mock mode), sign out and sign in again to see the challenge
- Open a borderline review and search the page (DevTools → Elements) for any student name, email or university; confirm none appears
- Try to decide the ticket whose original review you confirmed; confirm the console blocks it with the different-reviewer message
- Turn the kill switch on (typed confirm + step-up) and confirm the banner appears on every page; turn it off
- Leave the console idle past the warning and confirm the sign-out dialog and sign-out
- Turn on "reduce motion" in your OS: the Proof Ring and running-status pulse should show final states instantly
