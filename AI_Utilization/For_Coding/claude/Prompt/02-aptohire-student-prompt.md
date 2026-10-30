# PROMPT 2 of 5 — `student.aptohire.com` (Student portal)

> Paste everything below this line into your AI coding tool inside an **empty folder** named `aptohire-student/`.
> This project is **independent**. It does not import from, or share code with, the other portals. Where this prompt says "copy", copy the file; never create a cross-project import.

---

## 0. Your role and how to work

You are a senior frontend engineer and product designer. Build the **Student portal of AptoHire** (`student.aptohire.com`) as a complete, production-quality Vite + React + TypeScript app that talks to a **shared Supabase backend**.

Work in this order:

1. **Plan (short).** Write `docs/PLAN.md` (max 1 page): route list, feature modules, the `Backend` interface (section 4.1), what you will mock. Then build.
2. **Build** in the phases of section 15. After each phase run `npm run typecheck && npm run lint && npm run test` and fix failures before moving on.
3. **Verify** against section 16 at 360, 390, 768, 1024 and 1440 px.
4. **Report** in the exact format of section 17.

Rules for decisions:

- Do not ask me questions. If something is unclear, pick the sensible default, leave `// TODO(decision): …`, and list it in the report.
- Never invent product behaviour. Everything here comes from this prompt. No fake statistics, testimonials or logos.
- **All people, universities, organizations and data are fictional.** Use only the fictional seed data in section 4.7.
- The backend (`aptohire-backend/`) may not exist yet. **The app must run completely in mock mode** (`VITE_USE_MOCKS=true`) and switch to the real backend by changing one env value.

---

## 1. Product context (student view)

**AptoHire — "Hired for what you can do."** Students prove their skills by building real projects. A review scores each skill against a rubric, links every score to its **evidence**, and issues a **verified scorecard**. Organizations later find students **without seeing names**; identity is revealed only after the student accepts a request.

The student journey this portal must deliver:

1. **Register & verify** against a university (university email → roster match; personal email + student ID → hashed ID match; or no registered university → *Unverified*, with scores and job access **unaffected**).
2. **Pick a target** skill or role.
3. **Get a project** at the right level (project assistant).
4. **Build & submit**: title, description, **mandatory 3-minute demo video**, documents, live link, GitHub link, **AI-use disclosure**.
5. **Review**: live progress, then the **scorecard** (Proof Ring, level, evidence trail). Skills scoring **above the pass mark (50, read from config, never hard-coded)** become **Verified**.
6. **Appeal** if unhappy: a ticket, decided by a *different* reviewer.
7. **Sharing**: organizations send access requests; the student accepts (choosing scope and duration) or declines, and can **withdraw access any time**.
8. **Profile that lasts**: one permanent profile across university changes, graduation, dropout, masters; a downloadable **digital resume** with a QR code to the public verification page.

Levels: **Emerging** (0–50, not verified) · **Competent** (51–69) · **Proficient** (70–84) · **Advanced** (85–100). Scores **40–60 are borderline**: shown as "Under human review" and **not shared with organizations** until an admin confirms.

---

## 2. Where this project fits

AptoHire is **five separate web projects sharing one Supabase database**:

| Project | Host | Port (local) |
|---|---|---|
| `aptohire-root/` | `aptohire.com` | 5173 |
| **`aptohire-student/` ← YOU ARE BUILDING THIS** | `student.aptohire.com` | **5174** |
| `aptohire-university/` | `university.aptohire.com` | 5175 |
| `aptohire-organization/` | `organization.aptohire.com` | 5176 |
| `aptohire-admin/` | `admin.aptohire.com` | 5177 |

A sixth shared folder, `aptohire-backend/`, owns migrations, RLS policies, seed data and Edge Functions. **This project never edits any of that.** It only *consumes* the database through the browser-safe anon key, the signed-in student's JWT, Row-Level Security (RLS), Storage, Realtime and the Edge Functions in section 4.

If `../aptohire-root` exists next to this folder, you may **read** it and **copy** these files (do not import across projects): `src/styles/tokens.css`, `src/lib/scoring.ts`, `src/components/proof/ProofRing.tsx`, `StatusChip.tsx`, `src/components/brand/*`. If it doesn't exist, build them from sections 6–7.

**Hard boundaries:**

- This site serves **students only**. A signed-in user whose role is not `student` sees the "Go to your portal" screen and **no student data is fetched** (section 5).
- **Never link to or mention the admin portal.**
- **The service-role key must never exist in this repo.** Only `VITE_*` values.
- The student can never see another student's data. Do not build anything that tries.

---

## 3. Tech stack and project setup

| Layer | Choice |
|---|---|
| Build | **Vite + React 18 + TypeScript (strict, no `any`; use `unknown` + Zod)** |
| Routing | React Router (data router), every route `React.lazy` except the shell and login |
| Data | **TanStack Query** + `@supabase/supabase-js` (used **only** inside `src/lib/backend/supabase/`) |
| Forms | React Hook Form + **Zod** (`@hookform/resolvers`) |
| Styling | **Tailwind CSS 3.4 (pinned)** + CSS variables (section 6) + **shadcn/ui** (Radix), install only primitives you use |
| Icons | `lucide-react`, stroke 1.75, 20 px |
| Fonts | Self-host via Fontsource: `@fontsource-variable/bricolage-grotesque`, `@fontsource-variable/instrument-sans`, `@fontsource/jetbrains-mono` (400) |
| Charts / graph | `recharts` (score history), `d3-force` (skill graph), both **lazy-loaded** |
| Uploads | `tus-js-client` for resumable video upload to Supabase Storage (lazy-loaded on the submission page) |
| Resume | `@react-pdf/renderer` + `qrcode` (**dynamic import on click only**) |
| Motion | No animation library. CSS + SVG + a tiny `useCountUp` hook |
| Head | `react-helmet-async` (`noindex` on every route: this is an app, not a landing site) |
| Tests | Vitest + React Testing Library, Playwright + `@axe-core/playwright` |
| Lint | ESLint (typescript-eslint, react-hooks, jsx-a11y) + Prettier |

### Scripts

`dev` (port **5174**, `--strictPort`), `build`, `preview`, `typecheck`, `lint`, `test`, `e2e`, `db:types` (`supabase gen types typescript --project-id $SUPABASE_PROJECT_REF > src/lib/database.types.ts`; the generated file is committed; until it exists use hand-written domain types).

### Environment (`.env.example` committed; `.env.local` ignored)

```
VITE_APP_ENV=local                  # local | demo | production
VITE_DEMO_MODE=true                 # enables demo account switcher + prototype notice
VITE_USE_MOCKS=true                 # true = in-memory backend, no network
VITE_SITE_URL=http://localhost:5174
VITE_ROOT_URL=http://localhost:5173
VITE_ROOT_DOMAIN=localhost
VITE_UNIVERSITY_URL=http://localhost:5175
VITE_ORGANIZATION_URL=http://localhost:5176
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=             # public anon/publishable key ONLY
VITE_DEMO_PASSWORD=                 # demo/local only; fictional accounts; build fails in production if set
VITE_VIDEO_MAX_MB=50                # FALLBACKS only; real values come from public config (4.4)
VITE_VIDEO_MAX_SECONDS=190
VITE_SUPPORT_EMAIL=support@aptohire.test
```

`src/config/env.ts` parses these with Zod at startup and throws a readable error. **Build fails** if `VITE_APP_ENV=production` and any of `VITE_USE_MOCKS`, `VITE_DEMO_MODE` is true or `VITE_DEMO_PASSWORD` is set.

`src/config/links.ts` exports `rootUrl(path)` and `portalUrl('university' | 'organization', path)` (explicit env first, otherwise `https://{portal}.${VITE_ROOT_DOMAIN}`). **No other code builds cross-site URLs. There is no admin option.**

### Folder structure

```
aptohire-student/
├─ README.md  .env.example  index.html  package.json  vite.config.ts  tailwind.config.ts
├─ vercel.json            # rewrites + security headers (section 12)
├─ public/                # favicon.svg, robots.txt (Disallow: /)
├─ docs/PLAN.md
├─ src/
│  ├─ main.tsx
│  ├─ app/                # router.tsx, providers.tsx, RoleGuard.tsx, AuthGate.tsx, ErrorBoundary.tsx
│  ├─ config/             # env.ts, links.ts
│  ├─ components/
│  │  ├─ shell/           # AppShell, SideRail, TopBar, BottomTabBar, MoreSheet, AccountMenu, NotificationBell, SkipLink, PrototypeNotice
│  │  ├─ proof/           # ProofRing, EvidenceRow, EvidenceChip, EvidenceSheet, StatusChip, SkillGraph, ScoreHistory
│  │  ├─ brand/           # Logo, SealMark
│  │  └─ ui/              # Button, Field, Input, Select, Textarea, Checkbox, Switch, Tabs, Table, Dialog, Sheet, Toast, Skeleton, EmptyState, ...
│  ├─ features/           # one folder per area; each: api hooks (use*.ts), components/, schemas
│  │  ├─ auth/ onboarding/ profile/ affiliations/ projects/ submissions/ audit/ scorecard/
│  │  ├─ skillgraph/ tickets/ sources/ resume/ sharing/ notifications/ config/ account/
│  ├─ pages/              # route components (thin; compose features)
│  ├─ lib/
│  │  ├─ backend/         # types.ts (Backend interface), index.ts (factory), supabase/, mock/
│  │  ├─ supabase/client.ts   # the ONLY createClient call
│  │  ├─ domain/          # types.ts, mappers.ts (DB row → domain)
│  │  ├─ schemas/         # zod schemas for every API response and form
│  │  ├─ scoring.ts  format.ts  files.ts  media.ts  cn.ts
│  └─ styles/             # tokens.css, globals.css
├─ tests/                 # vitest
└─ e2e/                   # playwright
```

`scoring.ts`: `levelFor(score)`, `isVerified(score, passMark)`, `isBorderline(score, [low, high])`. **`passMark` and the borderline range always come from public config (4.4); the default constants exist only as fallbacks** with a comment: *"Mirror of app_config. Keep in sync."*

---

## 4. Backend contract (what this app consumes)

### 4.1 One interface, two implementations

Define `Backend` in `src/lib/backend/types.ts` grouped by feature (`auth`, `student`, `affiliations`, `skills`, `projects`, `submissions`, `uploads`, `audits`, `scores`, `tickets`, `sources`, `sharing`, `notifications`, `config`, `account`) with **Zod-validated** inputs and outputs. Two implementations: `supabase/` (real) and `mock/` (in-memory, section 4.6). The factory picks by `VITE_USE_MOCKS`. Provide it through a `BackendProvider` context. **Components never import supabase or call fetch**; they use feature hooks (`useSubmissions`, `useScorecard`, …) built on TanStack Query. Query keys are arrays (`['submissions', studentId]`); invalidate on mutation; Realtime updates patch the cache.

### 4.2 Direct table access (RLS enforces; the app only ever asks for the signed-in student's own rows)

| Table / view | Student can | Used for |
|---|---|---|
| `profiles` | read own (role) | role guard |
| `students` | read/update own limited columns | profile, visibility, credential id |
| `affiliations` | read own | education timeline, lifecycle |
| `my_affiliation_proposals` (view, **backend must add**) | read own | "Confirm new university" |
| `universities` | read **public columns** of approved/registered (backend exposes them as the view `public_universities`) | university picker |
| `organizations`, `jobs` | read **public columns** (views `public_organizations`, `public_jobs`) | on access request cards |
| `skills`, `skill_edges` | read | taxonomy, skill graph |
| `project_assignments` | read own; update own `status` | projects list |
| `submissions` | CRUD own while `draft`; read own after | submission form, status |
| `agent_runs` | read own (by submission) | live progress (`stage`, `status`) |
| `audits` | read own (via submission) | scorecard, evidence |
| `skill_scores`, `skill_score_history` | read own | scorecard, graph, history |
| `tickets` | insert + read own | appeals |
| `access_requests` | read own, joined with org/job public columns (backend view `student_access_requests_v`) | Sharing |
| `feedback` | read own | feedback received |
| `notifications` | read own; update `read_at` | notification centre |
| `profile_sources` | read/update own consent flags | connected sources |
| `public_config` (view, **backend must add**) | read | pass mark, borderline range, `REQUIRE_VIDEO`, `VIDEO_MAX_MB`, `VIDEO_MAX_SECONDS`, `AI_KILL_SWITCH`, `AI_DAILY_CAP_PER_USER` |

> **Exact object names come from `docs/CONTRACT.md`** (the user copies it here from `aptohire-backend`). If it differs from the names in this prompt, **the contract wins**.

The student **never** reads `roster_entries`, `candidate_profiles`, `match_results`, or any other student's rows. Never request columns you don't render.

### 4.3 Storage (owner-scoped, private; use signed URLs)

| Bucket | Path | Rules (also enforced client-side) |
|---|---|---|
| `videos` | `{student_id}/{submission_id}/demo.{ext}` | mp4/webm/mov, ≤ config MB, ≤ config seconds (+10 s grace in the message only); **resumable (TUS)** |
| `documents` | `{student_id}/{submission_id}/{name}` | pdf/docx/md/zip, ≤ 20 MB each, max 5 files |
| `avatars` | `{student_id}.jpg` | ≤ 1 MB; client-side crop to 512 px square and re-encode to JPEG (this strips EXIF/location) |
| `resumes` | `{student_id}/resume.pdf` | optional cache; the PDF is generated client-side |

**No ID-card images are ever uploaded anywhere.** Signed URLs for playback: 10 minutes, created on demand, never persisted.

### 4.4 Edge Functions (called with the student's JWT; base `${VITE_SUPABASE_URL}/functions/v1`)

All errors have the shape `{ error: { code, message, fields? } }`. Map every code to calm copy (section 9); never show raw messages.

| Function | Request → Response |
|---|---|
| `complete-signup` | `{ portal: 'student' }` → `204`. Sets `profiles.role` once. Idempotent |
| `register-student` | `{ full_name, program_level: 'ug'\|'pg', year?: number, choice: {kind:'detect'} \| {kind:'student_id', university_id, student_id_number} \| {kind:'unregistered', university_name} }` → `{ student_id, outcome: 'verified_roster'\|'needs_student_id'\|'verified_student_id'\|'unverified'\|'confirm_existing'\|'roster_no_match', affiliation?, matched_existing_person: boolean }`. The **raw student ID number is sent only to this function over TLS**; never log it, store it in state after submit, or put it in a URL |
| `confirm-affiliation` | `{ proposal_id }` **or** `{ new: { university_id?, university_name_text?, student_id_number?, program_level, year?, status } }` → `{ affiliation }`. Closes the old affiliation, opens the new one; scores untouched |
| `assign-project` | `{ target: {skill_slug} \| {role_text}, level?: 'Competent'\|'Proficient'\|'Advanced' }` → `201 { assignment_id, brief: ProjectBrief }`. Errors: `NO_TEMPLATE`, `QUOTA_EXCEEDED`, `KILL_SWITCH`, `VALIDATION_FAILED` |
| `submit-project` | `{ submission_id, confirm: true }` → `202 { run_id, status, mode }`. Errors: `VIDEO_REQUIRED`, `VIDEO_TOO_LONG`, `MISSING_SKILLS`, `QUOTA_EXCEEDED`, `KILL_SWITCH`. Also used to **retry** a `failed` submission |
| `respond-access` | `{ request_id, decision: 'accept'\|'decline', granted_scope?: Scope[], duration_days?: 7\|30\|90 }` → `{ status, expires_at? }` |
| `withdraw-access` | `{ request_id }` → `{ status: 'withdrawn' }` |
| `profile-sync` | `{ action: 'github_start' }` → `{ authorize_url }`; `{ action: 'github_sync' \| 'linkedin_parse' \| 'revoke', source_id?, text?, file_path?, delete_data?: boolean }` → `{ proposals?: ProfileProposal[], last_synced_at }` |
| `export-my-data` (**backend to add**) | `{}` → `{ signed_url }` (JSON export) |
| `delete-my-account` (**backend to add**) | `{ password, confirm: 'DELETE' }` → `204` |

Types (`src/lib/schemas/`): `Scope = 'profile'\|'scorecard'\|'projects'\|'contact'`; `ProjectBrief`, `AuditOutput` summary fields and `ProfileProposal` exactly as:

```ts
type ProjectBrief = {
  title: string; scenario: string; level: 'Competent'|'Proficient'|'Advanced';
  target_skills: string[]; deliverables: string[]; constraints: string[];
  suggested_stack?: string[]; estimated_hours: number;
  rubric_preview: { criterion: string; weight: number }[];
  video_guidance: string; originality_questions: string[];
};
type EvidenceItem = { ref: string; kind: 'video'|'file'|'repo'|'doc'|'live'; label: string; pointer: string; excerpt?: string };
type AuditCriterion = { name: string; weight: number; score: number | null; rationale: string; evidence_refs: string[]; insufficient_evidence?: boolean };
type Audit = { id: string; submission_id: string; created_at: string; overall: number; confidence: number; summary: string;
  next_steps: string[]; flags: string[]; borderline: boolean; review_status: 'none'|'pending'|'confirmed'|'adjusted';
  per_skill: { skill_id: string; score: number; criteria: AuditCriterion[] }[];
  evidence: EvidenceItem[]; transcript?: { t: number; text: string }[] };
type ProfileProposal = { source: 'github'|'linkedin_export';
  changes: { field: 'headline'|'skills_interest'|'experience'|'education'|'links'|'project_candidate';
             action: 'add'|'update'; value: unknown; evidence: string }[] };
```

Video evidence `pointer` = seconds as a string; file/repo pointer = `path#Lstart-Lend`; render chip labels as `Video 01:42`, `src/cart.tsx`, `README.md`, `Live site`. Confidence label: ≥ 0.75 **High**, ≥ 0.5 **Medium**, else **Low** (`// TODO(decision)` thresholds).

### 4.5 Realtime

Subscribe (Postgres changes, RLS-filtered) to: `submissions` (own), `agent_runs` (own, for the open submission), `notifications` (own), `access_requests` (own). Always unsubscribe on unmount. **Fallback:** if the channel isn't joined within 5 s or drops, poll every 5 s while a review is in progress. Never mark a review "Ready" from the UI's own timer; only from `submissions.status`.

### 4.6 Mock backend (required, in-memory, deterministic)

- Seeded with the fictional data in 4.7. State lives in memory and `sessionStorage` (try/catch) so refresh keeps the demo state; a **Reset demo data** link (demo mode only, in Account) clears it.
- Latency 300–800 ms. Realtime via an in-process event emitter.
- **Mock review:** `submit-project` moves the submission to `auditing` and advances `agent_runs.stage` through `received → reading → watching → scoring → writing → done` every 2–3 s (≈ 12 s total), then writes a deterministic `Audit` generated from a hash of title + description for the claimed skills, using criteria and weights of rubric `web-frontend@1` (Functionality 25 · Code quality and structure 20 · UI, UX and accessibility 15 · Testing and reliability 10 · Documentation 10 · Problem solving and depth 10 · Communication (video) 10), rationales of ≤ 280 characters, evidence refs, a summary ≤ 600 characters and ≤ 3 next steps. Overall = weighted sum recomputed locally. Scores 40–60 set `borderline` and `review_status: 'pending'`. Title containing `[fail]` produces a `failed` run (to test Retry); title containing `[slow]` takes 40 s.
- Mock `assign-project`: Web/Frontend skills return a full `ProjectBrief` (vary scenario by hash); other families return `NO_TEMPLATE`; the 6th request of a day returns `QUOTA_EXCEEDED`.
- Mock `register-student`: email domain `unia.edu.test` → `verified_roster` (Uni A); university "Uni B" + ID `B-1042` → `verified_student_id`; any other ID → `needs_student_id` then `unverified` option; unknown university name → `unverified`; `avi@demo.aptohire.test` → `confirm_existing`.
- Mock uploads: simulate progress. Mock signed URLs return a local placeholder (the video panel then shows the transcript at the selected time instead of a player).

### 4.7 Fictional seed data (mock mode)

Demo password for all mock accounts is `VITE_DEMO_PASSWORD` (any non-empty value in mock mode).

| Account | Situation |
|---|---|
| `aarav@unia.edu.test` | Uni A, year 2, **Verified university (roster)**. **React 72 Proficient, verified**, TypeScript 58 Competent verified; one scored submission with full evidence; consent states: **one pending** request (Northwind Labs, profile + scorecard, 30 days), **one accepted** (Brightlane, expires in ~20 days), **one declined**, **one withdrawn**; unread notifications |
| `naman@demo.aptohire.test` | Uni B, personal email, **Verified (student ID)**; brand-new: no projects, no scores (the empty states and first-project flow) |
| `avi@demo.aptohire.test` | Alumnus of Uni A, SQL **Advanced** verified; has a **pending proposal** to confirm a new affiliation at Uni B (masters, `pg`); scores must be unchanged after confirming |
| `niya@demo.aptohire.test` | Alumna of Uni B moving to Uni D (not registered): affiliation **Unverified**; scores and badges present and unaffected |

Skills seeded for all families (Web/Frontend: HTML/CSS, JavaScript, React, TypeScript, Accessibility, API integration, Testing · Backend: Node.js, Python, SQL, REST design, Auth · Data: SQL analytics, Python/Pandas, Visualization, ML basics · Management/Commerce: Market analysis, Financial modelling, Project planning, Communication) with edges (React–JavaScript, React–TypeScript, TypeScript–JavaScript, React–Testing, Accessibility–HTML/CSS, API integration–JavaScript, …). Organizations: **Northwind Labs** (mid-size corporate), **Brightlane** (start-up), both with a *Verified organization* chip.

### 4.8 What the backend project must provide (list these in your final report too)

`public_config` view · `my_affiliation_proposals` view · `agent_runs.stage` (`received|reading|watching|scoring|writing|done`) readable by the submission's owner · extra `students` columns (`location`, `languages text[]`, `target_skill_ids uuid[]`, `target_roles text[]`) · `tickets` insert policy + a trigger that sets `submissions.status='appealed'` · a safe way to update only `students.visibility` and own profile fields · `profile_proposals` storage for synced changes · `export-my-data` and `delete-my-account` functions · `evidence[].excerpt` and `audits.transcript` (optional) · `access_requests` readable together with the organization's and job's **public** columns. Treat every one as optional at runtime: if missing, hide that UI or fall back, and note it with `TODO(backend)`.

---

## 5. Auth, sessions and role guard

- **Supabase Auth, email + password** (OTP optional later). Single browser client in `src/lib/supabase/client.ts`: `storageKey: 'aptohire-student'`, `persistSession`, `autoRefreshToken`, `detectSessionInUrl`, `flowType: 'pkce'`. **No cookie shared across subdomains**; the root site and other portals have separate sessions.
- **Public routes:** `/login`, `/signup`, `/forgot-password`, `/reset-password`, `/auth/callback`. Everything else is behind `AuthGate`.
- **Sign-up:** step 1 account (full name, email, password ≥ 10 characters with a clear strength hint) → `auth.signUp` → if email confirmation is required show "Check your email" with a resend button → after confirmation `/auth/callback` runs **`useBootstrap()`**: (a) no role yet → `complete-signup({portal:'student'})`; (b) no student/affiliation → onboarding; (c) done → Home. Make every step **idempotent and resumable**.
- **Role guard (F-ID-03):** after login, fetch **only** the user's own `profiles` row. `student` → mount the app. `university` or `organization` → render the **"Go to your portal"** screen with a button to `portalUrl(role)`. Any other role (including admin) → generic screen "This account can't use the student site." with **no link and no role named**. No student data hooks may run before the guard passes (enforce with a test).
- **Sessions (F-ID-04):** short-lived tokens with refresh rotation (server setting). Account page offers **Sign out of all devices** (`signOut({scope:'global'})`). `// TODO(decision):` a session list needs a backend function; omit until provided.
- **Login page:** email + password, "Forgot password", link to sign-up, link back to `rootUrl('/')`. In demo mode, a **Demo accounts** panel with one-click buttons for the four accounts in 4.7 (uses `VITE_DEMO_PASSWORD`). This panel and its code are excluded from production builds.
- **Email change (F-ID-08):** `auth.updateUser({email})`, then show "Check your new address to confirm. Your old email works until you do." Never take over a login by roster match.
- Sign-out clears the TanStack Query cache and Realtime channels.

---

## 6. Design system (identical across all five portals)

**Concept: "Proof you can see."** Quiet, white, spacious; one thing stands out: **evidence**. Green = verified, trusted, done. Orange = in progress, attention, energy. They meet in the **Proof Ring**. White is the product. Spend boldness in one place: the Proof Ring and the evidence trail. Everything else is calm and disciplined.

### 6.1 Tokens → `src/styles/tokens.css` (mirror in `tailwind.config.ts`)

```css
:root {
  --canvas:#ffffff; --mist:#f6f8f7; --hairline:#e3e8e5; --ink:#17261e; --moss:#5b6a62;
  --verdant-50:#ecfaf2; --verdant-100:#d2f3e1; --verdant-300:#8fe0b2;
  --verdant-600:#0e9f5b; --verdant-700:#0b7f49; --verdant-800:#0a6139;
  --ember-50:#fff3e8; --ember-100:#ffe2c7; --ember-300:#ffb470; --ember-500:#ff7a1a; --ember-700:#b94a06;
  --danger-600:#c4311f; --danger-50:#fdece9;
  --grad-dawn: linear-gradient(135deg,#e6f8ee 0%,#ffffff 46%,#ffebd6 100%);
  --grad-mesh:
    radial-gradient(60% 60% at 12% 8%, #ddf6e8 0%, transparent 70%),
    radial-gradient(50% 50% at 92% 88%, #ffe7d0 0%, transparent 70%), #ffffff;
  --grad-verified: linear-gradient(135deg,#0e9f5b,#34c98a);
  --r-control:10px; --r-panel:16px; --r-feature:28px;
  --shadow-1: 0 1px 2px rgba(23,38,30,.06), 0 8px 24px -10px rgba(23,38,30,.14);
  --shadow-2: 0 2px 4px rgba(23,38,30,.06), 0 24px 48px -16px rgba(23,38,30,.22);
  --font-display:"Bricolage Grotesque Variable","Bricolage Grotesque","Segoe UI",system-ui,sans-serif;
  --font-body:"Instrument Sans Variable","Instrument Sans","Segoe UI",system-ui,sans-serif;
  --font-mono:"JetBrains Mono",ui-monospace,monospace;
  --ease-out:cubic-bezier(.2,.8,.2,1); --dur-fast:150ms; --dur-base:220ms; --dur-reveal:900ms;
  --portal-strip: var(--grad-dawn);   /* student identity */
}
@media (prefers-reduced-motion: reduce){ *{animation-duration:.001ms!important;transition-duration:.001ms!important;scroll-behavior:auto!important} }
```

Tailwind names: colors `canvas, mist, hairline, ink, moss, verdant.{50,100,300,600,700,800}, ember.{50,100,300,500,700}, danger.{50,600}`; radius `control, panel, feature`; shadow `float` (shadow-1), `modal` (shadow-2); fonts `display, body, mono`. **No hard-coded hex in components.** Light theme only.

### 6.2 Rules

- **Colour:** primary button = **white on `verdant-700`**; accent button = **`ink` on `ember-500`** (never white on orange). Use the accent button for the single most energetic action on a page: **Get a project** on Home and in empty states. *Accept request* stays primary. `verdant-600` and `ember-500` for **graphics and large elements only**; small green text `verdant-700`, small orange text `ember-700`. Colour is never the only signal (icon or word alongside).
- **Gradients:** the student identity is `dawn`, used **only** in the top bar strip and the Home header; **at most one gradient surface per viewport**; none on buttons, cards or behind text smaller than 16 px.
- **Type:** Bricolage Grotesque (headings, 600–700, scores 600) + Instrument Sans (UI, 400/500/600) + JetBrains Mono only for IDs, code and timestamps. Sentence case everywhere; no all-caps tracked labels; no eyebrow labels unless they carry information; don't accent one word of a headline. `tabular-nums` on numbers. Lines ≤ 68 characters. Left-aligned.

| Token | Size | Line-height |
|---|---|---|
| `h1` | `clamp(28px, 4vw, 36px)` | 1.2, −0.015em |
| `h2` | `clamp(22px, 2.6vw, 28px)` | 1.3 |
| `h3` | `clamp(18px, 2vw, 20px)` | 1.35 |
| `body-l` | 18px | 1.6 |
| `body` | **16px minimum** | 1.6 |
| `small` | **14px minimum** | 1.5 |

- **Spacing** (4 px base): 4 · 8 · 12 · 16 · 24 · 32 · 48 · 72. Content max **1200 px**, page padding 24 px desktop / 16 px mobile.
- **Radius encodes hierarchy:** `control` 10 (inputs, buttons; chips full-pill), `panel` 16 (request cards, dialogs), `feature` 28 (scorecard hero panel).
- **Flat by default** with hairline borders. Shadow only for what floats (`float`: menus, popovers; `modal`: dialogs, side sheets). Data lives in **tables and lists with hairlines**; cards only for things you act on (access requests, the current project).
- **Motion:** exactly one orchestrated moment, the Proof Ring reveal (7.1). Everything else responds to actions only. No fade-ups on sections, no hover lift on every card. Reduced motion → final states instantly.
- **Icons:** Lucide. **Illustrations** (empty states): small compositions of palette circles/arcs on `mesh`; no stock photography, no photos of people (the student's own avatar is the only photo).

### 6.3 App shell (this is the signature layout of the portal)

```
┌────────────┬──────────────────────────────────────────────────────┐
│ ◖ AptoHire │ [☰ at 768–1023]              🔔   Aarav ▾             │ ← 56px top bar, dawn strip behind
│ student    ├──────────────────────────────────────────────────────┤
│ Home       │   Page title (h1)                       [Primary]    │
│ Projects   │   One-line description (moss)                        │
│ Scorecard  │   ───────────────────────────────────────────────    │
│ Sharing    │   content (max 1200)                                 │
│ Profile    │                                                      │
│ Help       │                                                      │
└────────────┴──────────────────────────────────────────────────────┘
```

| Width | Behaviour |
|---|---|
| ≥ 1280 | Full 232 px rail, side sheets open beside content |
| 1024–1279 | Rail collapses to 72 px icons with tooltips |
| 768–1023 | Rail becomes a **drawer** opened by a hamburger (☰/✕, 44×44, `aria-expanded`, focus trap, Escape closes and returns focus, closes on navigation and on resize ≥ 1024) |
| < 768 | **Bottom tab bar**: Home · Projects · Scorecard · Sharing · **More** (a sheet with Profile, Help, Account & security, Sign out). Sheets become full-screen; scorecard ring 160 px; evidence rows stack; skill graph defaults to list view |

Nav items are **exactly**: Home · Projects · Scorecard · Sharing · Profile (Help and Account live in the rail footer / More). Show an unread-count badge on Sharing for pending requests (with a text alternative). Skip-to-content link first. Minimum width **360 px**. Respect `env(safe-area-inset-*)` for the top bar and bottom tab bar.

---

## 7. Signature components

### 7.1 `ProofRing`

Props: `score: number | null`, `skill: string`, `state: 'verified'|'not-verified'|'under-review'|'loading'`, `size: 56|96|160|240`, `level?`, `animate?`, `showCaption?`, `passMark`.

- Track `hairline`, stroke 12 px at 240 (scale down). Arc: SVG circle, **round caps**, length = score ÷ 100 from 12 o'clock clockwise, gradient `#FF7A1A → #FFB470 (30%) → #8FE0B2 (60%) → #0E9F5B` (a `linearGradient` is fine). A small **tick at the pass mark** (50%). Centre: score in Bricolage 600 (64 px at 240) with the level caption below.
- `verified`: seal (check in `grad-verified`, 40 px at 240) on the ring top-right; caption "Verified · Proficient". `not-verified`: solid `ember-500` arc; caption "Not verified yet · 9 points to go". `under-review`: arc `ember-300` with a dashed final segment; caption "Under human review". `loading`: track + skeleton.
- A11y: `role="img"`, `aria-label="React, score 72 of 100, Proficient, verified"`; the number is real text.
- **Reveal:** arc draws 0 → score over **900 ms** `ease-out`, number counts up in sync, seal scales 0.8 → 1 over a 180 ms settle. Plays **once** when a new score first appears (and once per page entry for the scorecard hero); reduced motion → final state immediately. Unit-test the caption logic and aria-label builder.

### 7.2 Evidence trail (`EvidenceRow`, `EvidenceChip`, `EvidenceSheet`)

Beside every score, a list of rows: criterion name, score bar (`verdant-600` on `mist`), number, one-line rationale, **evidence chips**. `insufficient_evidence` shows "Not enough evidence to score this" with a neutral bar and lowers the displayed confidence. Selecting a chip opens the **side sheet** (full-screen on mobile) with the actual evidence:

- `video` → a player that **seeks to the timestamp** (signed URL, created on demand) with the **transcript as text** beside it (current line highlighted). If no video file is available (mock), show the transcript at that time only.
- `repo` / `file` → path and line range, the `excerpt` in JetBrains Mono with line numbers if present, plus **Open on GitHub** (built from the submission's GitHub URL) in a new tab with `rel="noopener noreferrer"`.
- `doc` → file name and excerpt if present. `live` → the URL, **Open live site**.

### 7.3 `StatusChip`

| Chip | Look |
|---|---|
| Verified | `verdant-100` fill, `verdant-800` text, seal icon |
| Under review | `ember-100` fill, `ember-700` text |
| Pending | `ember-50` fill, `ember-700` text |
| Unverified university | white, dashed `hairline` border, `moss` text |
| Verified university | `verdant-50` fill, small seal |
| Verified organization | `verdant-50` fill, small seal |
| Declined / Closed / Withdrawn / Expired | `mist` fill, `moss` text |
| Auditing | `ember-50` fill with a pulsing `ember-500` dot (pulse disabled under reduced motion) |

### 7.4 `SkillGraph`

`d3-force` layout (lazy). Nodes sized by score; **verified** = `verdant-600` fill + check; **in progress** = `ember-500` outline; **not assessed** = dashed hairline. Edges 1 px `#D5DDD8`. Hover/focus highlights neighbours; nodes are focusable with **arrow-key navigation**; selecting a node opens a panel with its projects and evidence. **Always paired with a list-view toggle**; list is default below 768 px and for `prefers-reduced-motion` (no animated simulation: compute the layout statically).

### 7.5 `ScoreHistory`

Recharts line chart of `skill_score_history` per skill with a **"View as table"** toggle (accessible alternative). Empty (< 2 points): "Your history appears after your second review."

### 7.6 Other UI

Buttons (primary / accent / secondary / ghost / danger; height 44 mobile, 40 desktop; label 15/600; loading keeps label and fixed width), Field (label **always above**, helper 14 px, error with icon, `aria-describedby`), Tabs (underline, 2 px `verdant-700`), Table (header `mist`, hover `mist`, sticky header, scrolls **inside its own container**; below 640 px convert key tables to labelled lists), Dialog and Sheet (focus trap, Escape, restore focus), Toast (bottom-centre, 4 s, reuses the action's verb, includes Undo where possible), Skeleton (matches final layout, shimmer disabled under reduced motion), EmptyState (palette circles on `mesh`, one sentence, one button), Switch, Checkbox (44 px hit area), Stat (plain figure + label inline, no card; rows divided by hairlines).

---

## 8. Screens and flows

Routes: `/login` · `/signup` · `/forgot-password` · `/reset-password` · `/auth/callback` · `/onboarding` · `/` Home · `/projects` · `/projects/new` · `/projects/:assignmentId` · `/submissions/new` (query `?assignment=` or `?own=1`) · `/submissions/:id/edit` · `/submissions/:id` · `/scorecard` · `/scorecard/:skillSlug` · `/sharing` (tabs) · `/profile` (tabs) · `/resume` · `/notifications` · `/account` · `/help` · `*` 404. Every page: `h1`, document title, loading skeleton, empty state, error state with **Retry**, and `noindex`.

### 8.1 Onboarding and affiliation (F-ST-01..07)

After sign-up, `/onboarding` is a short guided flow (3 steps, progress shown in words "Step 2 of 3"):

1. **About you:** full name (prefilled), program level (Undergraduate / Postgraduate), year of study or status (Incoming · Year 1–5 · Alumni · Dropout).
2. **Your university:** searchable picker of registered universities plus **"My university isn't listed"**. The result is driven by `register-student.outcome`:
   - `verified_roster` → success panel with the **Verified university** chip: "We matched you to Uni A. Your university is verified."
   - `needs_student_id` → show a student-ID field (never stored client-side after submit; `autocomplete="off"`, masked paste-safe) with copy: "Enter your student ID. We store only a secure hash. Never upload an ID card." → `verified_student_id` or offer **Continue without verifying** (→ `unverified`).
   - `unverified` → panel with the dashed chip and exactly: "Your university isn't on AptoHire yet. Your scores and job matches are not affected."
   - `confirm_existing` → "We found your existing profile" → **Confirm new university** flow (8.1.2).
   - `roster_no_match` → "We couldn't find you on this university's list." with options: try a student ID, or continue as unverified. Don't reveal which roster fields were close.
3. **Your target (F journey step 2):** choose a target skill (chips grouped by family, searchable) or type a role ("Frontend developer"); up to 3. Saved to the profile. **Skip for now** is allowed.

#### 8.1.2 Claim and merge (F-ST-07)

If `my_affiliation_proposals` has rows, show a **persistent but dismissible banner** on Home and a dedicated panel on Profile → Education: "Uni B added you to its list. Confirm your new university." with details (university, program level, year, verification chip) and **Confirm** / **Not me**. Confirm calls `confirm-affiliation({proposal_id})` and then shows: the old affiliation moves to Alumni, the new one is current, "Your scores and credential didn't change." **No second account is ever created.** Test with Avi.

#### 8.1.3 Adding a university yourself

Profile → Education → **Add a new university** (e.g., Niya → Uni D): registered university (with ID path) or free-text name → `confirm-affiliation({new:…})`. Unverified affiliations are **self-editable** (status, year); verified ones are read-only with the note "Managed by your university."

### 8.2 Home `/`

Header with the `dawn` strip. H1: "Welcome back, Aarav." Below it a single **"What to do next"** block that adapts to state (priority order): pending affiliation proposal → review in progress (mini progress + link) → score just ready ("Your React score is ready." → Scorecard) → pending access requests ("2 requests are waiting for you." → Sharing) → project assigned or draft in progress ("Continue: <title>") → no projects (accent button **Get a project**). Then:

- A plain **Stat row** (hairline-divided, no cards): Verified skills · Projects reviewed · Active sharing.
- **Your skills:** list rows with 56 px rings, level chip, "Verified"/"Under review"/"Not verified yet"; empty → "No skills yet. Get your first project."
- **Requests waiting** (max 3 `AccessRequestCard`s) when any.
- **Suggested next project** from the weakest criterion/next level (client-side rule; links to `/projects/new?skill=…`).

### 8.3 Projects (F-ST-10..13, 12)

**List `/projects`:** table with status chips: **Assigned → In progress → Submitted → Auditing → Scored / Needs review / Appealed / Failed**. Mapping: assignment without submission = *Assigned* (or *In progress* once started); `draft` = *In progress*; `submitted` = *Submitted*; `auditing` = *Auditing*; `scored` = *Scored* (or *Needs review* when the audit is borderline and review is pending); `needs_review` = *Needs review*; `appealed` = *Appealed*; `failed` = "Review failed · Retry". Filters as tabs (All · Active · Scored). Primary button **Get a project**; secondary **Upload my own project**. Empty: "No projects yet. Pick a skill and get your first project." → *Get a project*.

**Project assistant `/projects/new` (F-ST-10):** a guided, chat-style screen without gimmicks (no typing animation). "What do you want to get hired for?" → choose a skill or type a role (`?skill=` prefills) → shows the **target level** ("We'll aim for Proficient, one step above your current Competent in React." from `skill_scores`; override via select limited to levels above current, defaulting to Competent for new learners) → **Get project** → loading brief skeleton ("Building a project at your level…", 20 s timeout with Retry) → navigates to the brief. Handle `NO_TEMPLATE` ("Projects for <skill> aren't available yet. You can upload a project you've already built." with a button to `/submissions/new?own=1`), `QUOTA_EXCEEDED` ("You've reached today's limit for new projects. Try again tomorrow."), `KILL_SWITCH` ("New projects are paused. Try again later.").

**Brief `/projects/:assignmentId`:** title, level chip, "About N hours", scenario, **deliverables as a checklist** (progress stored per assignment in `localStorage`, try/catch), constraints, suggested stack chips, **rubric preview** as a plain table with weight bars, **What to show in your 3-minute video** (`video_guidance`), the **originality questions** (answered later on the submission form). Actions: **Start project** (sets in progress, goes to `/submissions/new?assignment=:id`) and secondary **Get a different project**.

**Upload my own project (F-ST-13):** same submission form without an assignment; the skills-claimed field is required.

### 8.4 Submission form `/submissions/:id/edit` (F-ST-20..24)

A single, calm page with sections (not a wizard): **Project** (title 3–120, description ≤ 2000, skills claimed: multi-select from the taxonomy, at least one) · **Demo video** (required when config says so) · **Files** (documents) · **Links** (live URL, GitHub URL; validate `https` and `github.com/owner/repo`) · **Originality** (the assignment's 2–3 questions, text answers ≤ 600 each, or tick "I'll answer these in my video") · **AI use** (Did you use AI tools? yes/no; tools multi-select: ChatGPT, Claude, Copilot, Gemini, Cursor, Other; notes ≤ 300). Helper text under AI use: "Telling us doesn't lower your score. We review what you built and whether you understand it."

- **Autosave (F-ST-22):** debounced 1.5 s upsert of the `draft` row; a quiet "Saved just now" status (`aria-live="polite"`); "Draft resumed" on reload; drafts are deleted after 30 days (state it in helper text).
- **Video dropzone (F-ST-21):** dashed hairline panel; on select run **client checks before upload**: type (mp4/webm/mov), size ≤ `VIDEO_MAX_MB`, duration via a hidden `<video>` `loadedmetadata` ≤ `VIDEO_MAX_SECONDS`. Failure copy exactly: **"This video is 3:42. Videos must be 3:10 or shorter. Trim it and upload again."** (use the real duration/limit; for size: "This video is 74 MB. The limit is 50 MB. Compress it and upload again."). On success show preview (object URL), duration and size, then **resumable upload (TUS)** with progress bar (`verdant-600`), pause/resume, cancel, and automatic retry on network loss. Revoke object URLs on unmount. **Fallback:** "Can't upload? Add an unlisted video link instead" (https URL field; allowed hosts not restricted but must be a valid URL). With `REQUIRE_VIDEO=true`, the form cannot be submitted without an uploaded video **or** a video link.
- **Documents (F-ST-24):** extension allow-list pdf/docx/md/zip, ≤ 20 MB each, max 5; verify the **file signature (magic bytes)** matches the extension (`%PDF`, `PK` for docx/zip; `.md` must be valid UTF-8 text); show per-file progress and remove buttons.
- **Submit:** button **Submit project**. First validate everything with inline errors and move focus to the first invalid field. Then a **confirm dialog** with two required checkboxes: "I've told you accurately how I used AI tools" and "I agree to my project being reviewed by AI and by a human reviewer if needed." → `submit-project({submission_id, confirm:true})`. Map error codes to copy (section 9). On success: toast "Project submitted. We're reviewing it now." and navigate to `/submissions/:id`.

### 8.5 Review progress `/submissions/:id` (F-ST-30)

- **Status `submitted`/`auditing`:** vertical **audit progress** list: Received · **Reading your code and files** · **Watching your demo video** · **Scoring against the rubric** · **Writing your feedback** · Ready. Icons: done (green check), active (ember dot with subtle 1.2 s pulse), waiting (hollow). Wrapper `aria-live="polite"` announces step changes. Driven by `agent_runs.stage` (4.5/4.8); if `stage` is unavailable show the indeterminate "We're reviewing your project" with elapsed time and **no fake step progress**. Headline: "Project submitted. We're reviewing it now." Text: "You can leave this page. We'll notify you when your score is ready." Long waits show the steps, not a spinner.
- **Status `scored`:** banner "Your React score is ready." with the Proof Ring (reveal plays once) and **View scorecard**.
- **`needs_review`:** "Under human review. A reviewer will confirm your result within 3 working days." with the under-review ring state.
- **`failed`:** "We couldn't finish the review. Retry now, or contact support if it happens again." with **Retry** (calls `submit-project` again) and a support mailto.
- **Kill switch:** "Reviews are paused" calm message.
- Always show a summary of what was submitted (title, links, files, AI-use disclosure) in a plain description list.

### 8.6 Scorecard (F-ST-31..37)

**`/scorecard`:** list of the student's skills grouped by family: each row = 96 px ring, skill, level chip, verified/under-review state, last reviewed date, link. Above it a **list ⇄ graph** toggle showing the `SkillGraph` (40–60 skills not assessed appear dashed). Buttons: **Download resume** (→ `/resume`) and **Get a project**.

**`/scorecard/:skillSlug`** (follow this layout):

```
Scorecard › React                                       [Download resume]
┌────────────────────┐   Proficient React developer
│   ◯  72            │   Reviewed 12 Oct · Confidence high
│   Verified ✓       │   "<audit.summary>"
└────────────────────┘
Evidence
 Functionality           ▇▇▇▇▇▇▇▁▁▁ 74   <rationale>        [Live site] [src/cart.tsx]
 Code quality ...                                            [src/hooks/…]
 ...
────────────────────────────────────────────────────────────────────────
 Score history (chart ⇄ table)            Next steps (≤ 3, from the audit)
                                          [Get a project]   (prefilled /projects/new?skill=…)
 Not happy with this result?  [Raise a ticket]
```

- Hero panel `r-feature` with the 240 px ring (160 px below 768). **Based on** selector: the project/audit shown, with other audits listed.
- States: **Verified** ("You're verified in React at the Proficient level."), **Not verified** ("Not verified yet. You're 9 points away. Here's what to improve." + next steps), **Under review** (borderline: "Under human review. A reviewer will confirm your result within 3 working days."; show the score but mark it pending).
- **Appeal (F-ST-36):** **Raise a ticket** opens a dialog (reason 20–1000 characters, counter, "A different reviewer will look at this."). Inserts into `tickets`; the page then shows the ticket timeline: *Ticket received → Acknowledged → Under review by a different reviewer → Decision* with the recorded decision and note. Never show reviewer identity. Disable if an open ticket exists for that submission.
- **Next-step suggestions (F-ST-35):** use `audit.next_steps`; always include the link to the project assistant.
- Flags (`inconsistent_claims`, `possible_copied_work`, `video_unusable`, `repo_unreachable`) are **not shown as accusations**; show neutral guidance ("We couldn't open your repository. Check that it's public and resubmit.") for `repo_unreachable` / `video_unusable`, and nothing for the others (the human review handles them).

### 8.7 Sharing and consent (F-ST-50..53) `/sharing`

Tabs: **Requests** · **Active access** · **History** · **Feedback**.

- **Requests:** `AccessRequestCard` (cards are allowed here: these are things you act on): organization name + **Verified organization** chip, the job title, their message, the data they ask for as plain words (Profile · Scorecard · Projects · Contact details), duration. Copy: "Northwind Labs would like to see your profile and scorecard for 30 days." Action **Review request** opens the **Consent sheet** (`modal` shadow; full-screen on mobile): what they asked for; a **checkbox per scope with a plain description** (Profile: "Your name, photo, headline, bio and university" · Scorecard: "Your verified skills, levels and evidence summary" · Projects: "Your project files and videos" · Contact details: "Your email address"); duration select **7 / 30 / 90 days**; **Accept request** (primary) and **Decline** (secondary); footnote "You can withdraw access any time from Sharing." Pre-check only the scopes they requested; the student may uncheck. Accept needs at least one scope. Toasts: "Access shared until 12 Nov." / "Request declined." Nothing is revealed client-side; the server decides.
- **Active access:** list of grants (organization, scopes, **expires on**), **Withdraw access** with a confirm dialog → toast "Access withdrawn." Empty: "No active access. When you accept a request it will appear here."
- **History:** declined, withdrawn, expired (quiet `mist` chips).
- **Feedback (F-ST-53):** structured options from organizations mapped to friendly labels + optional note (≤ 280 characters). Empty: "No feedback yet."
- Empty (Requests): "No requests yet. When an organization asks to see your profile, it will appear here."
- Explain borderline: if the student has a skill under review, a quiet note "Results under human review aren't shared with organizations until confirmed."

### 8.8 Profile `/profile` (F-ST-05, 06, 04, 40, 41, 43)

Tabs: **Overview** · **Education** · **Connected sources** · **Visibility**.

- **Overview:** photo (crop + re-encode, 1 MB cap, remove button), full name, headline (≤ 80), bio (≤ 500), location, languages, links (website, GitHub, LinkedIn, portfolio; validated https), target skills/roles. Autosave per field group with "Saved" feedback. Show the student's **credential ID** in mono with a copy button and a link **View my public page** → `rootUrl('/verify/' + credential_id)` (new tab).
- **Education:** timeline of affiliations (current first): university (or the free-text name), program level, **lifecycle chip** (Incoming · Enrolled (year N) · Dropout · Alumni · Masters), verification chip (Verified university / dashed Unverified), dates, plus the proposals panel (8.1.2) and **Add a new university** (8.1.3). For unverified affiliations show: "Your university isn't on AptoHire yet. Your scores and job matches are not affected."
- **Connected sources (F-ST-40/41):** one row per source: **GitHub** (**Connect** → `profile-sync github_start` → redirect to GitHub's authorize URL; read-only; after return `/profile?tab=sources&connected=github`), **LinkedIn** (**Upload your data export** or **Paste your profile text** ≤ 20,000 characters; copy: "AptoHire never accesses LinkedIn directly. You provide your own export."), **Manual links**. Each source has its own **consent switch** (revocable), "Last synced 2 days ago", **Sync now**, and **Disconnect** (dialog with a checkbox "Also delete data we derived from this source"). Below, **Suggested updates** (`ProfileProposal`s): list of changes with the evidence line ("From your GitHub repository…"), **Add** / **Dismiss** per change, and a per-source **Approve automatically** switch. Copy must state: "Synced data can add to your profile and evidence. It never changes your verified scores." `// TODO(backend)` if proposals aren't available: hide the suggestions panel.
- **Visibility (F-ST-43):** two groups of switches. **Public verification page** (what anyone with your link or QR can see): Skills (on), Levels (on), Scores (off), Name (off), University (off). **Discovery:** "Let organizations find my skills (without my name)". A live **preview** of the public page using the same fields (static component) and an "Open my public page" link. Plain note: "Organizations never see your name until you accept a request." Saved through the allowed update path (4.8).

### 8.9 Resume `/resume` (F-ST-42)

Web view of the **digital resume**: name, headline, current affiliation with verification chip, verified skills with levels (and scores only if allowed by Visibility), approved profile content, and a **QR code + link** to `rootUrl('/verify/' + credential_id)` (QR generated with `qrcode`). **Download PDF** dynamically imports `@react-pdf/renderer`, builds an **A4** document with the same content (register fonts from static `woff` files of Bricolage Grotesque 700 and Instrument Sans 400/600; fall back to Helvetica if registration fails), the seal mark, and the QR; the file is named `AptoHire-Resume-<Name>.pdf`. Loading state on the button ("Preparing your resume…"). Print stylesheet for the web view. Only verified skills appear; skills under human review are excluded until confirmed.

### 8.10 Notifications (F-ST-54, F-XC-03/04)

Bell in the top bar with an unread count (`aria-label="Notifications, 3 unread"`) opening a popover (latest 8) with **Mark all as read** and **View all** (`/notifications`, grouped Today / Earlier). Kinds and deep links: `audit_ready` → scorecard skill · `review_needed`/`review_decided` → scorecard · `appeal_update` → scorecard ticket · `access_request` → Sharing · `access_expiring` → Active access · `affiliation_proposal` → Profile → Education · `feedback_received` → Sharing → Feedback · `source_proposals` → Profile → Sources. Realtime inserts show a toast when the tab is open. Empty: "You're all caught up."

### 8.11 Account `/account` and Help `/help`

- **Account:** email (change flow 5), password change, **Sign out of all devices**, **Download my data** (`export-my-data`), **Delete my account** (dialog: password re-entry + type `DELETE`; explains what is deleted and that organizations lose access), and in demo mode **Reset demo data**.
- **Help:** short page: how reviews work in plain words, what Verified means, appeals, sharing, contact support (mailto), and links to `rootUrl('/ai-and-fairness')`, `rootUrl('/privacy')`, `rootUrl('/terms')`.

### 8.12 System screens

Wrong-portal screen (5), 404 ("That page doesn't exist." + **Go home**), `ErrorBoundary` (**Reload page**; never a stack trace), offline banner ("You're offline. Changes will save when you reconnect." and queue autosaves), and the **PrototypeNotice** (slim bar, dismissible per session) when `VITE_APP_ENV !== 'production'`: "Prototype. All people, universities and organizations shown are fictional."

---

## 9. Copy and voice

Plain, warm, specific. Sentence case. Active verbs. **No hype words**, no "AI-powered" headlines. Say what happened, not how the system works. Errors say what happened and the fix and **don't apologize**. Vocabulary (use consistently): **Project · Submit · Review · Scorecard · Verified · Evidence · Request access · Accept · Decline · Sharing · Withdraw access.** Avoid "audit engine", "agent", "pipeline", "RLS", "token" in the UI. A button's verb is reused in its toast.

| Moment | Copy |
|---|---|
| Submit button | Submit project |
| After submit | Project submitted. We're reviewing it now. |
| Review steps | Reading your code and files · Watching your demo video · Scoring against the rubric · Writing your feedback |
| Score ready | Your React score is ready. |
| Verified | You're verified in React at the Proficient level. |
| Not verified | Not verified yet. You're 9 points away. Here's what to improve. |
| Borderline | Under human review. A reviewer will confirm your result within 3 working days. |
| Video too long | This video is 3:42. Videos must be 3:10 or shorter. Trim it and upload again. |
| Access request | Northwind Labs would like to see your profile and scorecard for 30 days. |
| Accept / toast | Accept request · Access shared until 12 Nov. |
| Withdraw / toast | Withdraw access · Access withdrawn. |
| Unverified university | Your university isn't on AptoHire yet. Your scores and job matches are not affected. |
| Failed review | We couldn't finish the review. Retry now, or contact support if it happens again. |
| Reviews paused | Reviews are paused. Your project is saved. Try again later. |
| Rate limited | Too many attempts. Try again in a few minutes. |
| Network error | We couldn't reach AptoHire. Check your connection and try again. |
| Session expired | Your session ended. Sign in again to continue. |

Error-code mapping: `VIDEO_REQUIRED` "Add your demo video or a video link before submitting." · `MISSING_SKILLS` "Choose at least one skill this project shows." · `QUOTA_EXCEEDED` "You've reached today's limit for reviews. Try again tomorrow." · `KILL_SWITCH` "Reviews are paused…" · `VALIDATION_FAILED` map `fields` to inputs · unknown → network error copy.

All strings live in `src/content/*.ts` (or per-feature `copy.ts`) for easy editing.

---

## 10. Quality bars

### 10.1 Responsiveness (built in, not deferred)

Breakpoints: **Mobile < 768 (base) · Tablet 768–1023 · Desktop ≥ 1024**. Tailwind `md` 768, `lg` 1024, `xl` 1280.

- `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">`; safe-area insets on fixed bars.
- **No horizontal scroll from 360 px up.** Fix root causes; never `overflow-x:hidden` on `body` as a patch. Tables, code, and the graph scroll inside their own container.
- Multi-column layouts collapse below 768; flex rows wrap or stack; mobile section padding ≥ 40 px vertical; nothing touches the screen edge.
- Body ≥ 16 px, small ≥ 14 px, line-height ≥ 1.4; headings use the fluid tokens.
- **Touch targets ≥ 44×44 px** (buttons, icon buttons, tab bar items, chips that act, checkboxes, table row actions).
- **Forms:** inputs 100% width on mobile, **font-size ≥ 16 px**, labels above inputs, form buttons full-width on mobile, correct `type`/`inputMode`/`autocomplete`.
- Test at **360, 375, 390, 768, 1024, 1440**.

### 10.2 Accessibility (WCAG 2.2 AA)

Landmarks, one `h1` per page, skip link, **visible 2 px `verdant-700` focus outline with 2 px offset plus a 4 px white halo** (never removed), all flows keyboard-operable, focus trapped and restored in dialogs/sheets/drawers/menus, `aria-live="polite"` for autosave, upload and review progress, errors announced and focus moved to the first invalid field, colour never the only signal, reduced motion respected, captions/transcript available for video evidence, skill graph has a list alternative and arrow-key navigation. `@axe-core/playwright` zero serious/critical on every route.

### 10.3 Performance

Mobile Lighthouse ≥ 90 (Performance), ≥ 95 (Accessibility, Best Practices). Route-level code splitting; **lazy-load** `d3-force`, `recharts`, `tus-js-client`, `@react-pdf/renderer`, `qrcode`; initial JS for login and Home ≤ **200 KB gzipped**. Skeletons that match final layouts. Virtualise nothing unless lists exceed 200 rows.

---

## 11. Security and privacy rules (never break)

1. **No secrets in repo or bundle.** Only `VITE_*`. Never the service-role/secret key. `.env*` (except `.env.example`) git-ignored. Build fails on `service_role`, `sb_secret`, or hard-coded hex in components (static script).
2. **Never trust the client for role, ownership or consent.** The UI hides things for convenience; **RLS and Edge Functions decide**. Never solve a permission error by moving a check into the frontend or weakening a query.
3. **Query only the signed-in student's own rows and only the columns rendered.** The app must not contain code that requests other students' data, `roster_entries`, `candidate_profiles` or `match_results`.
4. **No PII in logs, analytics, error messages, URLs or storage.** The student ID number is held only in the form until submit, then cleared. Allowed `localStorage`/`sessionStorage` keys: Supabase session (`aptohire-student`), deliverables checklist progress, notice dismissals, mock state (mock mode only).
5. **Uploads are untrusted:** type allow-list, size/duration caps, magic-byte checks, generated safe file names (`demo.ext`, sanitised document names), avatars re-encoded client-side. Never render uploaded content as HTML. Signed URLs are short-lived and never stored.
6. **Render all API strings as text** (React escaping). **No `dangerouslySetInnerHTML`.** Treat audit summaries, transcripts, org names and messages as untrusted text.
7. **External links** use `rel="noopener noreferrer"`; cross-site links built only via `links.ts`.
8. **Demo-only code** (demo switcher, mocks, reset, prototype notice) is excluded or disabled in production.
9. **Identity reveal is never simulated in the real backend implementation:** the UI shows only what the server returns for the grant. In mock mode, label any mocked reveal data "Sample".
10. Pin exact dependency versions; run `npm audit` and report results.

---

## 12. Deployment (its own Vercel project)

`vercel.json`: rewrite everything not a file to `/index.html`. Headers on all routes: `Content-Security-Policy` (`default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob: https://*.supabase.co; media-src 'self' blob: https://*.supabase.co; font-src 'self'; connect-src 'self' https://*.supabase.co wss://*.supabase.co; frame-ancestors 'none'; base-uri 'self'; form-action 'self'; object-src 'none'; worker-src 'self' blob:`), `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy: camera=(), microphone=(), geolocation=()`, `X-Frame-Options: DENY`, `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`, `Cross-Origin-Opener-Policy: same-origin`, `X-Robots-Tag: noindex`. Cache hashed assets `immutable`; `index.html` `no-cache`.

README must explain: deploy as **its own Vercel project** with domain `student.aptohire.com`; set the `VITE_*` variables; in Supabase **Authentication → URL Configuration** add `https://student.aptohire.com/**` (and `http://localhost:5174/**` for development); the other four portals are separate projects; Vercel's free Hobby plan is for non-commercial demos.

---

## 13. Testing

| Level | Cover |
|---|---|
| Unit (Vitest) | `scoring.ts` (levels at 0, 50, 51, 69, 70, 84, 85, 100; verified only **above** pass mark; borderline range from config), ProofRing caption/aria builder, file validators (types, sizes, magic bytes), video duration/size messages (exact copy), status mapping table (8.3), GitHub/URL validators, confidence labels, Zod schemas (valid and invalid), `links.ts`, `env.ts`, mock review generator determinism |
| Component (RTL) | RoleGuard (student ok; university/org → "Go to your portal"; admin → generic screen without link; **no data hooks fire before the guard passes**), onboarding branches for each `register-student.outcome`, submission form (autosave, video rejection, document rejection, REQUIRE_VIDEO gating, confirm dialog, error mapping), audit progress (stage changes announced, failed → Retry, no fake progress without `stage`), consent sheet (scope checkboxes, durations, accept needs ≥ 1 scope), withdraw dialog, appeal dialog, evidence sheet per kind, skill graph keyboard navigation and list toggle, notification deep links |
| E2E (Playwright, mock mode) | Demo steps for the student: Aarav signs in via the demo panel → asks for a React project → starts it → submits (with a small sample video file and a document) → watches the review → scorecard with ring reveal and evidence chips → accepts the pending request with a reduced scope → sees active access → withdraws it → downloads the resume PDF (assert download) → opens the public page link. Also: Naman empty states and first project; Avi confirm-new-university keeps scores; Niya unverified chip with unchanged scores; `[fail]` title shows Retry; 4-minute video rejected with exact copy; wrong-role login; axe on every route; **no horizontal overflow at 360, 390, 768, 1024, 1440**; keyboard-only run of the submission and consent flows; reduced-motion run |
| Static | `typecheck`, `lint`, secret/hex grep over `src/` and `dist/` |

---

## 14. README

Beginner-friendly (short sentences, numbered steps): what this project is, how it relates to the other four portals and the shared database, prerequisites (Node LTS, Git), install, copy `.env.example`, **run in mock mode**, how to point it at a real Supabase project (URL + anon key, set `VITE_USE_MOCKS=false`, run `npm run db:types`), demo accounts table (section 4.7, emails only), commands table, folder tour, deployment to Vercel, how to edit copy and tokens, common problems (white page → env typo; "permission denied" → wrong role; upload fails → size/duration; Supabase project paused → Restore), glossary (RLS, anon key, signed URL, mock mode, verified, borderline). State clearly: **this app never needs the service-role key.**

---

## 15. Build order (finish each phase, run the checks, then continue)

1. **Foundations:** Vite/TS/Tailwind, tokens, fonts, `env.ts`, `links.ts`, router, providers, `Backend` interface, **mock backend skeleton**, Supabase client module, lint/test/e2e setup.
2. **Design primitives and shell:** Button, Field family, Dialog/Sheet/Toast/Tabs/Table/Skeleton/EmptyState, `StatusChip`, Logo/SealMark, `AppShell` (rail, top bar, drawer, bottom tab bar, More sheet), PrototypeNotice.
3. **Auth:** login (+ demo panel), sign-up, callback, `useBootstrap`, `RoleGuard`, wrong-portal screens, forgot/reset password, account menu, tests.
4. **Onboarding and affiliation:** all outcomes, claim and merge, add-university, profile basics.
5. **Config and scoring:** `public_config` hook, `scoring.ts`, `ProofRing` (all states + reveal) with tests.
6. **Projects:** assistant, brief, list, own-project path.
7. **Submission:** form, autosave, video (TUS) and document handling, confirm dialog, error mapping.
8. **Review and scorecard:** progress page with Realtime + polling fallback, scorecard list/detail, evidence rows and sheet, skill graph, history, appeals.
9. **Sharing and notifications:** requests, consent sheet, active access, history, feedback, bell, notification centre, Realtime.
10. **Profile extras and resume:** sources, proposals, visibility + preview, resume web view and PDF.
11. **Account, help, system screens, Home dashboard.**
12. **Hardening:** real-backend `supabase/` implementation of every `Backend` method, axe, Lighthouse, overflow checks, keyboard and reduced-motion passes, secret/hex greps, `npm audit`, README.

---

## 16. Acceptance checklist

- [ ] `typecheck`, `lint`, `test`, `build`, `e2e` pass; no `any`; no leftover `console.log`
- [ ] Entire app works with `VITE_USE_MOCKS=true` and no network; switching to the real backend needs only env changes plus the backend items in 4.8
- [ ] Role guard works for student, university/organization, and admin; no student data fetched before the guard passes; admin is never linked or named
- [ ] Onboarding handles all six `register-student` outcomes; unverified affiliation never changes scores; Avi's confirm keeps scores and creates no second account
- [ ] Submission cannot be sent without a video or video link when required; a 3:42 video is rejected with the exact copy; resumable upload with progress; files validated by magic bytes
- [ ] Review progress is driven by real status/stage, announced to screen readers, recoverable on failure (Retry), and never fakes "Ready"
- [ ] Scorecard: Proof Ring in all four states, reveal plays once, every score has openable evidence (video seek + transcript, code excerpt, links), skill graph with list alternative, history with table alternative, appeal flow with ticket timeline
- [ ] Borderline results show "Under human review" and are excluded from the resume
- [ ] Consent sheet: scope checkboxes, 7/30/90 days, accept/decline, withdraw; toasts reuse the action's verb
- [ ] Profile: photo re-encoded, visibility switches with live preview, sources with per-source revocable consent, QR resume PDF downloads and opens the public page
- [ ] Verified pass mark and borderline range read from config, never hard-coded in UI logic
- [ ] At most one gradient surface per viewport; none on buttons/cards/text; sentence case; no tracked all-caps labels; contrast rules respected (white on `verdant-700`, `ink` on `ember-500`)
- [ ] Bottom tab bar < 768, drawer 768–1023, icon rail 1024–1279, full rail ≥ 1280; touch targets ≥ 44 px; inputs ≥ 16 px; no horizontal scroll at 360–1440
- [ ] Axe: zero serious/critical on every route; Lighthouse targets met; initial JS ≤ 200 KB gzipped
- [ ] No secrets or PII in repo/bundle/logs; demo code excluded from production builds; production build fails if mocks/demo are on
- [ ] README and `docs/PLAN.md` complete; every `TODO(decision)` and `TODO(backend)` listed in the report

---

## 17. Final report (output exactly this at the end)

### ✅ Student portal build report

**Stack used:** [versions]

**Routes built:** [list]

**Backend interface:** [methods implemented in `mock/` and in `supabase/`; any method still mock-only]

**Backend items required (4.8) and their runtime fallbacks:** [list; mark which are blocking]

**Decisions made (`TODO(decision)` / `TODO(backend)`):** [each item and the default chosen]

**Quality results:**
- Lighthouse (mobile): Performance [x] · Accessibility [x] · Best Practices [x]
- Initial JS (gzipped): login [x KB] · Home [x KB]
- Axe violations (serious/critical): [0 per route]
- Overflow check at 360 / 375 / 390 / 768 / 1024 / 1440: [pass/fail]

**Navigation:** [bottom tab bar, drawer and rail behaviours verified]

**Files created:** [grouped list]

**What to check manually:**
- Chrome DevTools → Device Toolbar (Ctrl+Shift+M) at 375, 390, 768, 1024 and 1440 px
- Sign in as each of the four demo accounts and run the Aarav flow end to end with the keyboard only
- Turn on "reduce motion" in your OS: the Proof Ring and progress pulse should show final states instantly
- Upload a video longer than 3:10 and a file with a wrong extension; confirm the messages
- Submit a project titled `[fail] test` to see Retry
