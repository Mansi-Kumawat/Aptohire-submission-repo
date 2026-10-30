# PROMPT 3 of 5 — `university.aptohire.com` (University portal)

> Paste everything below this line into your AI coding tool inside an **empty folder** named `aptohire-university/`.
> This project is **independent**. It does not import from, or share code with, the other portals. Where this prompt says "copy", copy the file; never create a cross-project import.

---

## 0. Your role and how to work

You are a senior frontend engineer and product designer. Build the **University portal of AptoHire** (`university.aptohire.com`) for placement cells (TPOs) and university staff, as a complete, production-quality Vite + React + TypeScript app that talks to the **shared Supabase backend**.

Work in this order:

1. **Plan (short).** Write `docs/PLAN.md` (max 1 page): route list, feature modules, the `Backend` interface (section 4.1), what you will mock. Then build.
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

## 1. Product context (university view)

**AptoHire — "Hired for what you can do."** Students prove skills by building real projects; an AI review issues a **verified scorecard**; organizations find students **without seeing names** until the student says yes. Universities get this portal to see **how their students are doing**.

What a university does here (the TPO's job):

1. **Applies** on `aptohire.com`; an admin approves. Until then the portal is a **read-only preview**.
2. **Uploads a roster** (CSV): name, email, year, program, status, student ID. Student IDs are **hashed on the server**; AptoHire never stores them readably. Rosters let students register with **roster-verified** status.
3. **Manages the roster**: update year, mark **Dropout** / **Alumni**, add a single student. These changes update **the student's affiliation record on AptoHire only**. AptoHire never edits the university's own master records.
4. **Sees its students**: verification level, registered or not, verified-skills count, lifecycle status, placement. **Only this university's students.**
5. **Reads analytics**: verified students, skills distribution, average score by skill, **activation funnel (Invited → Registered → Submitted → Verified)**, cohort filters, own trends over time. **No cross-university comparison and no public ranking.**
6. **Tracks placements**: **Placed / Not placed / Unknown**, with "last updated". **Unknown is never treated as Not placed.**
7. **Sees unmatched demand**: when an organization posts a job and no candidate matches, the university sees **the skill, counts and reasons, never student identities**.
8. **Curriculum gap report** (demand vs verified supply) and an **organizations directory** with job posts.

Levels: **Emerging** (0–50, not verified) · **Competent** (51–69) · **Proficient** (70–84) · **Advanced** (85–100). Results under human review (borderline 40–60) are **excluded** everywhere in this portal until confirmed.

**Privacy boundaries this portal must respect (never relax):** a university sees only its own students; student project **files** are never shown (a flag exists, default off, no UI in this phase); the university never learns *who* matched or didn't match a job; unmatched-demand views contain no identities; a student's affiliation with another university is never shown.

---

## 2. Where this project fits

AptoHire is **five separate web projects sharing one Supabase database**:

| Project | Host | Port (local) |
|---|---|---|
| `aptohire-root/` | `aptohire.com` | 5173 |
| `aptohire-student/` | `student.aptohire.com` | 5174 |
| **`aptohire-university/` ← YOU ARE BUILDING THIS** | `university.aptohire.com` | **5175** |
| `aptohire-organization/` | `organization.aptohire.com` | 5176 |
| `aptohire-admin/` | `admin.aptohire.com` | 5177 |

`aptohire-backend/` owns migrations, RLS, seed data and Edge Functions. **This project never edits any of that.** It consumes the database with the anon key, the signed-in staff member's JWT, RLS, Storage, Realtime and the Edge Functions in section 4.

If `../aptohire-student` exists you may **read** it and **copy** (never import): `src/styles/tokens.css`, `src/lib/scoring.ts`, `src/components/proof/ProofRing.tsx`, `StatusChip.tsx`, `src/components/brand/*`, `src/components/shell/*` patterns, `src/lib/supabase/client.ts` pattern. If not, build from sections 6–7.

**Hard boundaries**

- Serves **university staff only**. A signed-in user whose role is not `university` sees "Go to your portal" and **no university data is fetched** (section 5).
- **Never link to or mention the admin portal.**
- **The service-role key must never exist in this repo.** Only `VITE_*` values.
- Never build anything that tries to read another university's data or any student identity beyond this university's own students.

---

## 3. Tech stack and project setup

| Layer | Choice |
|---|---|
| Build | **Vite + React 18 + TypeScript (strict, no `any`; `unknown` + Zod)** |
| Routing | React Router (data router); every route `React.lazy` except shell and login |
| Data | **TanStack Query** + `@supabase/supabase-js` (used **only** in `src/lib/backend/supabase/`) |
| Tables | `@tanstack/react-table` (headless) with our own styling |
| Forms | React Hook Form + **Zod** |
| CSV | `papaparse` (parse and serialise), lazy-loaded on roster pages |
| Styling | **Tailwind CSS 3.4 (pinned)** + CSS variables (section 6) + **shadcn/ui** (Radix); install only primitives you use |
| Icons | `lucide-react`, stroke 1.75, 20 px |
| Fonts | Self-host via Fontsource: `@fontsource-variable/bricolage-grotesque`, `@fontsource-variable/instrument-sans`, `@fontsource/jetbrains-mono` (400) |
| Charts | `recharts` (trends), **lazy-loaded**. Funnel, bars and reason bars are plain SVG/CSS |
| Motion | No animation library. CSS + SVG + a tiny `useCountUp` hook |
| Head | `react-helmet-async` (`noindex` on every route) |
| Tests | Vitest + React Testing Library, Playwright + `@axe-core/playwright` |
| Lint | ESLint (typescript-eslint, react-hooks, jsx-a11y) + Prettier |

### Scripts

`dev` (port **5175**, `--strictPort`), `build`, `preview`, `typecheck`, `lint`, `test`, `e2e`, `db:types` (`supabase gen types typescript --project-id $SUPABASE_PROJECT_REF > src/lib/database.types.ts`; committed; until it exists use hand-written domain types).

### Environment (`.env.example` committed; `.env.local` ignored)

```
VITE_APP_ENV=local                  # local | demo | production
VITE_DEMO_MODE=true                 # demo switcher + prototype notice + demo tools
VITE_USE_MOCKS=true                 # true = in-memory backend, no network
VITE_SITE_URL=http://localhost:5175
VITE_ROOT_URL=http://localhost:5173
VITE_ROOT_DOMAIN=localhost
VITE_STUDENT_URL=http://localhost:5174
VITE_ORGANIZATION_URL=http://localhost:5176
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=             # public anon/publishable key ONLY
VITE_DEMO_PASSWORD=                 # demo/local only; fictional accounts
VITE_ROSTER_MAX_MB=5
VITE_ROSTER_MAX_ROWS=10000
VITE_SUPPORT_EMAIL=support@aptohire.test
```

`src/config/env.ts` parses with Zod at startup and throws a readable error. **Build fails** if `VITE_APP_ENV=production` and `VITE_USE_MOCKS` or `VITE_DEMO_MODE` is true, or `VITE_DEMO_PASSWORD` is set.

`src/config/links.ts` exports `rootUrl(path)` and `portalUrl('student' | 'organization', path)` (explicit env first, else `https://{portal}.${VITE_ROOT_DOMAIN}`). **No other code builds cross-site URLs. There is no admin option.**

### Folder structure

```
aptohire-university/
├─ README.md  .env.example  index.html  package.json  vite.config.ts  tailwind.config.ts
├─ vercel.json               # rewrites + security headers (section 12)
├─ public/                   # favicon.svg, robots.txt (Disallow: /), samples/roster-sample.csv, samples/roster-sample-clean.csv
├─ docs/PLAN.md
├─ src/
│  ├─ main.tsx
│  ├─ app/                   # router.tsx, providers.tsx, RoleGuard.tsx, AuthGate.tsx, UniversityGate.tsx, ErrorBoundary.tsx
│  ├─ config/                # env.ts, links.ts, demoAccounts.ts
│  ├─ components/
│  │  ├─ shell/              # AppShell, SideRail, TopBar, BottomTabBar, MoreSheet, AccountMenu, NotificationBell, SkipLink, PrototypeNotice, PendingBanner
│  │  ├─ data/               # DataTable, FilterBar, CohortFilters, StatRow, Funnel, ReasonBar, SupplyDemandRow, ChartCard, SampleDataBadge
│  │  ├─ proof/              # ProofRing, StatusChip, ConfidenceNote
│  │  ├─ roster/             # CsvDropzone, ColumnMapper, ValidationTable, ImportReport, EntryForm, EntryEditSheet
│  │  ├─ brand/              # Logo, SealMark
│  │  └─ ui/                 # Button, Field, Input, Select, Textarea, Checkbox, Switch, Tabs, Dialog, Sheet, Toast, Skeleton, EmptyState, ...
│  ├─ features/              # api hooks (use*.ts), components, schemas per area
│  │  ├─ auth/ university/ roster/ students/ analytics/ placements/ demand/ directory/ notifications/ account/ config/
│  ├─ pages/                 # thin route components
│  ├─ lib/
│  │  ├─ backend/            # types.ts (Backend interface), index.ts (factory), supabase/, mock/
│  │  ├─ supabase/client.ts  # the ONLY createClient call
│  │  ├─ domain/             # types.ts, mappers.ts
│  │  ├─ schemas/            # zod for every response and form
│  │  ├─ csv.ts  csvExport.ts  rosterValidation.ts  funnel.ts  gaps.ts  placements.ts  scoring.ts  format.ts  cn.ts
│  └─ styles/                # tokens.css, globals.css
├─ tests/                    # vitest
└─ e2e/                      # playwright
```

`scoring.ts`: `levelFor(score)` and `isVerified(score, passMark)` (pass mark and borderline range **always from public config**, constants are fallbacks only, commented *"Mirror of app_config. Keep in sync."*).

---

## 4. Backend contract (what this app consumes)

### 4.1 One interface, two implementations

Define `Backend` in `src/lib/backend/types.ts`, grouped (`auth`, `university`, `roster`, `students`, `analytics`, `placements`, `demand`, `directory`, `notifications`, `config`, `account`) with **Zod-validated** inputs and outputs. Two implementations: `supabase/` (real) and `mock/` (in-memory, 4.6). The factory picks by `VITE_USE_MOCKS`, provided through a `BackendProvider`. **Components never import supabase or call fetch**; they use feature hooks on TanStack Query. Query keys include the university id (`['students', universityId, filters]`); invalidate on mutation; Realtime patches the cache.

### 4.2 Data access (RLS enforces; the app asks only for its own university's data)

| Object | Staff can | Used for |
|---|---|---|
| `profiles` | read own (role) | role guard |
| `university_members` | read own | which university am I in |
| `universities` | read **own row in full** (`id, name, short_name, kind, tier, city, state, status, email_domains, approved_at`) | pending/approved state, settings display |
| `roster_entries` | read own university's rows; **update only** `year, status, program, program_level`; delete **unclaimed** rows | roster management. `student_id_hash` is never selectable and never writable |
| `university_students_v` | read | Students list: `roster_entry_id?, student_id (null if unclaimed), full_name, status, year, program_level, program?, verification ('roster'\|'student_id'\|'unverified'\|null), claimed, verified_skills_count, last_activity, placement_status` |
| `university_student_skills_v` (**backend to add**) | read | Student detail: per own student `skill_name, family, level, score?, verified_on` for **verified and shareable** skills only |
| `university_overview_v` | read | `students_total, verified_students, active_this_month, projects_reviewed` |
| `university_funnel_v` | read | grouped rows `year, program, status, invited, registered, submitted, verified` (client sums by cohort filter) |
| `university_skills_v` | read | `skill_id, skill_name, family, verified_students, assessed_students, avg_score, open_jobs` (cohort-free; `open_jobs` = open jobs demanding the skill across all organizations) |
| `university_trends_v` | read | weekly `week_start, verified_total, reviews, new_registrations` for the last 26 weeks |
| `university_demand_v` | read | unmatched demand rows `id, skill_name, jobs_count, reasons (jsonb: no_eligible, incomplete_assessments, insufficient_evidence, skills_mismatch, opted_out, not_assessed), last_posted_at` (**no student ids**) |
| `placements` | read/insert/update for own students | Placements page: `student_id, status ('placed'\|'not_placed'\|'unknown'), organization_name_text, role_text, source ('university'\|'student'), updated_at` |
| `public_organizations` | read | directory: `id, name, kind, size, website, verified` |
| `public_jobs` (and `jobs` columns universities may read) | read | directory job posts: `id, organization_id, title, mode, location, status` (+ required skills if the contract exposes them) |
| `notifications` | read own; update `read_at` | bell and centre |
| `public_config` | read | pass mark, borderline range |

The portal **never** reads other universities, `students`/`affiliations` of other universities, `match_results`, `candidate_profiles`, `submissions`, `audits` (only the aggregates above), or any storage object except its own `rosters/{university_id}/…` upload.

> **Exact names win from `docs/CONTRACT.md`.** If a view's columns differ, adapt the mapper in `lib/backend/supabase/`. If a view or column is missing, hide that UI and leave `TODO(backend)`.

### 4.3 Storage

| Bucket | Path | Rules |
|---|---|---|
| `rosters` | `{university_id}/{timestamp}.csv` | CSV only, ≤ `VITE_ROSTER_MAX_MB`; write only for **own** university; the backend deletes the file after the real import; **if the user cancels after a dry run, delete the object** (`// TODO(backend)`: ensure a delete policy for the owner, else rely on retention) |

### 4.4 Edge Functions (staff JWT; base `${VITE_SUPABASE_URL}/functions/v1`)

Errors: `{ error: { code, message, fields? } }`. Map every code to calm copy (section 9); never show raw messages.

| Function | Request → Response |
|---|---|
| `complete-signup` | `{ portal: 'university', application_ref?: string }` → `204`. Sets `profiles.role` once (idempotent). If the **verified** email equals an application's contact email (and `application_ref` matches when given) the backend attaches the user to that university. Different role already set → `409 ROLE_ALREADY_SET` |
| `roster-import` | `{ university_id, source: { csv_path } \| { rows }, dry_run?: boolean }` → `ImportReport`. Used for **dry run**, **real import**, and **adding a single student** (`rows` with one entry) |

```ts
type LifeStatus = 'incoming' | 'enrolled' | 'dropout' | 'alumni';   // "Masters" = enrolled + program_level 'pg'
type CanonicalRow = {
  full_name: string; email?: string; student_id_number?: string;     // raw ID travels only to roster-import over TLS
  program?: string; program_level: 'ug' | 'pg'; year?: number; status: LifeStatus;
};
type ImportReport = {
  total: number; valid: number;
  invalid: { row: number; field: string; reason: string }[];
  duplicates: number; created: number; updated: number;
  matched_existing: number;      // COUNT ONLY. Never show which rows or which people.
  proposals_created: number;     // COUNT ONLY
};
```

**Canonical CSV header (what the client sends):** `full_name,email,student_id_number,program,program_level,year,status`. The client maps the user's columns to these, **drops every unmapped column before upload** (data minimisation), and re-serialises. Unmapped columns never leave the browser.

### 4.5 Realtime

Subscribe (RLS-filtered) to `notifications` (own). Always unsubscribe on unmount. If the channel isn't joined within 5 s or drops, poll every 30 s.

### 4.6 Mock backend (required, in-memory, deterministic)

- State in memory and `sessionStorage` (try/catch). **Reset demo data** link in demo mode (Account). Latency 300–800 ms.
- **Three universities** (see 4.7). Mock `roster-import` applies the same validation as the client, detects duplicates against the existing roster, supports `dry_run`, and (for **approved** universities only) reports `matched_existing` / `proposals_created` when an uploaded email equals one of the fictional cross-university people (`avi@demo.aptohire.test`, `niya@demo.aptohire.test`). **Pending** universities save rows but report `matched_existing: 0, proposals_created: 0` (locked until approval).
- In demo mode only, Account shows a **Simulate admin approval** button for the pending university (flips status to `approved`, pushes an `application_decided` notification). This button and its code are excluded from production builds.
- Notifications and demand rows are seeded; the clock for "active this month" and "last activity" is relative to *now* so demos stay fresh.

### 4.7 Fictional seed data (mock mode)

Demo password for all mock accounts is `VITE_DEMO_PASSWORD` (any non-empty value in mock mode).

| Account | University | State |
|---|---|---|
| `tpo@unia.edu.test` | **Uni A** (Tier 2, **approved**, domain `unia.edu.test`) | Rich data: 6 named entries (**Aman** incoming · **Ajay** year 1 · **Aarav** year 2 · **Akshit** year 3 · **Aarti** dropout · **Avi** alumnus) **plus ~84 generated fictional students**, giving a funnel of roughly 90 invited → 58 registered → 31 submitted → 19 verified; skills (React, JavaScript, TypeScript, HTML/CSS, SQL, Accessibility, Testing, Python…) with verified counts, average scores and open-job demand; 26 weeks of trends; placements (Placed / Not placed / **Unknown** examples) |
| `tpo@unib.edu.test` | **Uni B** (Tier 3, **approved**, no email domain) | Smaller: ~40 students, thin analytics (tests sparse states) |
| `tpo@unic.edu.test` | **Uni C** (**pending** application, reference `AH-7K2Q9F`) | Roster pre-uploaded and **locked**: Sumit, Sourav, Sam, Shiva, Shruti, Sohan; dashboard is a labelled **sample preview** |

Unmatched demand (Uni A): **Data Visualization** (1 job, reasons: not yet assessed 40, no eligible candidates 12, skills mismatch 5, incomplete assessments 4, opted out 2, insufficient evidence 3) and **Node.js** (2 jobs). Organizations: **Northwind Labs** (mid-size corporate) and **Brightlane** (start-up), both with the *Verified organization* chip, with their open jobs (e.g., "Frontend Developer (React)", "Junior Full-stack Intern", "UI Engineer (React)", "Data Visualization Analyst").

`public/samples/roster-sample.csv` (fictional; **deliberately contains** an invalid email, a year of `9`, a duplicate email, an unknown status, and one row matching `avi@demo.aptohire.test`) and `roster-sample-clean.csv` (all valid) for the demo and tests.

### 4.8 What the backend project must provide (list in your report; treat each as optional at runtime)

`university_student_skills_v` · stable `roster_entry_id` in `university_students_v` · `program` in `university_students_v` · `my_university_v` or equivalent exposing the application `reference` and a rejection `status_note` · `placements` keyed so **unclaimed roster entries can also carry a placement** (else only registered students can be edited) · a storage **delete** policy for the uploader's own `rosters/` objects · column grants allowing update of `year, status, program, program_level` on own roster rows · demo logins `tpo@unib.edu.test` and `tpo@unic.edu.test` in the seed · the exact shapes of the analytics views. If something is missing: hide the dependent UI, keep the rest working, and mark `TODO(backend)`.

---

## 5. Auth, sessions and gates

- **Supabase Auth, email + password.** Single client in `src/lib/supabase/client.ts`: `storageKey: 'aptohire-university'`, `persistSession`, `autoRefreshToken`, `detectSessionInUrl`, `flowType: 'pkce'`. **No cookie shared across subdomains.**
- **Public routes:** `/login`, `/signup`, `/forgot-password`, `/reset-password`, `/auth/callback`. Everything else is behind gates, in this order: **`AuthGate` → `RoleGuard` → `UniversityGate`**.
- **Sign-up** (`/signup?ref=AH-7K2Q9F`): full name, **work email**, password ≥ 10 chars with a strength hint, and an **Application reference** field prefilled from `?ref` (editable, optional). Helper: "Use the email address you applied with so we can link your account." → `auth.signUp` → "Check your email" screen with resend → `/auth/callback` runs `useBootstrap()`: no role → `complete-signup({portal:'university', application_ref})`; then membership check. Every step is **idempotent and resumable**.
- **Role guard (F-ID-03):** after login fetch **only** the user's own `profiles` row. `university` → continue. `student` or `organization` → **"Go to your portal"** screen with a button to `portalUrl(role)`. Any other role (including admin) → generic "This account can't use the university site." with **no link and no role named**. **No university data hook may run before the guard passes** (enforce with a test).
- **`UniversityGate`:** load `university_members` + own `universities` row.
  - **No membership:** screen "You're not linked to a university yet." Explain: "Apply for access on aptohire.com, then sign up with the email you applied with." Buttons: **Apply for university access** → `rootUrl('/apply/university')`, **Sign out**. Nothing else renders.
  - **`pending`:** app shell in **preview mode** (section 8.1).
  - **`rejected`:** screen "Your application wasn't approved." with support mailto and (if provided) the admin note. All data and write actions are blocked; Sign out available.
  - **`approved`:** full portal.
- **Sessions:** account page offers **Sign out of all devices** (`signOut({scope:'global'})`). Sign-out clears the Query cache and Realtime channels.
- **Login page:** email + password, "Forgot password", link to sign-up, link to `rootUrl('/')`. In demo mode a **Demo accounts** panel with one-click buttons for the three accounts in 4.7 (`src/config/demoAccounts.ts`; `// TODO(backend)` keep in sync with the seed). Excluded from production builds.
- Multiple memberships: use the first, `// TODO(decision):` switcher later.

---

## 6. Design system (identical across all five portals)

**Concept: "Proof you can see."** Quiet, white, spacious; one thing stands out: **evidence and clear numbers**. Green = verified, trusted, done. Orange = attention, in progress, energy. White is the product. For the university portal the character is **institutional and calm**: data-dense but airy, tables and plain figures, not dashboards-of-cards. "Slate" in the identity means `ink` and `moss`, not a new colour.

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
  --portal-strip: linear-gradient(180deg,#e6f8ee,#ffffff);   /* university identity */
}
@media (prefers-reduced-motion: reduce){ *{animation-duration:.001ms!important;transition-duration:.001ms!important;scroll-behavior:auto!important} }
```

Tailwind names: colors `canvas, mist, hairline, ink, moss, verdant.{50,100,300,600,700,800}, ember.{50,100,300,500,700}, danger.{50,600}`; radius `control, panel, feature`; shadow `float` (shadow-1), `modal` (shadow-2); fonts `display, body, mono`. **No hard-coded hex in components.** Light theme only.

### 6.2 Rules

- **Colour:** primary button = **white on `verdant-700`**; accent = **`ink` on `ember-500`** (never white on orange); accent is used for the single most energetic action on a page (**Upload roster** on empty states). `verdant-600` / `ember-500` for graphics and large elements only; small green text `verdant-700`, small orange text `ember-700`. Colour is never the only signal (icon or word alongside).
- **Gradients:** `portal-strip` only in the top bar strip and the Overview header; **at most one gradient surface per viewport**; none on buttons, cards, table rows or behind text smaller than 16 px.
- **Type:** Bricolage Grotesque (headings 600–700; large numbers 600), Instrument Sans (UI), JetBrains Mono only for IDs, references, CSV fragments. Sentence case; no tracked all-caps labels; no eyebrow labels unless they carry information; `tabular-nums` everywhere numbers appear. Lines ≤ 68 chars in prose. Left-aligned.

| Token | Size | Line-height |
|---|---|---|
| `h1` | `clamp(28px, 4vw, 36px)` | 1.2, −0.015em |
| `h2` | `clamp(22px, 2.6vw, 28px)` | 1.3 |
| `h3` | `clamp(18px, 2vw, 20px)` | 1.35 |
| `stat` | `clamp(28px, 3.4vw, 40px)` Bricolage 600 | 1.1 |
| `body` | **16px minimum** | 1.6 |
| `small` | **14px minimum** (table cells, helper text) | 1.5 |

- **Spacing** (4 px base): 4 · 8 · 12 · 16 · 24 · 32 · 48 · 72. Content max **1200 px** (tables may use the full shell width), page padding 24 px desktop / 16 px mobile.
- **Radius encodes hierarchy:** `control` 10 (inputs, buttons; chips full-pill), `panel` 16 (dialogs, sheets), `feature` 28 (Overview header panel only).
- **Flat by default** with hairline borders. Shadow only for floating things (`float`: menus, popovers; `modal`: dialogs, side sheets). **Data lives in tables and lists with hairlines; cards are not used in this portal except for the Directory's organization rows if needed.** No card grids.
- **Motion:** this portal has **no signature animation**; the Proof Ring reveal appears only in the student detail. Everything else responds to actions only. Reduced motion → final states instantly.
- **Icons:** Lucide. Empty states: palette circles/arcs on `mesh`; no stock photography, no photos of people.

### 6.3 App shell

```
┌────────────┬──────────────────────────────────────────────────────┐
│ ◖ AptoHire │ [☰ at 768–1023]       🔔   Uni A · T. Rao ▾          │ ← 56px top bar, portal-strip behind
│ university ├──────────────────────────────────────────────────────┤
│ Overview   │   Page title (h1)                       [Primary]    │
│ Students   │   One-line description (moss)                        │
│ Roster     │   ───────────────────────────────────────────────    │
│ Placements │   content                                            │
│ Skills     │                                                      │
│ demand     │                                                      │
│ Directory  │                                                      │
│ Help       │                                                      │
└────────────┴──────────────────────────────────────────────────────┘
```

| Width | Behaviour |
|---|---|
| ≥ 1280 | Full 232 px rail |
| 1024–1279 | Rail collapses to 72 px icons with tooltips |
| 768–1023 | Rail becomes a **drawer** opened by a hamburger (☰/✕, 44×44, `aria-expanded`, focus trap, Escape closes and returns focus, closes on navigation and on resize ≥ 1024) |
| < 768 | **Bottom tab bar**: Overview · Students · Roster · Placements · **More** (sheet: Skills demand, Directory, Notifications, Account & security, Help, Sign out). Sheets become full-screen |

Nav items are **exactly**: Overview · Students · Roster · Placements · Skills demand (Directory, Help, Account in the rail footer / More). Show the university name and a **status chip** (Approved / Pending) next to it in the account menu. Skip-to-content link first. Minimum width **360 px**. Respect `env(safe-area-inset-*)`.

---

## 7. Signature components

### 7.1 `Funnel` (the portal's signature)

Four labelled rows **Invited → Registered → Submitted → Verified**. Each row: label, count (`stat`-style number), a bar whose width = `count ÷ invited`, and the **conversion from the previous step** in words ("64% of invited"; "53% of registered"). Bars use solid tints: Invited `verdant-100`, Registered `verdant-300`, Submitted `verdant-600`, Verified `verdant-800`; bars sit on `mist` tracks with a 1 px hairline. A drop-off note under the funnel names the **largest drop** in plain words ("The biggest drop is between Registered and Submitted. 27 students signed up but haven't submitted a project yet."). Bars grow on first view over 600 ms (disabled under reduced motion). Accessible as a real table in a visually-hidden element or `aria-label` per row. Pure `lib/funnel.ts` for the math (guard divide-by-zero → "—").

### 7.2 `DataTable`

Built on `@tanstack/react-table`: sticky header on `mist`, 1 px hairlines, row height 48 px desktop, sortable columns with `aria-sort`, optional selectable rows (checkbox column, 44 px hit area, select-all with count "23 selected"), keyboard-navigable rows (Enter opens), pagination (25 per page, "Showing 1–25 of 212"), loading skeleton rows, empty and error states with Retry. **Scrolls inside its own container** at all widths; **below 640 px it becomes a labelled list** (each row stacked, `data-label` pairs) without losing actions.

### 7.3 `StatusChip` and domain chips

| Chip | Look |
|---|---|
| Verified (skills) / Verified university | `verdant-100` fill, `verdant-800` text, seal icon |
| Verified via roster / Verified via student ID | `verdant-50` fill, small seal, text names the method |
| Unverified | white, dashed `hairline` border, `moss` text |
| Registered | `verdant-50` fill, check |
| Not registered yet | `mist` fill, `moss` text |
| Lifecycle: Incoming · Enrolled · year N · Dropout · Alumni · Masters | `mist` fill `ink` text; Enrolled uses `verdant-50`; Dropout uses `ember-50`/`ember-700` |
| Placement: Placed | `verdant-100` / `verdant-800`, check |
| Placement: Not placed | `mist` / `moss` |
| Placement: **Unknown** | white, dashed `hairline`, `moss` (always visually distinct from Not placed) |
| Pending application | `ember-50` / `ember-700` |
| Approved | `verdant-100` / `verdant-800` |
| Verified organization | `verdant-50` fill, small seal |

### 7.4 `ReasonBar` (unmatched demand)

A stacked horizontal bar of the six reasons for one demanded skill, **with a legend and counts as text** (never colour alone; each segment has a distinct pattern or label). Labels: **No eligible candidates · Incomplete assessments · Insufficient evidence · Skills mismatch · Opted out · Not yet assessed**. Order by size. A one-line, rule-based suggestion beside it (see 8.5).

### 7.5 `ProofRing`

Used only in the **student detail** (56 px per verified skill). Props: `score`, `skill`, `state: 'verified'|'under-review'|'loading'`, `size: 56|96`, `level`, `animate?`. Track `hairline`; arc SVG circle with round caps, length = score ÷ 100 from 12 o'clock, gradient `#FF7A1A → #FFB470 → #8FE0B2 → #0E9F5B`; seal on verified; `role="img"` with `aria-label="React, score 72 of 100, Proficient, verified"`. Reveal 900 ms once; reduced motion → final state. Unit-test the aria-label builder.

### 7.6 Other UI

Buttons (primary / accent / secondary / ghost / danger; height 44 mobile, 40 desktop; label 15/600; loading keeps label and fixed width), Field (label **always above**, helper 14 px, error with icon, `aria-describedby`), Tabs (underline, 2 px `verdant-700`), Dialog and Sheet (focus trap, Escape, restore focus; side sheets `modal` shadow, full-screen on mobile), Toast (bottom-centre, 4 s, reuses the action's verb), Skeleton (matches layout), EmptyState (palette circles on `mesh`, one sentence, one button), `StatRow` (plain figure + label inline, hairline dividers, no cards), `CohortFilters` (Year, Program, Status selects; "Clear filters"; filter state in the URL query), `SampleDataBadge` (a quiet `mist` pill "Sample data").

---

## 8. Screens and flows

Routes: `/login` · `/signup` · `/forgot-password` · `/reset-password` · `/auth/callback` · `/` Overview · `/students` · `/students/:studentId` · `/roster` · `/roster/import` · `/placements` · `/skills` (tabs via `?tab=supply|unmatched|gaps`) · `/directory` · `/notifications` · `/account` · `/help` · `*` 404. Every page: `h1`, document title, loading skeleton, empty state, error state with **Retry**, `noindex`.

### 8.1 Pending application: read-only preview (F-UN-01)

While `status='pending'`:

- A persistent, non-dismissible **`PendingBanner`** under the top bar: "Your application is under review. Reference **AH-7K2Q9F**. You'll get an email when it's approved." (reference in mono with a copy button).
- **Overview, Students, Placements, Skills demand and Directory** render a **labelled sample preview** (`SampleDataBadge` + a plain note "This is sample data so you can see what your dashboard will show") built from deterministic fixtures, **all write actions disabled** with tooltips ("Available after approval"). Never present sample numbers as the university's own.
- **Roster is the exception:** upload and single-add are **allowed** (backend saves the rows locked). After a pending import show: "Your roster is saved. Matching and student sign-ups start once your university is approved." No matching counts are shown.
- The Students list for a pending university shows the **locked roster entries** (names, year, status) with a "Locked until approval" chip.
- When the status flips to `approved` (Realtime notification or next load), show a one-time success toast "Your university is approved." and refresh data. In mock demo mode this is triggered by **Simulate admin approval**.

### 8.2 Overview `/` (F-UN-06, F-UN-11)

Header panel (`r-feature`, `portal-strip` behind) with the university name, a status chip, and a one-line summary ("212 students on your roster. 19 have a verified skill."). Then, in a single wide column:

- **Stat row** (plain, hairline-divided, no cards): **Verified students · Active this month · Projects reviewed** (+ "Students on roster").
- **Cohort filters** (Year · Program · Status) applying to the funnel and the skills table; state in the URL.
- **Funnel** (7.1).
- **Skills table** (top 8 by verified students): **Skill · Verified students · Average score · Demand from open jobs** with a link "See all skills". Average score shown with its level word ("68 · Competent"). Verified-only counts; **review-pending results excluded**.
- **Unmatched demand** list (top 5): **skill, number of jobs, reason counts**, never names; link to Skills demand → Unmatched.
- **Trends** (F-UN-11): line chart (Recharts, lazy) of **verified students over time** and **reviews per week**, with **View as table** toggle. Copy: "Your own trends. AptoHire doesn't compare universities or publish rankings." Fewer than 3 data points → "Trends appear after a few weeks of activity."
- **Empty (no roster):** "Add your students to get started. Upload a CSV with name, email, and year." → accent **Upload roster**.

### 8.3 Roster `/roster` and import `/roster/import` (F-UN-02, F-UN-03)

**Roster page:** top-right buttons **Upload roster** (accent when the roster is empty, otherwise primary) and **Add a student** (secondary). A short, quiet explainer: "Roster is your master list. Updating a student here changes how they appear on AptoHire. Your own records aren't touched."

- Table (`DataTable`): **Name · Email · Program · Year · Status · Registered**; search by name or email (client filter over the loaded page; server `ilike` in the real implementation), filters for Year, Program, Status, Registered; selectable rows; row actions in a menu: **Edit**, **Mark as alumni**, **Mark as dropout**, **Remove** (**only for unregistered rows**; registered rows show "Registered students can't be removed. Mark them as alumni or dropout.").
- **Edit** (side sheet): year, program, program level, status only (name/email/ID aren't editable in this phase; `// TODO(backend)`). Warn when changing status to Dropout or Alumni: "This changes how the student appears on AptoHire. Their scores and credential don't change." Confirm dialog for destructive changes. Toast reuses the verb ("Marked as alumni.").
- **Bulk actions** for selected rows (max 500 per action): **Mark as alumni**, **Mark as dropout**, with a confirm dialog that states the count. Optimistic update with rollback on error.
- **Add a student** (dialog): the `CanonicalRow` fields (name required; email and/or student ID strongly recommended with the explanation "We use these to match students when they sign up"); submits through `roster-import({ rows:[…] })`; success toast "Student added."
- **Export CSV** of the current filtered view (section 8.8 rules).
- **Empty:** "Add your students to get started. Upload a CSV with name, email, and year." → **Upload roster**.

**Import `/roster/import`:** a guided wizard (step list in words: "Step 2 of 4"), keyboard- and screen-reader-friendly:

1. **Choose file.** Dropzone + "Browse". Accept `.csv` only (UTF-8). Link to **Download a sample CSV** (`/samples/roster-sample-clean.csv`). Client checks before reading: extension and MIME (`text/csv`, `application/vnd.ms-excel`, empty-MIME tolerated), **size ≤ `VITE_ROSTER_MAX_MB`**, text only (**reject if NUL bytes**), rows ≤ `VITE_ROSTER_MAX_ROWS`. Errors in plain words ("This file is 7.4 MB. The limit is 5 MB. Split it into smaller files.", "This doesn't look like a CSV file. Save your sheet as CSV (UTF-8) and try again."). Mention "Excel files (.xlsx) aren't supported. Save as CSV."
2. **Map columns.** Show the first 5 rows; for each canonical field a select of the file's headers with **best-guess auto-mapping** (case/space-insensitive synonyms: "Name/Full name/Student name", "Email/E-mail", "Roll no/Enrollment/ID", "Branch/Department/Program", "Year/Semester year", "Status"); required: **Name** and at least one of **Email** or **Student ID**; **Year**, **Program**, **Status**, **Program level** optional (defaults: status `enrolled`, level `ug`). Show a plain note: "Columns you don't map are removed in your browser and never uploaded." Value normalisation (case-insensitive): status synonyms (`active`→enrolled, `graduated`/`passed out`→alumni, `dropped out`/`left`→dropout, `new`/`admitted`→incoming); year accepts `1`–`7` or `1st year` style.
3. **Check.** Client validation table (`ValidationTable`, scrolls in its own container, errors first): **required name; valid email; year 1–7; known status; duplicates within the file (email, student ID)**; each problem has row number, field, reason in plain words. Then the **server dry run**: upload the minimal canonical CSV to `rosters/{university_id}/{timestamp}.csv` and call `roster-import({ csv_path, dry_run:true })`; show the `ImportReport` (total, valid, invalid, duplicates, "will be created", "will be updated"). **Downloadable error report** (CSV of invalid rows with row, field, reason). The user may **go back and fix mapping** or **continue importing valid rows only** (the invalid ones are skipped and listed). `matched_existing` and `proposals_created` appear **only as counts** after the real import ("1 student already has an AptoHire profile. We asked them to confirm their new university.") and **never** identify rows or people.
4. **Result.** After `roster-import({ csv_path, dry_run:false })`: success summary with created/updated/skipped, buttons **View roster** and **Import another file**. Pending university → the saved-and-locked note (8.1). Cancel at any step deletes the uploaded temp object.

Handle `VALIDATION_FAILED` (map to file-level message), `RATE_LIMITED` ("Too many imports. Try again in a few minutes."), `FORBIDDEN`, network errors (retry; the dry run is safe to repeat).

### 8.4 Students `/students` and detail `/students/:studentId` (F-UN-04, F-UN-05)

**List:** read-only analytical view: "Roster is your list. This page shows how each student is doing on AptoHire." Columns: **Name · Status (lifecycle chip) · Verification chip · Registered · Verified skills · Last activity · Placement**. Filters: search by name, Status, Verification (Verified via roster / Verified via student ID / Unverified), Registered/Not registered, Year, Program, **Has a verified skill**, Placement. Sort by name, verified skills, last activity. URL-synced filters. **Export CSV** (8.8).
Clicking a **registered** row opens the detail page. Clicking an **unregistered** row opens a small side sheet: roster details, "Hasn't registered on AptoHire yet.", and **Copy sign-up link** (`portalUrl('student','/signup')`). (No email invites in this phase; `// TODO(decision)`.)

**Detail `/students/:studentId`:** header with name, lifecycle chip, verification chip, program/year, registered date, last activity. Sections:

- **Verified skills:** list rows, each with a 56 px `ProofRing`, skill name, level chip, "Verified on 12 Oct 2026"; scores only if the backend returns them. Skills **under human review are never shown**. Empty: "No verified skills yet."
- **Scorecard summary:** counts (verified skills, projects reviewed) and the strongest skill, from the data returned.
- **Placement:** current status chip, organization and role text if present, "Last updated 2 days ago · Source: University", and **Update placement** (opens the same editor as the Placements page).
- A quiet privacy note: "Project files and videos are private to the student."
- Do not show other universities, emails of the student beyond the roster, or any organization interaction (requests, matches, feedback).

### 8.5 Skills demand `/skills` (F-UN-08, F-UN-09)

Tabs: **Supply vs demand** · **Unmatched demand** · **Curriculum gaps**.

- **Supply vs demand:** table **Skill · Family · Verified students · Assessed students · Average score · Open jobs**. Filter by family; sort; search. Verified-only; pending-review excluded.
- **Unmatched demand (alerts):** one row per demanded skill: skill name, "N jobs", last posted date, a `ReasonBar` with counts and legend, and a **rule-based suggestion** from the largest reason (keep copy in `content/demand.ts`):
  - Not yet assessed → "Many students haven't been assessed yet. Encourage them to submit a project in this skill."
  - Incomplete assessments → "Students started but didn't finish. A short workshop may help."
  - Insufficient evidence → "Reviews lacked evidence. Remind students to include a demo video and clear README."
  - Skills mismatch → "Students' verified skills differ from this job. Consider this skill for your curriculum."
  - No eligible candidates → "No students are verified in this skill yet."
  - Opted out → "Some students have chosen not to be discoverable. Remind them what discovery means: employers see skills, not names."
  Banner copy (always visible): "You see counts and reasons only. Student identities are never shown here." **Never render any student name in this tab** (tested).
- **Curriculum gaps (P1):** **Top demanded skills vs verified supply**: rank by open jobs; gap label from `lib/gaps.ts`: **No verified students** (supply 0), **Some gap** (supply < open jobs), **Covered**; document the thresholds in the file `// TODO(decision)`. Plain bars with numbers; copy "A gap means more jobs ask for a skill than students are verified in it. It's a signal, not a score."

### 8.6 Placements `/placements` (F-UN-07)

- **Stat row:** Placed · Not placed · **Unknown** (counts). Percentages are computed **only among students with a known status** and labelled that way ("62% of students with a known outcome"); Unknown is shown separately and **never added to Not placed** (`lib/placements.ts` + unit tests).
- Table: **Name · Status · Program/Year · Placement · Organization · Role · Source · Last updated**; filters: Alumni only (default on), Year, Program, Placement status. Selectable rows with bulk **Set status** (Placed / Not placed / Unknown).
- **Edit placement** (side sheet): status (radio: Placed / Not placed / Unknown), organization name (free text ≤ 80), role (≤ 80), shown "Source: University". Saving writes `placements` with `source='university'`. **Unknown is the default and is never inferred.** If the student set the status ("Source: Student confirmation"), show it and let the university override with a confirm dialog.
- Unregistered roster entries show Placement **Unknown (not registered)** read-only unless the backend supports roster-keyed placements (`TODO(backend)`).
- Export CSV (8.8).

### 8.7 Directory `/directory` (F-UN-10)

Organizations list (from `public_organizations`): name, kind, size, **Verified organization** chip, website link (`rel="noopener noreferrer"`), open-jobs count. Selecting an organization opens a side sheet with its job posts (**title, mode, location, status**, required skills if exposed). Search + filter by kind. **No recruiter names, emails or any candidate information.** Empty: "No organizations yet."

### 8.8 CSV export rules (students, roster, placements)

Client-side with `papaparse`, only the **visible columns**, UTF-8 with BOM. **CSV-injection guard:** any cell beginning with `=`, `+`, `-`, `@`, tab or carriage return is prefixed with a single quote (`'`). Before download show a dialog: "This file contains student names. Store it securely and delete it when you're done." with **Download** / **Cancel**. File names: `aptohire-<students|roster|placements>-<university-short-name>-<YYYY-MM-DD>.csv`. Unit-test the guard.

### 8.9 Notifications, account, help, system screens

- **Notifications:** bell with unread count (`aria-label="Notifications, 3 unread"`) opening a popover (latest 8) with **Mark all as read** and **View all** (`/notifications`, grouped Today / Earlier). Kinds and deep links: `application_decided` → Overview · `roster_matched` → Roster · `demand_alert` → Skills demand → Unmatched · `student_registered` → Students (if provided). Realtime inserts show a toast. Empty: "You're all caught up."
- **Account `/account`:** email (read-only here; change via support), password change, **Sign out of all devices**, and in demo mode **Reset demo data** and **Simulate admin approval** (pending university only).
- **Help `/help`:** short page: how the roster works, what "verified via roster" means, how placements work (Unknown ≠ Not placed), what unmatched demand shows, privacy of student data, contact support (mailto), links to `rootUrl('/privacy')`, `rootUrl('/terms')`, `rootUrl('/ai-and-fairness')`.
- **System screens:** wrong-portal screen (5), no-membership and rejected screens (5), 404 ("That page doesn't exist." + **Go home**), `ErrorBoundary` (**Reload page**; never a stack trace), offline banner, and the **PrototypeNotice** (slim, dismissible per session) when `VITE_APP_ENV !== 'production'`: "Prototype. All people, universities and organizations shown are fictional."

---

## 9. Copy and voice

Plain, warm, specific. Sentence case. Active verbs. **No hype words.** Say what happened, not how the system works. Errors say what happened and the fix and **don't apologize**. Vocabulary (use consistently): **Roster · Student · Registered · Verified · Placement · Demand · Cohort · Upload roster · Add a student.** Avoid "audit engine", "agent", "pipeline", "RLS", "hash" in visible copy (except "stored securely"). A button's verb is reused in its toast.

| Moment | Copy |
|---|---|
| Empty roster | Add your students to get started. Upload a CSV with name, email, and year. |
| Pending | Your application is under review. You'll get an email when it's approved. |
| Roster saved while pending | Your roster is saved. Matching and student sign-ups start once your university is approved. |
| ID privacy | We store only a secure fingerprint of student IDs, never the numbers themselves. |
| Import done | Roster imported. 84 students added, 6 updated, 2 skipped. |
| Matched existing | 1 student already has an AptoHire profile. We asked them to confirm their new university. |
| Dropout/alumni change | This changes how the student appears on AptoHire. Their scores and credential don't change. |
| Unknown placement | Unknown means we don't have an outcome yet. It doesn't mean not placed. |
| Demand privacy | You see counts and reasons only. Student identities are never shown here. |
| Not registered | Hasn't registered on AptoHire yet. |
| Export warning | This file contains student names. Store it securely and delete it when you're done. |
| Rate limited | Too many attempts. Try again in a few minutes. |
| Network error | We couldn't reach AptoHire. Check your connection and try again. |
| Session expired | Your session ended. Sign in again to continue. |
| File too large | This file is 7.4 MB. The limit is 5 MB. Split it into smaller files. |

Error-code mapping: `VALIDATION_FAILED` map `fields`/file-level message · `RATE_LIMITED` rate copy · `FORBIDDEN` "You don't have access to that." · `ROLE_ALREADY_SET` "This email is already used for another AptoHire site. Sign in there." (link via `portalUrl` only for student/organization) · unknown → network copy. All strings live in `src/content/*.ts` or per-feature `copy.ts`.

---

## 10. Quality bars

### 10.1 Responsiveness (built in, not deferred)

Breakpoints: **Mobile < 768 (base) · Tablet 768–1023 · Desktop ≥ 1024**. Tailwind `md` 768, `lg` 1024, `xl` 1280.

- `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">`; safe-area insets on fixed bars.
- **No horizontal scroll from 360 px up.** Fix root causes; never `overflow-x:hidden` on `body` as a patch. Tables scroll inside their own container at ≥ 640 px and become labelled lists below.
- Multi-column layouts collapse below 768; flex rows wrap/stack; mobile section padding ≥ 40 px vertical; nothing touches the screen edge.
- Body ≥ 16 px, small ≥ 14 px, line-height ≥ 1.4; headings use the fluid tokens.
- **Touch targets ≥ 44×44 px** (buttons, icon buttons, tab-bar items, row checkboxes, row-action menus, filter chips).
- **Forms:** inputs 100% width on mobile, **font-size ≥ 16 px**, labels above inputs, form buttons full-width on mobile, correct `type`/`inputMode`/`autocomplete`.
- The import wizard, column mapper and filters must be fully usable on a phone.
- Test at **360, 375, 390, 768, 1024, 1440**.

### 10.2 Accessibility (WCAG 2.2 AA)

Landmarks, one `h1` per page, skip link, **visible 2 px `verdant-700` focus outline with 2 px offset plus a 4 px white halo**, all flows keyboard-operable, focus trapped/restored in dialogs/sheets/drawers/menus, `aria-live="polite"` for import progress, toasts and filter-result counts ("212 students"), errors announced and focus moved to the first invalid field, **tables have proper headers, `scope`, captions or `aria-label`, `aria-sort`**, selection announced, colour never the only signal (reason bars, chips, placement states), reduced motion respected. `@axe-core/playwright` zero serious/critical on every route.

### 10.3 Performance

Mobile Lighthouse ≥ 90 (Performance), ≥ 95 (Accessibility, Best Practices). Route-level code splitting; **lazy-load** `recharts`, `papaparse` (roster routes and export only), `@tanstack/react-table` heavy columns where sensible; initial JS for login and Overview ≤ **200 KB gzipped**. Server-side pagination (25 per page) in the real implementation; client pagination only in the mock. Debounce search (300 ms). Skeletons match final layouts.

---

## 11. Security and privacy rules (never break)

1. **No secrets in repo or bundle.** Only `VITE_*`. Never the service-role/secret key. `.env*` git-ignored except `.env.example`. A static script fails the build on `service_role`, `sb_secret`, or hard-coded hex colours in components.
2. **Never trust the client for role, ownership or university scope.** The UI hides things for convenience; **RLS and Edge Functions decide**. Never solve a permission error by moving a check into the frontend or widening a query.
3. **Query only this university's data and only the columns rendered.** No code may request other universities' rows, `match_results`, `candidate_profiles`, or per-student submissions/audits.
4. **No PII in logs, analytics, error messages, URLs or storage.** Allowed storage: the Supabase session (`aptohire-university`), notice dismissals, mock state (mock mode only). **Student names must not appear in URLs** (use ids) **or in document titles**.
5. **Uploads are untrusted:** extension and MIME allow-list, size and row caps, NUL-byte rejection, UTF-8 only, parse with `papaparse`, **never render cell contents as HTML** (React escaping; no `dangerouslySetInnerHTML`), generated safe file names (`{timestamp}.csv`), **drop unmapped columns before upload**, delete abandoned temp objects.
6. **CSV export injection guard** (8.8) and the "contains student names" confirmation.
7. **Identity-safe analytics:** unmatched-demand and skills views must contain no student names, ids or emails. Add a test that renders those views with the mock names present in the store and asserts none appear in the DOM.
8. **External links** use `rel="noopener noreferrer"`; cross-site links built only via `links.ts`.
9. **Demo-only code** (demo switcher, mocks, reset, simulate approval, prototype notice) is excluded or disabled in production.
10. Pin exact dependency versions; run `npm audit` and report results.

---

## 12. Deployment (its own Vercel project)

`vercel.json`: rewrite everything not a file to `/index.html`. Headers on all routes: `Content-Security-Policy` (`default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob: https://*.supabase.co; font-src 'self'; connect-src 'self' https://*.supabase.co wss://*.supabase.co; frame-ancestors 'none'; base-uri 'self'; form-action 'self'; object-src 'none'; worker-src 'self' blob:`), `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy: camera=(), microphone=(), geolocation=()`, `X-Frame-Options: DENY`, `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`, `Cross-Origin-Opener-Policy: same-origin`, `X-Robots-Tag: noindex`. Cache hashed assets `immutable`; `index.html` `no-cache`.

README must explain: deploy as **its own Vercel project** with domain `university.aptohire.com`; set the `VITE_*` variables; in Supabase **Authentication → URL Configuration** add `https://university.aptohire.com/**` (and `http://localhost:5175/**`); the other four portals are separate projects; Vercel's free Hobby plan is for non-commercial demos.

---

## 13. Testing

| Level | Cover |
|---|---|
| Unit (Vitest) | `rosterValidation.ts` (required name, email format, year 1–7, status synonyms, duplicates, row cap), column auto-mapping, **CSV export injection guard**, `funnel.ts` (conversion %, divide-by-zero, largest drop), `gaps.ts` thresholds, `placements.ts` (**percentages exclude Unknown**; Unknown never counted as Not placed), status/lifecycle label mapping, `links.ts`, `env.ts`, Zod schemas (valid and invalid), mock `roster-import` (dry run, duplicates, pending-university lock, counts-only matching) |
| Component (RTL) | RoleGuard (university ok; student/org → "Go to your portal"; admin → generic without link; **no data hooks fire before the guard passes**), UniversityGate states (no membership, pending preview with disabled writes, rejected, approved), import wizard (each step, every error message, back/forward, cancel deletes temp object, invalid rows skipped, counts-only matching), roster edit/bulk actions (confirm dialogs, optimistic rollback), registered rows can't be removed, DataTable (sort, selection, keyboard, list mode below 640), Students filters + URL sync, placement editor (Unknown default, student-sourced override confirm), **demand views never render student names**, export confirmation dialog |
| E2E (Playwright, mock mode) | TPO Uni A signs in via the demo panel → Overview (stat row, funnel, skills, unmatched demand, trends toggle) → open Data Visualization demand and read the reasons → Students filtered to "Has a verified skill" → open Aarav → update placement → Roster: upload `roster-sample.csv`, see invalid rows, go back, upload `roster-sample-clean.csv`, import, see the result with counts only → export CSV (assert BOM and injection guard). Uni C pending: banner, sample preview badges, disabled writes, roster upload saved-and-locked note, **Simulate admin approval** → approved. Wrong-role login screens. Axe on every route; **no horizontal overflow at 360, 390, 768, 1024, 1440**; keyboard-only runs of the import wizard and placement editing; reduced-motion run |
| Static | `typecheck`, `lint`, secret/hex grep over `src/` and `dist/` |

---

## 14. README

Beginner-friendly (short sentences, numbered steps): what this project is; how it relates to the other four portals and the shared database; prerequisites (Node LTS, Git); install; copy `.env.example`; **run in mock mode**; connect to a real Supabase project (URL + anon key, set `VITE_USE_MOCKS=false`, copy `docs/CONTRACT.md` from `aptohire-backend`, run `npm run db:types`); demo accounts (emails only: the three in 4.7); commands table; folder tour; **how to prepare a roster CSV** (columns, example, tips) and where the sample files are; deployment to Vercel; how to edit copy and tokens; common problems (white page → env typo; "permission denied" → wrong role; import rejected → file size/format; Supabase project paused → Restore); glossary (roster, registered, verified via roster, cohort, unmatched demand, placement, RLS, mock mode). State clearly: **this app never needs the service-role key.**

---

## 15. Build order (finish each phase, run the checks, then continue)

1. **Foundations:** Vite/TS/Tailwind, tokens, fonts, `env.ts`, `links.ts`, router, providers, `Backend` interface, **mock backend skeleton**, Supabase client module, lint/test/e2e setup.
2. **Design primitives and shell:** Button, Field family, Dialog/Sheet/Toast/Tabs/Skeleton/EmptyState, chips, Logo/SealMark, `AppShell` (rail, top bar, drawer, bottom tab bar, More sheet), PrototypeNotice, `PendingBanner`.
3. **Auth and gates:** login (+ demo panel), sign-up with reference, callback, `useBootstrap`, `AuthGate` / `RoleGuard` / `UniversityGate` (no membership, pending, rejected, approved), password reset, account menu, tests.
4. **Data components:** `DataTable` (with list mode), `FilterBar`/`CohortFilters` (URL-synced), `StatRow`, `Funnel`, `ReasonBar`, `SampleDataBadge`, math libs with unit tests.
5. **Roster:** list, edit sheet, bulk actions, add student, **import wizard** (parse, map, validate, dry run, import, report), sample CSVs, tests.
6. **Students:** list, unregistered sheet, detail with `ProofRing` skills and placement summary, CSV export with guard.
7. **Analytics:** Overview (stat row, cohort filters, funnel, skills table, unmatched demand, trends), pending preview fixtures.
8. **Skills demand and placements:** supply vs demand, unmatched demand with suggestions, curriculum gaps, placements page and editor.
9. **Directory, notifications, account, help, system screens.**
10. **Hardening:** real `supabase/` implementation of every `Backend` method (adapting mappers to `docs/CONTRACT.md`), axe, Lighthouse, overflow checks, keyboard and reduced-motion passes, secret/hex greps, privacy DOM tests, `npm audit`, README.

---

## 16. Acceptance checklist

- [ ] `typecheck`, `lint`, `test`, `build`, `e2e` pass; no `any`; no leftover `console.log`
- [ ] Entire app works with `VITE_USE_MOCKS=true` and no network; switching to the real backend needs only env changes plus the items in 4.8
- [ ] Gates work in order: wrong roles blocked with no data fetched; admin never linked or named; no-membership, pending and rejected states each designed
- [ ] Pending university: banner with reference, **sample-labelled** preview, disabled writes, roster upload saved-and-locked, approval flow works
- [ ] Roster import: `.csv` only, size/row/NUL checks, auto-mapping, **unmapped columns never uploaded**, client validation + server dry run, downloadable error report, valid-only import, **counts-only** matching messages, temp object deleted on cancel
- [ ] Roster edits change only `year/status/program/program_level`; registered rows can't be removed; bulk actions confirmed and reversible on error
- [ ] Students and Roster have distinct purposes; student detail shows verified, shareable skills only (never under-review results) and no files
- [ ] Overview: stat row, cohort-filtered funnel with conversion and largest-drop note, skills table, unmatched demand, trends with a table alternative, no cross-university comparison anywhere
- [ ] Placements: Unknown is never counted or styled as Not placed; percentages exclude Unknown; source and last-updated shown
- [ ] Unmatched demand and skills views contain **no student names or ids** (DOM test)
- [ ] CSV export: visible columns only, BOM, injection guard, "contains student names" confirmation
- [ ] Pass mark and borderline range read from config, never hard-coded in UI logic
- [ ] At most one gradient surface per viewport; none on buttons/cards/table rows; sentence case; no tracked all-caps labels; contrast rules respected
- [ ] Bottom tab bar < 768, drawer 768–1023, icon rail 1024–1279, full rail ≥ 1280; touch targets ≥ 44 px; inputs ≥ 16 px; no horizontal scroll at 360–1440; tables become labelled lists below 640 px
- [ ] Axe: zero serious/critical on every route; Lighthouse targets met; initial JS ≤ 200 KB gzipped
- [ ] No secrets or PII in repo/bundle/logs/URLs; demo code excluded from production builds; production build fails if mocks/demo are on
- [ ] README and `docs/PLAN.md` complete; every `TODO(decision)` and `TODO(backend)` listed in the report

---

## 17. Final report (output exactly this at the end)

### ✅ University portal build report

**Stack used:** [versions]

**Routes built:** [list]

**Backend interface:** [methods implemented in `mock/` and in `supabase/`; any method still mock-only]

**Backend items required (4.8) and their runtime fallbacks:** [list; mark which are blocking]

**Decisions made (`TODO(decision)` / `TODO(backend)`):** [each item and the default chosen]

**Quality results:**
- Lighthouse (mobile): Performance [x] · Accessibility [x] · Best Practices [x]
- Initial JS (gzipped): login [x KB] · Overview [x KB]
- Axe violations (serious/critical): [0 per route]
- Overflow check at 360 / 375 / 390 / 768 / 1024 / 1440: [pass/fail]
- Privacy DOM test (no student names in demand/skills views): [pass/fail]

**Navigation:** [bottom tab bar, drawer and rail behaviours verified]

**Files created:** [grouped list]

**What to check manually:**
- Chrome DevTools → Device Toolbar (Ctrl+Shift+M) at 375, 390, 768, 1024 and 1440 px
- Sign in as each of the three demo accounts; run the Uni A flow with the keyboard only
- Upload `roster-sample.csv`, read the errors, fix, then upload `roster-sample-clean.csv`
- As Uni C, confirm every write action is disabled until you click **Simulate admin approval**
- Export a CSV and open it in a spreadsheet; confirm no cell is interpreted as a formula
- Turn on "reduce motion" in your OS: the funnel and Proof Ring should show final states instantly
