# PROMPT 4 of 5 — `organization.aptohire.com` (Organization portal)

> Paste everything below this line into your AI coding tool inside an **empty folder** named `aptohire-organization/`.
> This project is **independent**. It does not import from, or share code with, the other portals. Where this prompt says "copy", copy the file; never create a cross-project import.

---

## 0. Your role and how to work

You are a senior frontend engineer and product designer. Build the **Organization portal of AptoHire** (`organization.aptohire.com`) for recruiters, as a complete, production-quality Vite + React + TypeScript app that talks to the **shared Supabase backend**.

This is the portal where **privacy is the product**: recruiters find verified talent **without seeing who it is**, and identity appears only after a student says yes. Every decision below protects that promise.

Work in this order:

1. **Plan (short).** Write `docs/PLAN.md` (max 1 page): route list, feature modules, the `Backend` interface (4.1), what you will mock. Then build.
2. **Build** in the phases of section 15. After each phase run `npm run typecheck && npm run lint && npm run test` and fix failures before moving on.
3. **Verify** against section 16 at 360, 390, 768, 1024 and 1440 px.
4. **Report** in the exact format of section 17.

Rules for decisions:

- Do not ask me questions. If something is unclear, pick the sensible default, leave `// TODO(decision): …`, and list it in the report.
- Never invent product behaviour or numbers. No fake testimonials, logos or statistics. Anything shown that isn't real data must be labelled **"Sample"**.
- **All people, universities and organizations are fictional.**
- The backend (`aptohire-backend/`) may not exist yet. **The app must run completely in mock mode** (`VITE_USE_MOCKS=true`) and switch to the real backend by changing one env value.
- **Exact backend object names come from `docs/CONTRACT.md`** (the user copies it here from `aptohire-backend`). If it differs from the names in this prompt, **the contract wins**; adapt only the `supabase/` implementation and mappers, never the UI.

---

## 1. Product context (organization view)

**AptoHire — "Hired for what you can do."** Students prove skills by building real projects; an AI review issues a **verified scorecard**; organizations find students **without seeing names** until the student says yes.

What a recruiter does here:

1. **Applies** on `aptohire.com`; an admin approves and the organization gets the **Verified organization** badge. Until then the portal is a **read-only preview**.
2. **Subscription** (prototype: an admin toggles it). If inactive, candidate matching and new requests are blocked behind a paywall panel.
3. **Posts a job** (F-OR-03): title, description (paste the job description), required skills with **minimum level**, location/mode, openings.
4. **JD reader** (F-OR-04): an assistant reads the pasted description and **suggests skills and levels**; the recruiter edits them.
5. **Sees ranked, anonymous candidates** (F-OR-05/06/11): an alias ("Candidate K7F2"), a university **category** (never the name), verified skills and levels, **"why they match"**, gaps, confidence, and a match explanation. **No name, email, photo, university name or file links, ever, before consent.**
6. **Requests access** (F-OR-07) to one or several candidates: a message, the data they ask for (profile, scorecard, projects, contact details), and a duration (7 / 30 / 90 days). The **student decides**.
7. **Views accepted candidates** (F-OR-08): the full details **within the granted scope and duration**, with the expiry shown. After expiry or withdrawal the data disappears.
8. **Runs a pipeline** (F-OR-09): Matched → Requested → Accepted → Interviewing → Offered/Closed (manual statuses).
9. **Sends structured feedback** (F-OR-10) to students and **browses a directory** (F-OR-12) of universities and organizations (public information only).
10. Is **notified** when a new verified candidate matches an open job (F-OR-13) and when requests are answered or about to expire.

Levels: **Emerging** (0–50, not verified) · **Competent** (51–69) · **Proficient** (70–84) · **Advanced** (85–100). Job minimums use **Competent / Proficient / Advanced** only. Results under human review (borderline 40–60) never appear to organizations until confirmed (the backend enforces this; the UI never tries to show them).

**Fairness and privacy rules this portal must respect (never relax):**

- Matching is presented as **"match with reasons and confidence", never as a prediction of hiring success.**
- The university **category** on a card is context only. **Do not offer filters or sorting by university category, college, region or any other proxy for protected attributes.** Filter and sort only by match, confidence, specific skills, study stage and project count.
- Aliases differ per job. Never build anything that tries to link candidates across jobs.
- Revealed data lives **in memory only**: never in storage, URLs, document titles, logs or error reports.

---

## 2. Where this project fits

AptoHire is **five separate web projects sharing one Supabase database**:

| Project | Host | Port (local) |
|---|---|---|
| `aptohire-root/` | `aptohire.com` | 5173 |
| `aptohire-student/` | `student.aptohire.com` | 5174 |
| `aptohire-university/` | `university.aptohire.com` | 5175 |
| **`aptohire-organization/` ← YOU ARE BUILDING THIS** | `organization.aptohire.com` | **5176** |
| `aptohire-admin/` | `admin.aptohire.com` | 5177 |

`aptohire-backend/` owns migrations, RLS, seed data and Edge Functions. **This project never edits any of that.** It consumes the database with the anon key, the recruiter's JWT, RLS, **blind views**, Realtime and the Edge Functions in section 4.

If `../aptohire-student` or `../aptohire-university` exists you may **read** it and **copy** (never import): `src/styles/tokens.css`, `src/components/proof/ProofRing.tsx`, `StatusChip.tsx`, `src/components/brand/*`, `src/components/shell/*` patterns, `src/lib/supabase/client.ts` pattern, `DataTable`. If not, build from sections 6–7.

**Hard boundaries**

- Serves **recruiters only**. A signed-in user whose role is not `organization` sees "Go to your portal" and **no organization data is fetched** (section 5).
- **Never link to or mention the admin portal.**
- **The service-role key must never exist in this repo.** Only `VITE_*` values.
- The app must **never** request `students`, `affiliations`, `match_results`, `candidate_profiles`, `submissions`, `audits`, `skill_scores`, `roster_entries` or any storage object directly. The blind views and `reveal-student` are the only paths to candidate data.

---

## 3. Tech stack and project setup

| Layer | Choice |
|---|---|
| Build | **Vite + React 18 + TypeScript (strict, no `any`; `unknown` + Zod)** |
| Routing | React Router (data router); every route `React.lazy` except shell and login |
| Data | **TanStack Query** + `@supabase/supabase-js` (used **only** in `src/lib/backend/supabase/`) |
| Forms | React Hook Form + **Zod** |
| Styling | **Tailwind CSS 3.4 (pinned)** + CSS variables (section 6) + **shadcn/ui** (Radix); install only primitives you use |
| Icons | `lucide-react`, stroke 1.75, 20 px |
| Fonts | Self-host via Fontsource: `@fontsource-variable/bricolage-grotesque`, `@fontsource-variable/instrument-sans`, `@fontsource/jetbrains-mono` (400) |
| Motion | No animation library. CSS + SVG + a tiny `useCountUp` hook |
| Head | `react-helmet-async` (`noindex` on every route) |
| Tests | Vitest + React Testing Library, Playwright + `@axe-core/playwright` |
| Lint | ESLint (typescript-eslint, react-hooks, jsx-a11y) + Prettier |

### Scripts

`dev` (port **5176**, `--strictPort`), `build`, `preview`, `typecheck`, `lint`, `test`, `e2e`, `db:types` (`supabase gen types typescript --project-id $SUPABASE_PROJECT_REF > src/lib/database.types.ts`; committed; until it exists use hand-written domain types).

### Environment (`.env.example` committed; `.env.local` ignored)

```
VITE_APP_ENV=local                  # local | demo | production
VITE_DEMO_MODE=true                 # demo switcher + prototype notice + demo tools
VITE_USE_MOCKS=true                 # true = in-memory backend, no network
VITE_SITE_URL=http://localhost:5176
VITE_ROOT_URL=http://localhost:5173
VITE_ROOT_DOMAIN=localhost
VITE_STUDENT_URL=http://localhost:5174
VITE_UNIVERSITY_URL=http://localhost:5175
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=             # public anon/publishable key ONLY
VITE_DEMO_PASSWORD=                 # demo/local only; fictional accounts
VITE_SUPPORT_EMAIL=support@aptohire.test
```

`src/config/env.ts` parses with Zod at startup and throws a readable error. **Build fails** if `VITE_APP_ENV=production` and `VITE_USE_MOCKS` or `VITE_DEMO_MODE` is true, or `VITE_DEMO_PASSWORD` is set.

`src/config/links.ts` exports `rootUrl(path)` and `portalUrl('student' | 'university', path)` (explicit env first, else `https://{portal}.${VITE_ROOT_DOMAIN}`). **No other code builds cross-site URLs. There is no admin option.**

### Folder structure

```
aptohire-organization/
├─ README.md  .env.example  index.html  package.json  vite.config.ts  tailwind.config.ts
├─ vercel.json               # rewrites + security headers (section 12)
├─ public/                   # favicon.svg, robots.txt (Disallow: /)
├─ docs/PLAN.md
├─ src/
│  ├─ main.tsx
│  ├─ app/                   # router.tsx, providers.tsx, RoleGuard.tsx, AuthGate.tsx, OrgGate.tsx, ErrorBoundary.tsx
│  ├─ config/                # env.ts, links.ts, demoAccounts.ts
│  ├─ components/
│  │  ├─ shell/              # AppShell, SideRail, TopBar, BottomTabBar, MoreSheet, AccountMenu, NotificationBell, SkipLink, PrototypeNotice, PendingBanner
│  │  ├─ candidates/         # CandidateCard, CandidateDrawer, AliasGlyph, MatchMeter, SkillBreakdown, MatchExplanation, LockRow, RequestDialog, BulkRequestBar
│  │  ├─ jobs/               # JobForm, JdReader, SkillRequirementEditor, SkillPicker, JobStatusChip, JobSummaryStrip
│  │  ├─ requests/           # RequestRow, RevealedCandidate, ScopeChips, ExpiryNotice, FeedbackDialog
│  │  ├─ pipeline/           # PipelineBoard, PipelineColumn, PipelineCard, MoveMenu
│  │  ├─ data/               # DataTable, FilterBar, StatRow, SampleDataBadge, SubscriptionPanel
│  │  ├─ proof/              # ProofRing, StatusChip
│  │  ├─ brand/              # Logo, SealMark
│  │  └─ ui/                 # Button, Field, Input, Select, Textarea, Checkbox, Switch, Tabs, Dialog, Sheet, Toast, Skeleton, EmptyState, SegmentedControl, ...
│  ├─ features/              # api hooks (use*.ts), components, schemas per area
│  │  ├─ auth/ organization/ jobs/ matching/ candidates/ requests/ reveal/ pipeline/ feedback/ directory/ notifications/ account/ config/
│  ├─ pages/                 # thin route components
│  ├─ lib/
│  │  ├─ backend/            # types.ts (Backend interface), index.ts (factory), supabase/, mock/
│  │  ├─ supabase/client.ts  # the ONLY createClient call
│  │  ├─ domain/             # types.ts, mappers.ts
│  │  ├─ schemas/            # zod for every response and form
│  │  ├─ match.ts  levels.ts  contactGuard.ts  hash.ts  format.ts  cn.ts
│  └─ styles/                # tokens.css, globals.css
├─ tests/                    # vitest
└─ e2e/                      # playwright
```

`levels.ts`: `levelFor(score)`, level order, `meetsMinimum(level, min)`. `match.ts`: `matchWords(final_score)` and `confidenceWords(confidence)` (4.2 note). Constants are fallbacks only and commented *"Mirror of backend config. Keep in sync."*

---

## 4. Backend contract (what this app consumes)

### 4.1 One interface, two implementations

Define `Backend` in `src/lib/backend/types.ts`, grouped (`auth`, `organization`, `jobs`, `skills`, `matching`, `candidates`, `requests`, `reveal`, `pipeline`, `feedback`, `directory`, `notifications`, `config`, `account`) with **Zod-validated** inputs and outputs. Two implementations: `supabase/` (real) and `mock/` (in-memory, 4.6). The factory picks by `VITE_USE_MOCKS`, provided through a `BackendProvider`. **Components never import supabase or call fetch**; they use feature hooks on TanStack Query. Query keys include the organization id. **Revealed data uses `gcTime: 0`, `staleTime: 0`, and is never persisted** (no query-persist plugin anywhere in this app).

### 4.2 Data access (RLS and views enforce; the app asks only for its own organization's data)

| Object | Recruiter can | Used for |
|---|---|---|
| `profiles` | read own (role) | role guard |
| `organization_members` | read own | which organization am I in |
| `organizations` | read **own row in full** (`id, name, kind, size, website, status, subscription_status, verified_at`) | gate states, paywall, account |
| `jobs` | read/insert/update **own organization's** rows (`title, description, required_skills [{skill_id, min_level, weight}], mode, location, openings, status`); never other organizations' jobs | job management |
| `skills` | read | skill picker and names |
| `org_candidate_cards` (view) | read, scoped to my jobs | candidates: `candidate_ref, job_id, alias, final_score, confidence, reasons, gaps, uni_category, skills [{skill_id, score, level, verified}], study_stage, project_count, request_status` (+ `structured_score, vector_score, scrubbed_summary` when the contract provides them) |
| `org_access_requests_v` (view) | read | requests/pipeline: `id, candidate_ref, alias, job_id, job_title, message, requested_scope, granted_scope, duration_days, status, created_at, responded_at, expires_at, pipeline_stage` — **no identity** |
| `access_requests.pipeline_stage` | update **only this column**, on own accepted rows | pipeline moves (no select-back; send the update without returning rows) |
| `feedback_options` | read | feedback choices (code + friendly label) |
| `feedback` | insert | structured feedback |
| `public_universities`, `public_organizations` | read | directory |
| `notifications` | read own; update `read_at` | bell and centre |
| `public_config` | read | pass mark, borderline range (display only) |

The portal **never** reads `students`, `affiliations`, `match_results`, `candidate_profiles`, `submissions`, `audits`, `skill_scores`, `roster_entries`, `org_revealed_students` (use `reveal-student` instead so every reveal is logged), `demand_signals`, or any storage object.

**Match wording** (`lib/match.ts`, thresholds `// TODO(decision)`): normalise `final_score` to 0–1 (if > 1, divide by 100). **Strong match** ≥ 0.75 · **Good match** ≥ 0.55 · **Partial match** otherwise. Confidence: ≥ 0.75 **High** · ≥ 0.5 **Medium** · else **Low**.

> **Exact names win from `docs/CONTRACT.md`.** If a view's columns differ, adapt the mapper in `lib/backend/supabase/`. If something is missing, hide that UI and leave `TODO(backend)`.

### 4.3 Edge Functions (recruiter JWT; base `${VITE_SUPABASE_URL}/functions/v1`)

Errors: `{ error: { code, message, fields? } }`. Map every code to calm copy (section 9); never show raw messages.

| Function | Request → Response |
|---|---|
| `complete-signup` | `{ portal: 'organization', application_ref?: string }` → `204`. Sets `profiles.role` once (idempotent). If the **verified** email equals an application's contact email (and `application_ref` matches when given) the backend attaches the user to that organization. `409 ROLE_ALREADY_SET` if a different role exists |
| `extract-job-skills` | `{ description }` → `{ skills: [{ skill_slug, min_level, weight }], title_suggestion?, nice_to_have?: string[], unclear?: string[] }` (JD text is untrusted; render outputs as text) |
| `match-job` | `{ job_id }` → `{ status: 'ready' \| 'running', candidate_count }`. Requires an approved organization with an **active** subscription (`402 SUBSCRIPTION_INACTIVE`). Rate-limited |
| `request-access` | `{ match_result_id (= candidate_ref), message?, requested_scope: Scope[], duration_days: 7\|30\|90 }` → `{ request_id }`. Errors: `CONFLICT` (already pending/accepted), `RATE_LIMITED` (20/day per organization), `SUBSCRIPTION_INACTIVE`, `FORBIDDEN`, `VALIDATION_FAILED` |
| `reveal-student` | `{ request_id }` → `Revealed` below. Only while the request is `accepted` and unexpired; otherwise `403 GRANT_EXPIRED`. **Every call is audit-logged by the backend**, so call it only when a recruiter opens a revealed candidate, never to render lists |

```ts
type Scope = 'profile' | 'scorecard' | 'projects' | 'contact';
type Revealed = {
  request_id: string; granted_scope: Scope[]; expires_at: string;
  profile?: { full_name: string; headline?: string; bio?: string; location?: string;
              links?: { label: string; url: string }[];
              university?: { name: string; verification: 'roster' | 'student_id' | 'unverified' };
              avatar_url?: string };                       // signed, short-lived
  scorecard?: { skills: { name: string; level: string; score?: number; verified: boolean;
                          verified_on?: string; summary?: string }[] };
  projects?: { id: string; title: string; description?: string; skills: string[];
               live_url?: string; github_url?: string;
               files?: { name: string; url: string }[]; video_url?: string }[];   // signed URLs, ~10 minutes
  contact?: { email: string };
};
```

Render **only the keys present**; if a scope is not granted, show "Not shared" for that section, never an empty fake. Parse with Zod; unknown keys are dropped.

### 4.4 Realtime

Subscribe (RLS-filtered) to `notifications` (own) and on each event invalidate the affected queries (`org_access_requests_v`, `org_candidate_cards`). The blind views are not Realtime sources, so also **poll `org_access_requests_v` every 30 s while the Requests or Pipeline page is open**, and poll `org_candidate_cards` every 3 s (max 30 s) while a matching run is `running`. Always unsubscribe on unmount.

### 4.5 Mock backend (required, in-memory, deterministic)

- State in memory and `sessionStorage` (try/catch), **except revealed data, which is never written to storage**. Latency 300–800 ms. **Reset demo data** in demo mode (Account).
- `extract-job-skills`: keyword matching against the taxonomy (e.g., "React" → React/Proficient/weight 1.0; "TypeScript" → TypeScript/Competent/0.6; "test" → Testing/Competent/0.3), a `title_suggestion`, and an `unclear` note when no seniority is stated.
- `match-job`: deterministic candidates from a hash of the job (≈ 14 for React jobs; **0 for the Data Visualization job**); aliases `Candidate K7F2`-style, **different per job**; `status: 'running'` for 1.5 s then `ready`; `402` if the organization's subscription is inactive.
- `request-access`: duplicate → `CONFLICT`; the 21st request per day → `RATE_LIMITED`; inactive subscription → `402`.
- `reveal-student`: returns fictional scoped data per the grant; expired grant → `403 GRANT_EXPIRED`.
- **Demo tools** (demo mode only, in Account; excluded from production builds): **Simulate admin approval** (pending organization), **Toggle subscription**, **Simulate student response** (pick a pending request → accept with a scope and duration / decline; pick an accepted one → withdraw), **Advance time 3 days** (to see the expiry warning and expiry). They exist because the student portal is a separate project.

### 4.6 Fictional seed data (mock mode)

Demo password for all mock accounts is `VITE_DEMO_PASSWORD` (any non-empty value in mock mode).

| Account | Organization | State |
|---|---|---|
| `recruiter@orga.demo.test` | **Northwind Labs** (mid-size corporate, approved, subscription active, **Verified organization**) | Jobs: **Frontend Developer (React)** (open, 2 openings; required React · Proficient, TypeScript · Competent) and **Frontend Intern (React)**. For Frontend Developer: ~14 ranked candidates including **K7F2** (Strong match · High confidence, **Requested · waiting for the student**), **M3Q8** (Strong), **T9R4** (Good); for Frontend Intern: one candidate **Declined** |
| `recruiter@orgb.demo.test` | **Brightlane** (start-up, approved, active, Verified) | Jobs: **Junior Full-stack Intern** (one **Accepted** request, expires in ~20 days, granted scope **profile + scorecard + projects, no contact**; this reveals the fictional student "Aarav Sharma" from Uni A), **UI Engineer (React)** (one request **Withdrawn**), **Data Visualization Analyst** (**no candidates**: shows the empty state). Also one accepted request **expiring in 2 days** and one **expired** (to show the warning and `GRANT_EXPIRED`) |
| `recruiter@orgc.demo.test` | **Orbit Analytics** (**pending** application, reference `AH-4M8Z2D`) | Read-only **sample** preview |
| `recruiter@orgd.demo.test` | **Cedar Works** (approved, **subscription inactive**) | Paywall states |

Feedback options (friendly labels): *Strong projects · Great communication · Needs more depth · Not a fit right now*. Universities and organizations for the directory: **Uni A** (Tier 2), **Uni B** (Tier 3), **Northwind Labs**, **Brightlane** (all fictional).

### 4.7 What the backend project must provide (list in your report; treat each as optional at runtime)

`org_candidate_cards` extended with `structured_score, vector_score, scrubbed_summary` (else hide the match explanation split and the proof summary) · `org_access_requests_v` including request `id`, `job_id`, `job_title`, `message`, `requested_scope`, `duration_days`, `created_at`, `responded_at` · a `my_organization_v` (or equivalent) exposing the application `reference` · `jobs` insert/update policy limited to **approved** organizations' members · readable `feedback_options` and `skills` · a server-side check that rejects contact details in request messages (the client also checks) · the exact `reveal-student` response shape · `match-job` behaviour (sync or `running`) · notification kinds in 9 · demo logins for a pending and an inactive-subscription organization (`recruiter@orgc.demo.test`, `recruiter@orgd.demo.test`) in the seed. If something is missing: hide the dependent UI, keep the rest working, and mark `TODO(backend)`.

---

## 5. Auth, sessions and gates

- **Supabase Auth, email + password.** Single client in `src/lib/supabase/client.ts`: `storageKey: 'aptohire-organization'`, `persistSession`, `autoRefreshToken`, `detectSessionInUrl`, `flowType: 'pkce'`. **No cookie shared across subdomains.**
- **Public routes:** `/login`, `/signup`, `/forgot-password`, `/reset-password`, `/auth/callback`. Everything else is behind gates, in order: **`AuthGate` → `RoleGuard` → `OrgGate`**.
- **Sign-up** (`/signup?ref=AH-4M8Z2D`): full name, **work email**, password ≥ 10 chars with a strength hint, **Application reference** (prefilled from `?ref`, optional). Helper: "Use the work email you applied with so we can link your account." → `auth.signUp` → "Check your email" with resend → `/auth/callback` runs `useBootstrap()`: no role → `complete-signup({portal:'organization', application_ref})`; then membership check. Every step is **idempotent and resumable**.
- **Role guard (F-ID-03):** after login fetch **only** the user's own `profiles` row. `organization` → continue. `student` or `university` → **"Go to your portal"** with a button to `portalUrl(role)`. Any other role (including admin) → generic "This account can't use the organization site." with **no link and no role named**. **No organization data hook may run before the guard passes** (enforce with a test).
- **`OrgGate`:** load `organization_members` + own `organizations` row.
  - **No membership:** "You're not linked to an organization yet." with **Apply for organization access** → `rootUrl('/apply/organization')` and **Sign out**. Nothing else renders.
  - **`pending`:** shell in **preview mode** (8.1).
  - **`rejected`:** "Your application wasn't approved." with support mailto. Data and writes blocked; Sign out available.
  - **`approved`:** full portal; if `subscription_status !== 'active'` show the **paywall states** (8.2).
- **Sessions:** account page offers **Sign out of all devices** (`signOut({scope:'global'})`). Sign-out clears the Query cache and Realtime channels, and **explicitly removes any revealed data from memory**.
- **Login page:** email + password, "Forgot password", link to sign-up, link to `rootUrl('/')`. In demo mode a **Demo accounts** panel with one-click buttons for the four accounts in 4.6 (`src/config/demoAccounts.ts`; `// TODO(backend)` keep in sync with the seed). Excluded from production builds.
- Multiple memberships: use the first, `// TODO(decision):` switcher later. Team roles (F-OR-14) are P2: show only the recruiter's own membership.

---

## 6. Design system (identical across all five portals)

**Concept: "Proof you can see."** Quiet, white, spacious; one thing stands out: **evidence**. Green = verified, trusted. Orange = energy, action, in progress. White is the product. For the organization portal the character is **decisive and orange-led** (portal strip `#FFEBD6 → #FFFFFF`), with **green reserved for verified things** (Verified organization, verified skills, accepted access).

### 6.1 Tokens → `src/styles/tokens.css` (mirror in `tailwind.config.ts`)

```css
:root {
  --canvas:#ffffff; --mist:#f6f8f7; --hairline:#e3e8e5; --ink:#17261e; --moss:#5b6a62;
  --verdant-50:#ecfaf2; --verdant-100:#d2f3e1; --verdant-300:#8fe0b2;
  --verdant-600:#0e9f5b; --verdant-700:#0b7f49; --verdant-800:#0a6139;
  --ember-50:#fff3e8; --ember-100:#ffe2c7; --ember-300:#ffb470; --ember-500:#ff7a1a; --ember-700:#b94a06;
  --danger-600:#c4311f; --danger-50:#fdece9;
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
  --portal-strip: linear-gradient(180deg,#ffebd6,#ffffff);   /* organization identity */
}
@media (prefers-reduced-motion: reduce){ *{animation-duration:.001ms!important;transition-duration:.001ms!important;scroll-behavior:auto!important} }
```

Tailwind names: colors `canvas, mist, hairline, ink, moss, verdant.{50,100,300,600,700,800}, ember.{50,100,300,500,700}, danger.{50,600}`; radius `control, panel, feature`; shadow `float` (shadow-1), `modal` (shadow-2); fonts `display, body, mono`. **No hard-coded hex in components.** Light theme only.

### 6.2 Rules

- **Colour:** primary button = **white on `verdant-700`** (Save job, Send, Confirm). Accent = **`ink` on `ember-500`** (never white on orange): used for **Request access** on candidate cards and **Post a job**. Never more than **one accent button per card or per dialog**. `verdant-600` / `ember-500` for graphics and large elements only; small green text `verdant-700`, small orange text `ember-700`. Colour is never the only signal.
- **Orange-led details (graphics only):** the active nav indicator bar (`ember-500`), job status dots, the "requested" ring on aliases. Green appears only for verified/accepted.
- **Gradients:** `portal-strip` only in the top bar strip and the page header of Jobs; **at most one gradient surface per viewport**; none on buttons, cards, rows or behind text smaller than 16 px.
- **Type:** Bricolage Grotesque (headings 600–700; scores 600), Instrument Sans (UI), JetBrains Mono only for references and IDs. Sentence case; no tracked all-caps labels; no eyebrow labels unless they carry information; `tabular-nums` for numbers. Lines ≤ 68 chars in prose. Left-aligned.

| Token | Size | Line-height |
|---|---|---|
| `h1` | `clamp(28px, 4vw, 36px)` | 1.2, −0.015em |
| `h2` | `clamp(22px, 2.6vw, 28px)` | 1.3 |
| `h3` | `clamp(18px, 2vw, 20px)` | 1.35 |
| `body` | **16px minimum** | 1.6 |
| `small` | **14px minimum** | 1.5 |

- **Spacing** (4 px base): 4 · 8 · 12 · 16 · 24 · 32 · 48 · 72. Content max **1200 px**, page padding 24 px desktop / 16 px mobile.
- **Radius encodes hierarchy:** `control` 10 (inputs, buttons; chips full-pill), `panel` 16 (**candidate cards**, dialogs, sheets), `feature` 28 (Jobs header panel only).
- **Flat by default** with hairline borders. Shadow only for floating things (`float`: menus, popovers, **candidate-card hover**; `modal`: dialogs, drawers). Data lives in tables and lists; **cards exist only for things you act on: candidates and pipeline cards**.
- **Motion:** the portal has **no signature animation**. The match meter fills once on first view (400 ms, disabled under reduced motion); everything else responds to actions only.
- **Icons:** Lucide. Empty states: palette circles/arcs on `mesh`; **no stock photography, no photos of people** (revealed avatars appear only on the revealed candidate page).

### 6.3 App shell

```
┌────────────┬──────────────────────────────────────────────────────┐
│ ◖ AptoHire │ [☰ at 768–1023]    🔔   Northwind Labs ✓ · R. Mehta ▾ │ ← 56px top bar, portal-strip behind
│ organization├─────────────────────────────────────────────────────┤
│ Jobs       │   Page title (h1)                       [Post a job] │
│ Candidates │   One-line description (moss)                        │
│ Requests   │   ───────────────────────────────────────────────    │
│ Pipeline   │   content                                            │
│ Directory  │                                                      │
│ Help       │                                                      │
└────────────┴──────────────────────────────────────────────────────┘
```

| Width | Behaviour |
|---|---|
| ≥ 1280 | Full 232 px rail; the candidate drawer opens beside the list |
| 1024–1279 | Rail collapses to 72 px icons with tooltips |
| 768–1023 | Rail becomes a **drawer** opened by a hamburger (☰/✕, 44×44, `aria-expanded`, focus trap, Escape closes and returns focus, closes on navigation and on resize ≥ 1024) |
| < 768 | **Bottom tab bar**: Jobs · Candidates · Requests · Pipeline · **More** (sheet: Directory, Notifications, Account & security, Help, Sign out). Sheets and drawers become full-screen |

Nav items are **exactly**: Jobs · Candidates · Requests · Pipeline · Directory (Help and Account in the rail footer / More). Requests shows an unread-count badge (unread answered/expiring notifications) with a text alternative. The account menu shows the organization name, a **Verified organization** chip when `verified_at` is set, and the subscription state. Skip-to-content link first. Minimum width **360 px**. Respect `env(safe-area-inset-*)`.

---

## 7. Signature components

### 7.1 `CandidateCard` (the portal's signature; design.md §8.4)

`r-panel`, 1 px hairline border, `float` shadow on hover/focus-within. An `<article>` with the alias as its heading.

```
┌───────────────────────────────────────────────────────────────────┐
│ ◔◑  Candidate K7F2        Tier 2 · University      Strong match  │
│     ▇▇▇▇▇▇▇▇▁▁   High confidence                                 │
│     React · Proficient   TypeScript · Competent   Testing · Comp. │
│     Verified React at Proficient and 3 reviewed projects.         │
│ 🔒  Identity hidden until the student agrees    [View] [Request access] │
└───────────────────────────────────────────────────────────────────┘
```

- **`AliasGlyph`** (48 px): generated mark from a deterministic seed (hash of the alias, `lib/hash.ts`): 3–4 overlapping circles/arcs, palette greens and oranges only, on `mist`. No faces, no initials. Same alias → same glyph. Unit-test determinism. A thin `ember-500` ring around the glyph when a request is pending.
- **Category chip:** the `uni_category` text from the view, shown as a neutral `mist` chip (context only; not filterable).
- **`MatchMeter`:** thin bar (`verdant-600` on `mist`) sized by the normalised final score, **plus words**: "Strong match" and "High confidence". Meter has `role="meter"` with `aria-valuenow` and a text label.
- **Skill chips:** up to three, **verified** skills with levels, highlighting those that **meet the job's minimum** (check icon) versus those below it (no icon, `mist`). Use words, never colour alone.
- **One reason line:** `reasons[0]`, rendered as text. Never render any other free text on the card.
- **`LockRow`:** lock icon + "Identity hidden until the student agrees".
- **Actions:** **View** (secondary) opens the drawer; **Request access** (accent) opens the request dialog. State variants driven by `request_status`: *Requested* (disabled button "Requested · waiting", `ember-50` chip), *Accepted* ("View profile" → the revealed page), *Declined* (muted chip, no actions), *Withdrawn/Expired* (muted chip, **Request again** allowed only if the contract allows a new request). A checkbox ("Select Candidate K7F2") enables bulk requests.

### 7.2 `CandidateDrawer` (F-OR-06, F-OR-11)

A side sheet (full-screen below 768; deep link `/candidates/:candidateRef`; `modal` shadow; focus trapped). Contents, **blind by construction**:

- Header: alias glyph, alias, category chip, study stage chip ("Year 2", "Alumni"…), **project count** ("3 reviewed projects").
- **Why they match:** `reasons` (≤ 3) as a list. **Gaps:** `gaps` (≤ 2) with a neutral icon ("No verified Testing yet").
- **Skill breakdown:** one row per **required skill**: skill name, **required level** vs **candidate level** (word + bar vs the minimum tick), score, verified mark. Skills the candidate has beyond the requirements are listed below under "Other verified skills".
- **Match explanation (F-OR-11):** plain text with two small bars: "Skills match 82%" and "Similar work 64%", and "The final match is 70% skills and 30% similar work." (Hide this block if `structured_score`/`vector_score` aren't provided.) Always followed by: "This is a match with reasons and confidence. It doesn't predict hiring success."
- **Proof summary:** `scrubbed_summary` as plain text (hide if absent). **No raw files, no links, no evidence URLs.**
- **Lock panel:** "Identity hidden until the student agrees. You'll see their name, university and the data they choose to share after they accept."
- Footer actions: **Request access** (accent) and **Close**.

### 7.3 `StatusChip` and domain chips

| Chip | Look |
|---|---|
| Verified organization | `verdant-50` fill, small seal |
| Verified (skill) | `verdant-100` / `verdant-800`, seal icon |
| Pending / Requested | `ember-50` / `ember-700` |
| Accepted | `verdant-100` / `verdant-800`, check |
| Declined · Withdrawn · Expired · Closed | `mist` / `moss` |
| Expiring soon (≤ 3 days) | `ember-100` / `ember-700`, clock icon |
| Job: Open · Paused · Closed | Open = `verdant-50`/`verdant-800`; Paused = `ember-50`/`ember-700`; Closed = `mist`/`moss` |
| Pipeline: Interviewing · Offered | `mist` / `ink` with a distinct icon each; Closed = `mist` / `moss` |
| Subscription: Active / Inactive | Active = `verdant-50`; Inactive = `ember-50` |

### 7.4 `ProofRing`

Used only on the **revealed candidate page** (56 px per verified skill). Props: `score`, `skill`, `state: 'verified'|'loading'`, `size: 56|96`, `level`, `animate?`. Track `hairline`; SVG arc with round caps, gradient `#FF7A1A → #FFB470 → #8FE0B2 → #0E9F5B`; seal on verified; `role="img"` with `aria-label="React, score 72 of 100, Proficient, verified"`. Reveal 900 ms once (reduced motion → final). Unit-test the aria-label builder.

### 7.5 Other UI

Buttons (primary / accent / secondary / ghost / danger; height 44 mobile, 40 desktop; label 15/600; loading keeps label and fixed width), Field (label **always above**, helper 14 px, error with icon, `aria-describedby`), Tabs (underline, 2 px `verdant-700`), `SegmentedControl` (duration 7/30/90), Dialog and Sheet (focus trap, Escape, restore focus), Toast (bottom-centre, 4 s, reuses the action's verb), Skeleton (matches layout; **candidate-card skeletons**), EmptyState (palette circles on `mesh`, one sentence, one button), `StatRow`, `SubscriptionPanel`, `SampleDataBadge` (quiet pill "Sample data"), `DataTable` (sticky header, hairlines, scrolls inside its container, becomes a labelled list below 640 px).

---

## 8. Screens and flows

Routes: `/login` · `/signup` · `/forgot-password` · `/reset-password` · `/auth/callback` · `/` Jobs · `/jobs/new` · `/jobs/:jobId` · `/jobs/:jobId/edit` · `/candidates` (`?job=`) · `/candidates/:candidateRef` (opens the drawer over the list) · `/requests` (tabs via `?tab=`) · `/requests/:requestId` · `/pipeline` · `/directory` · `/notifications` · `/account` · `/help` · `*` 404. Every page: `h1`, document title (**never contains a candidate's name or alias**), loading skeleton, empty state, error state with **Retry**, `noindex`.

### 8.1 Pending application: read-only preview (F-OR-01)

While `status='pending'`: a persistent, non-dismissible **`PendingBanner`** ("Your application is under review. Reference **AH-4M8Z2D**. You'll get an email when it's approved.", reference in mono with copy). **Jobs, Candidates, Requests, Pipeline and Directory** show **labelled sample content** (`SampleDataBadge` + "This is sample data so you can see what you'll get after approval") from deterministic fixtures, **all write actions disabled** with tooltips ("Available after approval"). Never present sample data as the organization's own, and never fetch candidate views. On approval (Realtime notification or next load) show a one-time toast "Your organization is approved." In mock demo mode this is triggered by **Simulate admin approval**.

### 8.2 Subscription paywall (F-OR-02)

Approved but `subscription_status !== 'active'` (prototype: an admin toggles it): **Jobs and Directory work normally** (recruiters can draft and edit jobs). **Candidates** shows a full-page `SubscriptionPanel` instead of any candidate data (**do not call the candidate view**): "Your subscription isn't active. Candidate matching is switched off until it is." with **Contact AptoHire** (mailto) and, in non-production, "Prototype: an admin turns this on." In Jobs, **Find candidates** and **Refresh matches** are disabled with a tooltip. `Request access` is unavailable. **Requests and Pipeline stay available for existing grants** (view, reveal until expiry, feedback, pipeline moves); `// TODO(decision):` revisit with the backend. Also handle `402 SUBSCRIPTION_INACTIVE` from `match-job` and `request-access` by showing the same panel/message.

### 8.3 Jobs `/` and job editor (F-OR-03, F-OR-04)

**Jobs list:** header (with `portal-strip`) and a primary-accent **Post a job** button. A short "Needs your attention" list (plain rows, only when non-empty): accepted requests not yet viewed, accesses **expiring within 3 days**, **new matches** (from notifications). Then a table: **Job · Status · Openings · Mode/Location · Required skills (chips with levels) · Candidates · Requests**. Row actions: **View candidates**, **Edit**, **Pause/Close** (confirm; closing keeps history). Empty: "Post your first job to see verified candidates." → accent **Post a job**.

**Job detail `/jobs/:jobId`:** summary strip (title · status · openings · mode/location), required skills with levels, counts (candidates, pending, accepted), buttons **View candidates**, **Edit job**, **Refresh matches**.

**Job editor** (`/jobs/new`, `/jobs/:jobId/edit`), one calm page in sections (not a wizard):

1. **Role:** title (3–120), openings (1–99), mode (On-site / Hybrid / Remote), location (≤ 80; required unless Remote), status (Open / Paused / Closed on edit).
2. **Description:** a large textarea (≤ 8,000 characters) with helper "Paste your job description. We can read the skills for you." and a quiet privacy note "Don't paste personal data."
3. **JD reader:** button **Read skills from this description** → `extract-job-skills`. Loading shows a skeleton table. Result panel: a table of **suggested skills** (checkbox, skill, **minimum level** select, **importance** select) plus "Worth clarifying" list from `unclear` and `nice_to_have` as plain text; **Add selected skills** merges them into the requirements (dedupe by skill). The recruiter can always edit everything. Failure copy: "We couldn't read the description. Add the skills yourself." (never blocks saving). If no known skills found: "We couldn't find known skills in this text. Add them yourself." Treat all outputs as untrusted text.
4. **Required skills (`SkillRequirementEditor`):** 1–8 rows: `SkillPicker` (searchable, grouped by family), **minimum level** (Competent / Proficient / Advanced), **importance** (**Must-have** = 1.0 · **Important** = 0.6 · **Nice-to-have** = 0.3; `// TODO(decision)` weights, in `lib/levels.ts`), remove button. Store as `[{skill_id, min_level, weight}]` (map `skill_slug` → `skill_id` via the taxonomy). Row-level errors.
5. **Save:** **Save job** (primary) → toast "Job saved." → a dialog **"Find candidates now?"** (**Find candidates** / **Later**) → `match-job`. After later edits to skills, show "Matches are out of date. Refresh matches." Disable **Refresh matches** for 60 s after a click (rate limit friendliness).

States: validation messages in plain words; unsaved-changes guard on navigation; autosave is **not** used (explicit save). In pending/inactive states follow 8.1/8.2.

### 8.4 Candidates `/candidates` (F-OR-05, F-OR-06, F-OR-11)

- **Job selector** (select; default the most recent open job; URL `?job=`). **Job summary strip** per design: "Frontend Developer (React) · Open · 2 openings · **Edit job**", then "Required: React · Proficient   TypeScript · Competent".
- **Matching status:** if `match-job` is `running` (or just triggered) show a calm banner "Finding candidates… This can take a moment." and `CandidateCard` skeletons; poll per 4.4.
- **Ranked list** of `CandidateCard`s (best match first). **Filters and sort (only these):** minimum match (Any / Good+ / Strong), **has a specific required skill at or above the minimum**, study stage (Year 1–5, Alumni, Masters; as provided), **minimum number of reviewed projects**, **confidence** (High only). Sort: Best match (default), Confidence, Most projects. Result count announced (`aria-live`). **No filter or sort by university category.** 20 per page ("Show more").
- **Bulk request (F-OR-07):** select up to **10** cards → a sticky **`BulkRequestBar`** ("3 selected · **Request access**") → the request dialog applies one message/scope/duration to all; the client sends them **sequentially** and reports per candidate ("Requested 2. Couldn't request 1: daily limit reached.").
- **Request dialog (`RequestDialog`):** title "Request access to Candidate K7F2" (or "…to 3 candidates"). Fields: **Message** (≤ 500, optional; helper "Tell the student about the role. Don't include contact details. The student decides what to share."), **What do you want to see?** checkboxes with plain descriptions: *Profile: name, photo, headline, bio and university* · *Scorecard: verified skills, levels and evidence summary* · *Projects: project descriptions, files and videos* · *Contact details: email address* (default: Profile + Scorecard; at least one required), **How long?** `SegmentedControl` **7 / 30 / 90 days** (default 30). Summary line: "Candidate K7F2 will be asked to share profile and scorecard for 30 days." Buttons **Request access** (accent) / **Cancel**.
  - **Contact-detail guard (`lib/contactGuard.ts`):** reject a message that looks like it contains an email, a phone number (7+ digits with separators), a URL, or a social handle: "Your message looks like it contains contact details. Remove them. The student decides what to share." (Unit-test with positive and negative samples. The backend also checks.)
  - Success toast "Access requested." The card changes to **Requested · waiting**. Errors: `CONFLICT` "You've already requested access for this candidate." · `RATE_LIMITED` "You've reached today's limit for requests. Try again tomorrow." · `SUBSCRIPTION_INACTIVE` paywall message.
- **Empty (no matches):** "No verified candidates match yet. Try lowering a required level or adding related skills." → **Edit job**.
- **Drawer:** 7.2. Closing returns focus to the card that opened it.

### 8.5 Requests `/requests` and revealed candidate `/requests/:requestId` (F-OR-07, F-OR-08, F-OR-10)

**List:** tabs **Pending · Accepted · Declined · Closed** (Closed = withdrawn + expired). Table/list: **Candidate (alias glyph + alias) · Job · Requested data (scope chips) · Duration · Sent · Status**. Accepted rows show "Access until 12 Nov" with **days left**; ≤ 3 days → `Expiring soon` chip and an `ember-700` note. Action on Accepted: **View candidate**. Pending rows: "Waiting for the student". Declined rows: "The student declined." (no reason is shown). Empty per tab (one sentence).

**Revealed candidate page `/requests/:requestId`:**

- If the request is **not accepted or has expired:** show a status page (Pending → "Waiting for the student", Declined → "The student declined this request.", Withdrawn → "The student withdrew access. This information is no longer available.", Expired → "Access ended on 12 Nov. This information is no longer available.") with **no data and no reveal call**.
- If **accepted and unexpired:** call `reveal-student({request_id})` **once on mount** (and again only on **Refresh links** or when signed media URLs fail). Render **only the granted scopes**:
  - **Header:** avatar (only if provided), full name, headline, **scope chips** ("Shared: Profile · Scorecard · Projects"), **`ExpiryNotice`** ("Access until 12 Nov · 20 days left. After that, this information disappears from your account.").
  - **Profile:** bio, location, links (`rel="noopener noreferrer"`), **current university with the verification chip** (Verified university / dashed Unverified).
  - **Scorecard:** verified skills each with a 56 px `ProofRing`, level chip, score if provided, summary text. Plain note "Scores come from reviewed projects with evidence."
  - **Projects:** title, description, skills, links, **files and video as buttons** that open signed URLs (new tab, `noopener noreferrer`); a note "Links expire after about 10 minutes. Refresh links to get new ones." with **Refresh links**.
  - **Contact:** email as a `mailto:` link with a **Copy** button. Sections whose scope wasn't granted show **"Not shared"** with a neutral icon.
  - **Resume view (print):** a **Print / Save as PDF** button using the browser print dialog and a clean print stylesheet (single column, no nav). Do not generate PDFs in the app.
  - **Pipeline:** a **Move to…** menu (Interviewing / Offered / Closed / Back to Accepted) with "These statuses are your own notes. The student isn't notified."
  - **Feedback (F-OR-10):** **Give feedback** → dialog with `feedback_options` checkboxes (friendly labels) and an optional note (≤ 280). Helper: "The student sees this. Be specific about the work. Don't include personal details." Insert via RLS; toast "Feedback sent." One submission per request (show "Feedback sent" afterwards).
- **`GRANT_EXPIRED` (403)** at any moment (including when media refresh fails): immediately **clear the data from memory**, show the "Access ended" state, invalidate queries.
- **Privacy rules for this page:** `gcTime: 0` queries; **no persistence anywhere**; document title stays "Candidate"; **no name in the URL**; never log the payload; `Cache-Control` is not relied on; navigating away discards the data.

### 8.6 Pipeline `/pipeline` (F-OR-09)

Board filtered by job (select; "All jobs"). Columns: **Matched → Requested → Accepted → Interviewing → Offered / Closed**.

- **Matched:** shows the **count** and the top 5 candidates for the job (alias, match words) with "View all in Candidates". (Matched = candidates with no request.)
- **Requested:** pending requests. **Accepted:** accepted requests without a stage (with expiry). **Interviewing / Offered / Closed:** from `pipeline_stage`.
- **Pipeline cards** show alias glyph + alias, job (when "All jobs"), status or expiry chip. **Names are not shown on the board**; they appear only on the revealed page (so every reveal is logged). `// TODO(decision):`.
- **Moves:** each card has a keyboard-accessible **Move to…** menu (no drag-and-drop in this phase). Allowed manual moves: Accepted ↔ Interviewing ↔ Offered ↔ Closed. Moves from Matched/Requested are not manual ("Request access to move this candidate"). Optimistic update with rollback and a toast "Moved to Interviewing."
- **Mobile:** columns become a `SegmentedControl`/tabs with one stage visible at a time.
- Copy under the board: "Statuses after Accepted are your own notes. The student isn't notified."

### 8.7 Directory `/directory` (F-OR-12)

Tabs **Universities** · **Organizations**. Universities (`public_universities`): name, tier, kind, city, state. Organizations (`public_organizations`): name, kind, size, **Verified organization** chip, website link. Search and filter by kind. **No contacts, no jobs of other organizations, no student or candidate information.** Empty: "Nothing here yet."

### 8.8 Notifications, account, help, system screens

- **Notifications:** bell with unread count (`aria-label="Notifications, 3 unread"`) opening a popover (latest 8) with **Mark all as read** and **View all** (`/notifications`, Today / Earlier). Kinds and deep links: `access_accepted` → Requests (Accepted) · `access_declined` → Requests (Declined) · `access_withdrawn` → Requests (Closed) · `access_expiring` → Requests (Accepted) · `new_match` ("A new verified candidate matches Frontend Developer (React).", **no identity**) → Candidates `?job=` · `application_decided` → Jobs · `subscription_changed` → Jobs. Realtime inserts show a toast. Empty: "You're all caught up."
- **Account `/account`:** organization details (name, kind, size, website; read-only in this phase), status chips (Approved, **Verified organization**), **subscription state** with "Contact AptoHire to change it", your membership (role), **Sign out of all devices**, and (demo mode only) **Demo tools** (4.5) and **Reset demo data**.
- **Help `/help`:** how matching works in plain words (verified skills, levels, reasons, confidence; not a prediction), what "identity hidden" means, how requests and consent work, how long access lasts, feedback etiquette, contact support (mailto), links to `rootUrl('/privacy')`, `rootUrl('/terms')`, `rootUrl('/ai-and-fairness')`.
- **System screens:** wrong-portal (5), no-membership and rejected (5), 404 ("That page doesn't exist." + **Go home**), `ErrorBoundary` (**Reload page**; never a stack trace or payload), offline banner, and the **PrototypeNotice** (slim, dismissible per session) when `VITE_APP_ENV !== 'production'`: "Prototype. All people, universities and organizations shown are fictional."

---

## 9. Copy and voice

Plain, warm, specific. Sentence case. Active verbs. **No hype words**, no "AI-powered" headlines, no predictions. Say what happened, not how the system works. Errors say what happened and the fix and **don't apologize**. Vocabulary (use consistently): **Job · Candidate · Match · Request access · Accepted · Declined · Withdrawn · Expired · Pipeline · Feedback.** Never say "applicant", "hire probability", "score of fit" or "AI picked". A button's verb is reused in its toast.

| Moment | Copy |
|---|---|
| Card lock | Identity hidden until the student agrees |
| Match words | Strong match · High confidence |
| No candidates | No verified candidates match yet. Try lowering a required level or adding related skills. |
| No jobs | Post your first job to see verified candidates. |
| Request sent | Access requested. |
| Waiting | Waiting for the student |
| Accepted note | Access until 12 Nov. After that, this information disappears from your account. |
| Declined | The student declined this request. |
| Withdrawn | The student withdrew access. This information is no longer available. |
| Expired | Access ended on 12 Nov. This information is no longer available. |
| Not shared | Not shared |
| Pipeline note | Statuses after Accepted are your own notes. The student isn't notified. |
| Match disclaimer | This is a match with reasons and confidence. It doesn't predict hiring success. |
| Contact guard | Your message looks like it contains contact details. Remove them. The student decides what to share. |
| Pending application | Your application is under review. You'll get an email when it's approved. |
| Paywall | Your subscription isn't active. Candidate matching is switched off until it is. |
| Rate limited | Too many attempts. Try again in a few minutes. |
| Network error | We couldn't reach AptoHire. Check your connection and try again. |
| Session expired | Your session ended. Sign in again to continue. |

Error-code mapping: `VALIDATION_FAILED` map `fields` · `CONFLICT` "You've already requested access for this candidate." · `RATE_LIMITED` per context (request limit vs generic) · `SUBSCRIPTION_INACTIVE` paywall copy · `GRANT_EXPIRED` the Expired/Withdrawn state · `FORBIDDEN` "You don't have access to that." · `ROLE_ALREADY_SET` "This email is already used for another AptoHire site. Sign in there." (link via `portalUrl` only for student/university) · unknown → network copy. All strings live in `src/content/*.ts` or per-feature `copy.ts`.

---

## 10. Quality bars

### 10.1 Responsiveness (built in, not deferred)

Breakpoints: **Mobile < 768 (base) · Tablet 768–1023 · Desktop ≥ 1024**. Tailwind `md` 768, `lg` 1024, `xl` 1280.

- `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">`; safe-area insets on fixed bars.
- **No horizontal scroll from 360 px up.** Fix root causes; never `overflow-x:hidden` on `body` as a patch. Tables scroll inside their own container at ≥ 640 px and become labelled lists below.
- Multi-column layouts collapse below 768; flex rows wrap/stack; mobile section padding ≥ 40 px vertical; nothing touches the screen edge. Candidate cards stack their actions full-width on mobile.
- Body ≥ 16 px, small ≥ 14 px, line-height ≥ 1.4; headings use the fluid tokens.
- **Touch targets ≥ 44×44 px** (buttons, icon buttons, tab-bar items, card checkboxes, chips that act, menu items).
- **Forms:** inputs 100% width on mobile, **font-size ≥ 16 px**, labels above inputs, form buttons full-width on mobile, correct `type`/`inputMode`/`autocomplete`.
- The job editor, request dialog and pipeline must be fully usable on a phone.
- Test at **360, 375, 390, 768, 1024, 1440**.

### 10.2 Accessibility (WCAG 2.2 AA)

Landmarks, one `h1` per page, skip link, **visible 2 px `verdant-700` focus outline with 2 px offset plus a 4 px white halo**, all flows keyboard-operable, focus trapped/restored in dialogs/drawers/menus (closing the drawer returns focus to the opening card), `aria-live="polite"` for matching progress, request results, filter counts and toasts, errors announced and focus moved to the first invalid field, **cards use real headings and labelled controls** ("Select Candidate K7F2", "Request access to Candidate K7F2"), `role="meter"` on match meters with text, colour never the only signal (meets/doesn't meet minimum, request states), pipeline moves available by keyboard menu, reduced motion respected. `@axe-core/playwright` zero serious/critical on every route.

### 10.3 Performance

Mobile Lighthouse ≥ 90 (Performance), ≥ 95 (Accessibility, Best Practices). Route-level code splitting; initial JS for login and Jobs ≤ **200 KB gzipped**. Server-side pagination (20 candidates per page) in the real implementation. Debounce filters (300 ms). Skeletons match final layouts.

---

## 11. Security and privacy rules (never break)

1. **No secrets in repo or bundle.** Only `VITE_*`. Never the service-role/secret key. `.env*` git-ignored except `.env.example`. A static script fails the build on `service_role`, `sb_secret`, or hard-coded hex colours in components.
2. **Blind by construction.** The only candidate data sources are `org_candidate_cards` and `reveal-student`. The app contains **no code that requests identity tables or storage**. **A DOM and network test** (section 13) renders every candidate surface (list, drawer, bulk bar, pipeline, tooltips, aria-labels, alt text, document title) with the mock store containing real-looking names, emails, university names, photo URLs and file URLs, and asserts **none of them appear before acceptance**, and that the mock `reveal-student` is **not called** while rendering lists.
3. **Revealed data is ephemeral.** `gcTime: 0`, never persisted to `localStorage`, `sessionStorage`, IndexedDB, cookies, URLs, titles, logs or error reports; cleared on `GRANT_EXPIRED`, navigation away and sign-out. A test spies on storage during a reveal and asserts nothing sensitive is written.
4. **Never trust the client for role, ownership, scope or entitlement.** The UI hides things for convenience; **RLS, views and Edge Functions decide**. Never solve a permission error by moving a check into the frontend or widening a query.
5. **No PII in logs, analytics, error messages, URLs or storage.** Allowed storage: the Supabase session (`aptohire-organization`), notice dismissals, mock state (mock mode only, never revealed data).
6. **Render all API strings as text** (React escaping); no `dangerouslySetInnerHTML`. Treat reasons, gaps, summaries, JD-reader output, student bios, project text and notification text as untrusted.
7. **Signed URLs** (avatar, files, video) are short-lived, opened with `rel="noopener noreferrer"`, never stored, never logged, and refreshed only on demand.
8. **No export of candidate data in bulk.** No CSV export of candidates or revealed profiles in this phase.
9. **Fairness guard rails:** no filters/sorts by university category, college, region, name or gender; no cross-job linking; match presented as reasons and confidence.
10. **Demo-only code** (demo switcher, mocks, reset, demo tools, prototype notice) is excluded or disabled in production.
11. **External links** use `rel="noopener noreferrer"`; cross-site links built only via `links.ts`.
12. Pin exact dependency versions; run `npm audit` and report results.

---

## 12. Deployment (its own Vercel project)

`vercel.json`: rewrite everything not a file to `/index.html`. Headers on all routes: `Content-Security-Policy` (`default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob: https://*.supabase.co; media-src 'self' blob: https://*.supabase.co; font-src 'self'; connect-src 'self' https://*.supabase.co wss://*.supabase.co; frame-ancestors 'none'; base-uri 'self'; form-action 'self'; object-src 'none'; worker-src 'self' blob:`), `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy: camera=(), microphone=(), geolocation=()`, `X-Frame-Options: DENY`, `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`, `Cross-Origin-Opener-Policy: same-origin`, `X-Robots-Tag: noindex`. Cache hashed assets `immutable`; `index.html` `no-cache`. For `/requests/*` also send `Cache-Control: no-store`.

README must explain: deploy as **its own Vercel project** with domain `organization.aptohire.com`; set the `VITE_*` variables; in Supabase **Authentication → URL Configuration** add `https://organization.aptohire.com/**` (and `http://localhost:5176/**`); the other four portals are separate projects; Vercel's free Hobby plan is for non-commercial demos.

---

## 13. Testing

| Level | Cover |
|---|---|
| Unit (Vitest) | `match.ts` (words and thresholds at 0.54/0.55/0.74/0.75, score > 1 normalisation, confidence words), `levels.ts` (`meetsMinimum`, weights mapping), `contactGuard.ts` (emails, phones, URLs, handles caught; ordinary text allowed), `hash.ts` / `AliasGlyph` determinism, skill-requirement validation (1–8 rows, no duplicates), request-dialog validation (≥ 1 scope, duration), Zod schemas (valid and invalid, including `Revealed` with missing scopes), mock `match-job` (different aliases per job, 0 for Data Visualization), mock `reveal-student` (scope slicing, `GRANT_EXPIRED`), `links.ts`, `env.ts` |
| Component (RTL) | RoleGuard (organization ok; student/university → "Go to your portal"; admin → generic without link; **no data hooks before the guard passes**), OrgGate states (no membership, pending preview with disabled writes, rejected, approved, inactive subscription → paywall without calling the candidate view), job editor (JD reader success/failure/no-skills, merge and dedupe, importance mapping, unsaved-changes guard), candidate card states (default, requested, accepted, declined, withdrawn), drawer (focus trap and return, no identity fields, hides missing blocks), bulk request (partial failure reporting), request dialog errors, requests tabs and expiry warning, **revealed page** (renders only granted scopes, "Not shared", `GRANT_EXPIRED` clears data, refresh links), feedback dialog, pipeline move menu (allowed/blocked moves, optimistic rollback), notifications deep links |
| **Privacy tests** | (a) **Blind DOM test** per section 11.2; (b) **Reveal-only-on-demand test**: list pages never call `reveal-student`; (c) **Ephemeral test**: storage spy shows no revealed data written, queries use `gcTime: 0`, sign-out and `GRANT_EXPIRED` clear data; (d) **No-proxy-filter test**: the candidates filter bar offers no university/region/name filter and the sort options are exactly the three listed |
| E2E (Playwright, mock mode) | Northwind recruiter signs in via the demo panel → posts a job (paste JD → read skills → edit → save) → finds candidates → sees ranked anonymous cards → opens the drawer → requests access to two candidates (message with a phone number is blocked, then fixed) → Demo tools: **Simulate student response → accept** → Requests shows Accepted → opens the revealed page (scoped data, expiry) → moves to Interviewing → gives feedback → **Advance time** → Expiring soon → expired state with no data. Brightlane: accepted reveal with contact **Not shared**; Data Visualization empty state → Edit job. Orbit (pending) preview with disabled writes → **Simulate admin approval**. Cedar (inactive) paywall. Wrong-role screens. Axe on every route; **no horizontal overflow at 360, 390, 768, 1024, 1440**; keyboard-only runs of the job editor, the request dialog and the pipeline; reduced-motion run |
| Static | `typecheck`, `lint`, secret/hex grep over `src/` and `dist/` |

---

## 14. README

Beginner-friendly (short sentences, numbered steps): what this project is; how it relates to the other four portals and the shared database; **how blind matching works in one paragraph**; prerequisites (Node LTS, Git); install; copy `.env.example`; **run in mock mode**; connect to a real Supabase project (URL + anon key, set `VITE_USE_MOCKS=false`, copy `docs/CONTRACT.md` from `aptohire-backend`, run `npm run db:types`); demo accounts (emails only: the four in 4.6) and the **demo tools** and what each does; commands table; folder tour; deployment to Vercel; how to edit copy and tokens; common problems (white page → env typo; "permission denied" → wrong role; paywall → subscription inactive; Supabase project paused → Restore); glossary (alias, blind view, scope, grant, expiry, pipeline, RLS, mock mode). State clearly: **this app never needs the service-role key and never reads identity tables.**

---

## 15. Build order (finish each phase, run the checks, then continue)

1. **Foundations:** Vite/TS/Tailwind, tokens, fonts, `env.ts`, `links.ts`, router, providers, `Backend` interface, **mock backend skeleton**, Supabase client module, lint/test/e2e setup.
2. **Design primitives and shell:** Button, Field family, Dialog/Sheet/Toast/Tabs/SegmentedControl/Skeleton/EmptyState, chips, Logo/SealMark, `AppShell` (rail, top bar, drawer, bottom tab bar, More sheet), PrototypeNotice, `PendingBanner`, `SubscriptionPanel`.
3. **Auth and gates:** login (+ demo panel), sign-up with reference, callback, `useBootstrap`, `AuthGate` / `RoleGuard` / `OrgGate` (no membership, pending, rejected, approved, inactive), password reset, account menu, tests.
4. **Jobs:** list, detail, editor, `SkillPicker`, `SkillRequirementEditor`, **JD reader**, validation, unsaved-changes guard.
5. **Matching and candidates:** `match-job` flow with polling, `CandidateCard`, `AliasGlyph`, `MatchMeter`, filters/sort, `CandidateDrawer`, match explanation, **blind DOM test**.
6. **Requests:** `RequestDialog` with contact guard, bulk requests, requests list and tabs, expiry notices.
7. **Reveal:** revealed candidate page (scoped rendering, refresh links, `GRANT_EXPIRED`), ephemeral-data tests, print view, feedback dialog.
8. **Pipeline:** board, move menus, mobile stage tabs.
9. **Directory, notifications (Realtime + polling), account, help, system screens, demo tools.**
10. **Hardening:** real `supabase/` implementation of every `Backend` method (adapting mappers to `docs/CONTRACT.md`), axe, Lighthouse, overflow checks, keyboard and reduced-motion passes, all privacy tests, secret/hex greps, `npm audit`, README.

---

## 16. Acceptance checklist

- [ ] `typecheck`, `lint`, `test`, `build`, `e2e` pass; no `any`; no leftover `console.log`
- [ ] Entire app works with `VITE_USE_MOCKS=true` and no network; switching to the real backend needs only env changes plus the items in 4.7
- [ ] Gates work in order: wrong roles blocked with no data fetched; admin never linked or named; no-membership, pending (sample-labelled, writes disabled), rejected and inactive-subscription states each designed
- [ ] **Blind guarantee:** no name, email, photo, university name or file URL appears anywhere in candidate surfaces before acceptance (DOM and network test); lists never call `reveal-student`
- [ ] Revealed page renders only granted scopes ("Not shared" otherwise), shows expiry, clears data on `GRANT_EXPIRED`, never persists anything; titles and URLs contain no names
- [ ] Job editor: JD reader works and fails gracefully; required skills 1–8 with level and importance; explicit save; "Find candidates now?" flow; matches-out-of-date notice
- [ ] Candidate cards match design.md §8.4: alias glyph, category chip, match meter with words, up to three verified skill chips (meets-minimum marked with icon and word), one reason line, lock row, View and Request access; all request states handled
- [ ] Drawer: reasons, gaps, skill breakdown vs required levels, match explanation (when data exists), disclaimer, proof summary (when data exists), no files or links
- [ ] **No filter or sort by university category or other proxies**; filters/sorts are exactly those in 8.4
- [ ] Request dialog: scope checkboxes with plain descriptions, 7/30/90 days, contact-detail guard, sequential bulk requests with partial-failure reporting, error codes mapped
- [ ] Pipeline: Matched → Requested → Accepted → Interviewing → Offered/Closed; keyboard-accessible moves; no names on the board; mobile stage tabs
- [ ] Feedback uses `feedback_options` and a 280-character note; directory shows public information only
- [ ] Match wording and thresholds live in `lib/match.ts`; match is never described as a prediction
- [ ] At most one gradient surface per viewport; none on buttons/cards/rows; sentence case; no tracked all-caps labels; contrast rules respected (white on `verdant-700`, `ink` on `ember-500`); at most one accent button per card/dialog
- [ ] Bottom tab bar < 768, drawer 768–1023, icon rail 1024–1279, full rail ≥ 1280; touch targets ≥ 44 px; inputs ≥ 16 px; no horizontal scroll at 360–1440; drawers full-screen on mobile
- [ ] Axe: zero serious/critical on every route; Lighthouse targets met; initial JS ≤ 200 KB gzipped
- [ ] No secrets or PII in repo/bundle/logs/URLs/storage; demo code excluded from production builds; production build fails if mocks/demo are on
- [ ] README and `docs/PLAN.md` complete; every `TODO(decision)` and `TODO(backend)` listed in the report

---

## 17. Final report (output exactly this at the end)

### ✅ Organization portal build report

**Stack used:** [versions]

**Routes built:** [list]

**Backend interface:** [methods implemented in `mock/` and in `supabase/`; any method still mock-only]

**Backend items required (4.7) and their runtime fallbacks:** [list; mark which are blocking]

**Decisions made (`TODO(decision)` / `TODO(backend)`):** [each item and the default chosen]

**Quality results:**
- Lighthouse (mobile): Performance [x] · Accessibility [x] · Best Practices [x]
- Initial JS (gzipped): login [x KB] · Jobs [x KB]
- Axe violations (serious/critical): [0 per route]
- Overflow check at 360 / 375 / 390 / 768 / 1024 / 1440: [pass/fail]
- Privacy tests: blind DOM [pass/fail] · reveal-only-on-demand [pass/fail] · ephemeral data [pass/fail] · no proxy filters [pass/fail]

**Navigation:** [bottom tab bar, drawer and rail behaviours verified]

**Files created:** [grouped list]

**What to check manually:**
- Chrome DevTools → Device Toolbar (Ctrl+Shift+M) at 375, 390, 768, 1024 and 1440 px
- Sign in as each of the four demo accounts; run the Northwind flow with the keyboard only
- On the Candidates page, open DevTools → Elements and search for any name, email or university; confirm none appears before acceptance
- Request access with a phone number in the message and confirm it is blocked
- Use **Simulate student response → accept**, open the revealed page, then **Advance time 3 days** and confirm the data disappears when access expires
- Turn on "reduce motion" in your OS: the match meter and Proof Ring should show final states instantly
