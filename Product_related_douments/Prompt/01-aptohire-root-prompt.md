# PROMPT 1 of 5 — `aptohire.com` (Root website)

> Paste everything below this line into your AI coding tool (Claude Code, Cursor, etc.) inside an **empty folder** named `aptohire-root/`.
> This project is **independent**. It does not contain, import from, or share code with the other four portals.

---

## 0. Your role and how to work

You are a senior frontend engineer and product designer. Build the **public root website of AptoHire** (`aptohire.com`) as a complete, production-quality Vite + React + TypeScript project.

Work in this order and do not skip steps:

1. **Plan (short).** Write `docs/PLAN.md` (max 1 page): file tree, route list, component list, anything you will mock. Then start building.
2. **Build** in the phases of section 15. After each phase run `npm run typecheck && npm run lint && npm run test` and fix failures before moving on.
3. **Verify** against the acceptance checklist (section 16). Open the site at 360, 390, 768, 1024 and 1440 px and fix issues.
4. **Report** using the exact format in section 17.

Rules for decisions:

- Do not ask me questions. If something is unclear, choose the sensible default, leave a `// TODO(decision): …` comment, and list it in the final report.
- Never invent facts about the product. Every claim on the site must come from section 1 or section 7. **No fake testimonials, no fake customer logos, no invented statistics, no invented prices.** Anything shown as a sample must be labelled "Sample".
- All people, universities, organizations and data shown in the UI are **fictional**.

---

## 1. Product context (what you are marketing)

**AptoHire — "Hired for what you can do."**

AptoHire is a skill-verification and hiring platform. Students prove their skills by building real projects. An AI reviews each project against a rubric and issues a **verified skill scorecard** with evidence links. Organizations discover verified talent **without seeing names** until the student says yes. Universities get a dashboard showing how their students are doing.

The three headline capabilities:

1. **Project review with evidence.** Students submit a real project (code, documents, a 3-minute demo video). The review scores each skill and links every score to the evidence behind it. Skills scoring **above 50** become **Verified**.
2. **A living skill scorecard.** One permanent profile per student that follows them across universities, graduation, dropout or further study, with no double entry. It produces a downloadable digital resume with a QR code that leads to a public verification page.
3. **Blind, consent-first matching.** Organizations post a job and see anonymous, ranked candidates with "why they match". They request access; the student accepts or declines; identity is revealed only after the student accepts, and can be withdrawn at any time.

Levels: **Emerging** (0–50, not verified) · **Competent** (51–69) · **Proficient** (70–84) · **Advanced** (85–100).

Audiences: **Students** (tech and management, Tier-2 and Tier-3 colleges are the focus), **Universities** (placement cells / TPOs), **Organizations** (verified recruiters), plus AptoHire staff (not visible on this site).

This is a **prototype**. In non-production environments the site must say so (section 8.9).

---

## 2. Where this project fits (read carefully)

AptoHire is **five separate web projects** sharing **one Supabase database**:

| Project folder | Host | Who it is for |
|---|---|---|
| **`aptohire-root/` ← YOU ARE BUILDING THIS** | `aptohire.com` | Everyone: marketing, pricing, chooser, public credential verification, legal |
| `aptohire-student/` | `student.aptohire.com` | Students |
| `aptohire-university/` | `university.aptohire.com` | University staff |
| `aptohire-organization/` | `organization.aptohire.com` | Recruiters |
| `aptohire-admin/` | `admin.aptohire.com` | AptoHire staff |

A sixth shared folder, `aptohire-backend/`, owns the Supabase migrations, RLS policies, seed data and Edge Functions. **This project does not own or edit any of that.**

### What this project does and does not do

| It does | It does not |
|---|---|
| Static marketing pages, pricing, FAQ, legal pages | Log anyone in. **There is no auth and no session on the root site** |
| Link visitors to the correct portal's own login/sign-up | Link to or mention the admin portal anywhere |
| Show the public **credential verification page** (`/verify/:credentialId`) by calling an Edge Function | Query database tables directly |
| Accept **university access applications** and **organization access requests** through an Edge Function | Hold the Supabase service-role key (it must never exist in this repo) |

Because the root site never touches tables directly, **do not install `@supabase/supabase-js`**. Call the two Edge Functions in section 4 with plain `fetch` and the public anon key. This keeps the bundle small and the attack surface minimal.

---

## 3. Tech stack and project setup

| Layer | Choice |
|---|---|
| Build | **Vite + React 18 + TypeScript (strict, no `any`)** |
| Routing | React Router (data router). Home route is eager; all other routes `React.lazy` |
| Styling | **Tailwind CSS 3.4 (pinned)** with tokens mapped in `theme.extend`, plus CSS variables in `src/styles/tokens.css`. Use Tailwind 3.4 so all five portals stay identical |
| UI primitives | shadcn/ui (Radix) restyled to the tokens in section 5. Install only the primitives you use (accordion, dialog, popover, sheet, tabs, toast, tooltip) |
| Forms | React Hook Form + **Zod** (`@hookform/resolvers`) |
| Icons | `lucide-react`, stroke 1.75, 20 px default |
| Fonts | **Self-host with Fontsource**: `@fontsource-variable/bricolage-grotesque`, `@fontsource-variable/instrument-sans`, `@fontsource/jetbrains-mono` (400 only). Import only the weights used. If a package is unavailable, fall back to Google Fonts with `preconnect` and `display=swap` |
| Head/SEO | `react-helmet-async` |
| Motion | No animation library. Use CSS transitions, SVG, and one tiny `useCountUp` hook (rAF). |
| Tests | Vitest + React Testing Library, Playwright + `@axe-core/playwright` |
| Lint | ESLint (typescript-eslint, react-hooks, jsx-a11y) + Prettier |

### Ports for local development

Each project uses its own port so all five can run together. **Root = `5173`.** (Student 5174, University 5175, Organization 5176, Admin 5177.)

### Scripts (`package.json`)

`dev`, `build`, `preview`, `typecheck`, `lint`, `test`, `e2e`, `lighthouse` (optional script that runs Lighthouse CI against `preview`).

### Environment variables (`.env.example`, committed; `.env.local`, ignored)

```
VITE_APP_ENV=local                 # local | demo | production
VITE_DEMO_MODE=true                # shows /demo page + prototype notice; must be false in production
VITE_USE_MOCKS=true                # true = never call the network; use fixtures (section 4.4)
VITE_SITE_URL=http://localhost:5173
VITE_ROOT_DOMAIN=localhost
VITE_SUPABASE_URL=                 # https://<ref>.supabase.co  (Edge Functions base)
VITE_SUPABASE_ANON_KEY=            # public anon/publishable key ONLY
VITE_STUDENT_URL=http://localhost:5174
VITE_UNIVERSITY_URL=http://localhost:5175
VITE_ORGANIZATION_URL=http://localhost:5176
VITE_TURNSTILE_SITE_KEY=           # optional Cloudflare Turnstile (free). Empty = no captcha
VITE_SUPPORT_EMAIL=support@aptohire.test
```

`src/config/env.ts` parses these with Zod at startup and throws a readable error if invalid. **Only `VITE_*` values may ever be read.** In production, if `VITE_USE_MOCKS=true` or `VITE_DEMO_MODE=true`, fail the build.

`src/config/portals.ts` exports `portalUrl(portal: 'student' | 'university' | 'organization', path = '/')`. It uses the explicit `VITE_*_URL` if set, otherwise `https://{portal}.${VITE_ROOT_DOMAIN}`. **No other code builds portal URLs.** There is deliberately no `admin` option.

### Folder structure

```
aptohire-root/
├─ README.md                    # plain-language setup (same tone as a beginner guide)
├─ .env.example
├─ index.html                   # viewport meta, theme-color, preload fonts, no inline scripts
├─ package.json  vite.config.ts  tsconfig.json  tailwind.config.ts  postcss.config.js
├─ vercel.json                  # rewrites + security headers (section 11)
├─ public/                      # favicon.svg, og-image.png, robots.txt, sitemap.xml, site.webmanifest
├─ docs/                        # PLAN.md and a copy of the design rules in this prompt
├─ src/
│  ├─ main.tsx
│  ├─ app/                      # router.tsx, providers.tsx, ErrorBoundary.tsx, ScrollToTop.tsx
│  ├─ config/                   # env.ts, portals.ts
│  ├─ pages/                    # Home, Pricing, GetStarted, ApplyUniversity, ApplyOrganization,
│  │                            # VerifyLookup, VerifyCredential, Privacy, Terms, AiFairness, Demo, NotFound
│  ├─ sections/home/            # Hero, HowItWorks, ForStudents, ForUniversities, ForOrganizations,
│  │                            # PrivacyPromise, Faq, ClosingCta
│  ├─ components/
│  │  ├─ layout/                # Header, MobileMenu, Footer, PrototypeNotice, NoticeBar, SkipLink, PageShell
│  │  ├─ brand/                 # Logo, SealMark
│  │  ├─ proof/                 # ProofRing, ScorecardPanel, EvidenceRow, EvidenceChip, StatusChip,
│  │  │                         # AliasGlyph, CandidateCardPreview, SkillGraphPreview
│  │  └─ ui/                    # Button, Input, Textarea, Select, Checkbox, Field, Toast, Accordion, ...
│  ├─ content/                  # copy.ts, sample.ts, pricing.ts, faq.ts, legal/*.tsx
│  ├─ lib/
│  │  ├─ api.ts                 # typed Edge Function client (section 4)
│  │  ├─ schemas/               # zod: application.ts, credential.ts, errors.ts
│  │  ├─ scoring.ts             # PASS_MARK, levelFor(score), isVerified(score)
│  │  ├─ mocks/                 # fixtures + mock handlers
│  │  ├─ seo.tsx  format.ts  cn.ts  hash.ts
│  └─ styles/                   # tokens.css, globals.css
├─ tests/                       # vitest
└─ e2e/                         # playwright
```

`PASS_MARK = 50` lives only in `lib/scoring.ts` with a comment: *"Mirror of app_config.PASS_MARK in the shared database. Keep in sync."* The verify page should prefer `verified` and `level` returned by the API and use `scoring.ts` only for sample content.

---

## 4. Shared-database contract (the only two network calls)

Base URL: `${VITE_SUPABASE_URL}/functions/v1`. Every request sends `apikey: <anon key>` and `Authorization: Bearer <anon key>`, `Content-Type: application/json`. These Edge Functions are built in `aptohire-backend/`. **You build the client against the contract below and a full mock layer** so this project works before the backend exists.

### 4.1 `public-verify` (anonymous, rate-limited, cached)

`GET /public-verify?id=<credentialId>`

`credentialId` is **18 lowercase hex characters** (`/^[a-f0-9]{18}$/`). Validate on the client before calling; invalid format → show the "not found" state without a network call.

Success `200`:

```jsonc
{
  "credential": {
    "credential_id": "a1b2c3d4e5f6a7b8c9",
    "issued_at": "2026-10-12",            // date of the first verified skill
    "last_updated": "2026-10-12",
    "display_name": "Aarav S.",           // OPTIONAL: present only if the student allowed it
    "affiliation": {                      // OPTIONAL
      "label": "Uni A",                   // university name, only if allowed
      "verification": "roster"            // "roster" | "student_id" | "unverified"
    },
    "skills": [                           // may be empty
      { "name": "React", "level": "Proficient", "verified": true,
        "score": 72,                      // OPTIONAL: only if the student allowed scores
        "verified_on": "2026-10-12" }
    ],
    "shared_fields": ["skills", "levels"] // what the student chose to show
  }
}
```

Errors (shape `{ "error": { "code": "...", "message": "..." } }`): `404 NOT_FOUND`, `429 RATE_LIMITED` (with `Retry-After`), `500/503 SERVICE_UNAVAILABLE`.

**The page must render only what the API returned.** Never assume `display_name`, `score` or `affiliation` exist. Never show unverified skills as verified. Parse every response with Zod (`lib/schemas/credential.ts`, `.strict()` off, unknown fields dropped).

### 4.2 `submit-application` (anonymous, rate-limited, validated)

`POST /submit-application`. Body is a discriminated union on `type`.

```ts
// university
{
  type: 'university',
  institution_name: string,            // 2–120
  short_name?: string,                 // ≤ 30
  kind: 'university' | 'college' | 'school',
  tier: 'tier1' | 'tier2' | 'tier3' | 'other',
  city: string, state: string,
  email_domains?: string[],            // e.g. ["unia.edu.test"], validated as domains
  student_count_band: '<500' | '500-2000' | '2000-5000' | '5000+',
  departments?: string,                // ≤ 200
  contact_name: string, contact_role: string,
  contact_email: string, contact_phone?: string,
  message?: string,                    // ≤ 1000
  consent: true,                       // literal true, required
  hp: string,                          // honeypot, must be ''
  captcha_token?: string,              // only when VITE_TURNSTILE_SITE_KEY is set
}
// organization
{
  type: 'organization',
  org_name: string,                    // 2–120
  org_kind: 'corporate' | 'startup' | 'agency' | 'other',
  size_band: '1-10' | '11-50' | '51-250' | '251-1000' | '1000+',
  website: string,                     // https URL
  hiring_focus: Array<'web_frontend' | 'backend' | 'data' | 'management_commerce' | 'other'>,
  contact_name: string, contact_role: string,
  work_email: string,                  // reject common free-mail domains with a friendly message
  message?: string,
  consent: true, hp: string, captcha_token?: string,
}
```

Headers: `Idempotency-Key: <crypto.randomUUID()>` (one per form instance, regenerated only after a successful submit).

Success `202`: `{ "reference": "AH-7K2Q9F", "status": "received" }`.

Errors: `422 VALIDATION_FAILED` with `{ error: { code, message, fields?: Record<string,string> } }` (map `fields` onto form fields), `429 RATE_LIMITED`, `503 SERVICE_UNAVAILABLE`.

> **Backend responsibilities (not built here, noted for `aptohire-backend`):** create a `pending` university/organization application from this payload, verify captcha if present, rate-limit by IP and email, never create a login, and let the admin portal approve or reject it.

### 4.3 What the visitor sees after a successful application

A confirmation panel (not a toast) with: the reference number in JetBrains Mono with a copy button, what happens next in 3 plain steps ("We review your application · You get an email when it is approved · Then you can sign in"), and a secondary button. `// TODO(decision):` for universities the secondary button is **"Create your university account"** linking to `portalUrl('university', '/signup?ref=AH-…')`; for organizations it is **"Back to home"**.

### 4.4 Mock layer (required)

`lib/api.ts` must switch to `lib/mocks/*` when `VITE_USE_MOCKS=true`, with a realistic 400–900 ms delay. Fixtures for `public-verify`, keyed by id:

| Id | Behavior |
|---|---|
| `a1b2c3d4e5f6a7b8c9` | Full: display name, university (roster), three verified skills with scores |
| `b2c3d4e5f6a7b8c9d0` | Levels only: no name, no scores, no university |
| `c3d4e5f6a7b8c9d0e1` | Student shares nothing: empty `skills` (render the "nothing shared" state) |
| `d4e5f6a7b8c9d0e1f2` | `unverified` affiliation (must show the dashed "Unverified university" chip and the note that scores are not affected) |
| `e5f6a7b8c9d0e1f2a3` | Always `429` |
| any other valid hex | `404` |

Mock `submit-application`: succeeds with a generated reference; email `fail@demo.test` returns `422` with a field error; `rate@demo.test` returns `429`.

Write Vitest tests for the client (success, each error code, network failure, malformed JSON, Zod rejection of a bad payload).

---

## 5. Design system (follow exactly; the other four portals use the same one)

**Concept: "Proof you can see."** Quiet, white, spacious, so one thing stands out: **evidence**. Green means verified and trusted. Orange means in progress, attention and energy. They meet in the **Proof Ring**. White is the product. Spend boldness in one place: the Proof Ring and the evidence trail beside it. Everything else is calm and disciplined.

### 5.1 Tokens → `src/styles/tokens.css` (and mirrored in `tailwind.config.ts`)

```css
:root {
  --canvas:#ffffff; --mist:#f6f8f7; --hairline:#e3e8e5; --ink:#17261e; --moss:#5b6a62;

  --verdant-50:#ecfaf2; --verdant-100:#d2f3e1; --verdant-300:#8fe0b2;
  --verdant-600:#0e9f5b; --verdant-700:#0b7f49; --verdant-800:#0a6139;

  --ember-50:#fff3e8; --ember-100:#ffe2c7; --ember-300:#ffb470;
  --ember-500:#ff7a1a; --ember-700:#b94a06;

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

  --ease-out:cubic-bezier(.2,.8,.2,1);
  --dur-fast:150ms; --dur-base:220ms; --dur-reveal:900ms;
}
@media (prefers-reduced-motion: reduce){ *{animation-duration:.001ms!important;transition-duration:.001ms!important;scroll-behavior:auto!important} }
```

Tailwind names: colors `canvas, mist, hairline, ink, moss, verdant.{50,100,300,600,700,800}, ember.{50,100,300,500,700}, danger.{50,600}`; `borderRadius`: `control, panel, feature`; `boxShadow`: `float` (shadow-1), `modal` (shadow-2); `fontFamily`: `display, body, mono`. **No hard-coded hex values in components.** Light theme only (no dark mode).

### 5.2 Colour rules (WCAG, verified ratios)

- Primary button: **white on `verdant-700`** (5.1:1). Accent button: **`ink` on `ember-500`** (6.0:1). **Never white on `ember-500`.**
- `verdant-600` and `ember-500` are for **graphics and large elements only** (≥ 24 px text), never small text. Small green text = `verdant-700`; small orange text = `ember-700`.
- Secondary text = `moss` (5.7:1). Colour is never the only signal: pair every green/orange state with an icon or word.
- **Gradients:** `grad-mesh` behind the hero only. At most **one gradient surface per viewport**. No gradient on buttons, cards, or behind text smaller than 16 px. The Proof Ring arc is the one other gradient.
- Root portal identity: `grad-mesh` hero, both accents in play, warm and open. Primary buttons are always `verdant-700`.

### 5.3 Typography

| Role | Face | Notes |
|---|---|---|
| Display + headings | **Bricolage Grotesque** variable | weights 600–700, tight tracking |
| Body + UI | **Instrument Sans** | weights 400/500/600 |
| Code, credential IDs | **JetBrains Mono** | only for IDs and technical strings, never decorative labels |

Fluid scale (reconciles the design system with the mobile minimums):

| Token | Size | Line-height | Tracking | Use |
|---|---|---|---|---|
| `display-xl` | `clamp(36px, 6vw, 64px)` | 1.06 | −0.025em | Home hero only |
| `display-l` | `clamp(28px, 4.2vw, 44px)` | 1.14 | −0.02em | Section headlines (H1 on inner pages uses `h1`) |
| `h1` | `clamp(28px, 4vw, 40px)` | 1.2 | −0.015em | Inner-page titles |
| `h2` | `clamp(24px, 3vw, 32px)` | 1.3 | −0.01em | |
| `h3` | `clamp(20px, 2.4vw, 24px)` | 1.35 | 0 | |
| `body-l` | `clamp(17px, 1.6vw, 19px)` | 1.6 | 0 | Lead paragraphs |
| `body` | 16px | 1.6 | 0 | Default (never below 16 px on mobile) |
| `small` | 14px | 1.5 | 0 | Metadata, table cells, helper text (**14 px minimum everywhere**) |

Rules: **sentence case everywhere**, no all-caps tracked labels, no eyebrow labels unless they carry information; line length ≤ 68 characters; text left-aligned (centre only for single-line empty states and the closing call to action); never justified; `font-variant-numeric: tabular-nums` on numbers; scores in Bricolage 600. Don't accent a single word in a headline: set whole headlines in `ink`.

### 5.4 Layout, shape, elevation

- Spacing scale (4 px base): 4 · 8 · 12 · 16 · 24 · 32 · 48 · 72 · 112. Max content width **1200 px**; page padding 24 px desktop / 16 px mobile; section rhythm **112 px desktop / 72 px mobile**.
- Radius encodes hierarchy: `r-control` 10 (inputs, buttons; chips are full-pill), `r-panel` 16 (candidate cards, dialogs), `r-feature` 28 (hero scorecard panel), `r-full` for pills/seal.
- Most things are **flat with a hairline border**. Shadow only marks what floats: `float` (menus, popovers, card hover), `modal` (dialogs, sheets). Data lives in tables and lists with hairlines. Cards exist only for things you act on (candidates, plans).

### 5.5 Logo and brand assets

- **Logo:** wordmark "AptoHire" in Bricolage 700 plus a small seal mark (a circle with a check whose stroke ends in an orange dot). Build as inline SVG components `Logo` and `SealMark` (versions for white and for `verdant-700` backgrounds).
- **Favicon:** the seal mark on white (`public/favicon.svg`). **OG image (1200×630):** the hero scorecard composition on the mesh wash. Generate it as a static PNG in `public/og-image.png` (draw it in SVG and rasterize with a small script, or hand-write the PNG-ready SVG and note the conversion command in README).
- **Illustrations:** composed only from palette circles/arcs. **No stock photography, no photos of people.**

### 5.6 Motion

Exactly **one orchestrated moment**: the Proof Ring reveal (section 6.1), played once when the hero scorecard first appears. Everything else only responds to user actions (menu open, accordion expand, button press, popover). No fade-up on every section, no hover lift on every card. Reduced motion → final states immediately.

---

## 6. Signature components

### 6.1 `ProofRing` (the memorable element; used on the hero, the verify page, and previews)

Props: `score: number | null`, `skill: string`, `state: 'verified' | 'not-verified' | 'under-review' | 'loading'`, `size: 56 | 96 | 160 | 240`, `level?: string`, `animate?: boolean`, `showCaption?: boolean`, `passMark?: number` (default from `scoring.ts`).

| Property | Spec |
|---|---|
| Track | `hairline`, stroke 12 px at 240 (scale proportionally) |
| Arc | SVG circle, `stroke-linecap: round`, length = score ÷ 100, starting at 12 o'clock going clockwise. Gradient orange → green: `#FF7A1A → #FFB470 (30%) → #8FE0B2 (60%) → #0E9F5B`. A `<linearGradient>` across the ring is acceptable; a true conic effect is optional. Do not sacrifice round caps or the animation |
| Pass notch | Small tick on the track at the pass mark position (50%) |
| Centre | Score in Bricolage 600 (64 px at size 240), caption below with the level |
| `verified` | Seal (check in `grad-verified` circle, 40 px at 240) on the ring top-right; caption "Verified · Proficient" |
| `not-verified` | Arc solid `ember-500`; caption "Not verified yet · 9 points to go" |
| `under-review` | Arc `ember-300` with dashed final segment; caption "Under human review" |
| `loading` | Track only plus a skeleton shimmer (none under reduced motion) |
| Accessibility | `role="img"`, `aria-label="React, score 72 of 100, Proficient, verified"`; the score number is real text |
| Reveal | Arc draws 0 → score over **900 ms** `ease-out` while the number counts up in sync; the seal scales 0.8 → 1 over a 180 ms settle at the end. Plays once, only when it enters the viewport (IntersectionObserver). Reduced motion: final state immediately |

Unit-test the state/caption logic and the aria-label builder.

### 6.2 Evidence trail (`EvidenceRow` + `EvidenceChip`)

Each row = a criterion name, a score bar (`verdant-600` on `mist`), the number, a one-line rationale, and **evidence chips** (`Video 01:42`, `src/App.tsx`, `README.md`, `Live site`). Clicking or pressing a chip opens a **popover** (Radix) with a small sample of that evidence: a code excerpt with line numbers in JetBrains Mono, a video timestamp card ("Jumps to 01:42 in the demo video"), or a README excerpt. These are clearly labelled **Sample evidence**. Structure here is information: the rows exist because the score is explainable.

### 6.3 `StatusChip`

| Chip | Look |
|---|---|
| Verified | `verdant-100` fill, `verdant-800` text, seal icon |
| Under review | `ember-100` fill, `ember-700` text |
| Pending | `ember-50` fill, `ember-700` text |
| Unverified university | white, dashed `hairline` border, `moss` text |
| Verified university | `verdant-50` fill, small seal |
| Declined / Closed | `mist` fill, `moss` text |

### 6.4 `AliasGlyph`

48 px generated mark per alias: 3–4 overlapping circles/arcs from a deterministic seed (hash of the alias string), palette greens and oranges only, on `mist`. No faces, no initials. Same alias → same glyph. Put the hash function in `lib/hash.ts` and unit-test determinism.

### 6.5 `ScorecardPanel`, `CandidateCardPreview`, `SkillGraphPreview`

- `ScorecardPanel`: the hero's real-UI scorecard (ring + level + 3 evidence rows). `r-feature` radius.
- `CandidateCardPreview` (non-interactive sample): alias glyph + alias ("Candidate K7F2"), university **category** chip ("Tier 2 · State university", never a name), match meter (thin `verdant-600` bar on `mist`) + words ("Strong match · High confidence"), top skill chips with levels, one reason line, lock row ("Identity hidden until the student agrees"), buttons *View* (secondary) and *Request access* (accent). Buttons are visibly part of the sample and open a short popover "This is a sample. Organizations send real requests from their own portal." with a link to `/get-started`.
- `SkillGraphPreview`: small SVG with nodes sized by score (verified = `verdant-600` fill + check; in progress = `ember-500` outline; not assessed = dashed hairline), edges 1 px `#D5DDD8`, with a **graph ⇄ list toggle**; list is the default below 640 px. Nodes are keyboard-focusable and highlight neighbours.

---

## 7. Pages, content and layout

All copy uses this vocabulary consistently: **Project · Submit · Review · Scorecard · Verified · Evidence · Request access · Accept · Decline · Sharing · Withdraw access.** Do not use "audit engine", "agent" or "pipeline" in visitor-facing text. Voice: plain, warm, specific, active verbs, no hype words ("revolutionary", "AI-powered" as a headline). Errors state what happened and how to fix it; they don't apologize. Put all strings in `src/content/*` so they are easy to edit.

Routes: `/` · `/pricing` · `/get-started` (also `/login`, `/signup` redirect here with an `intent`) · `/apply/university` · `/apply/organization` · `/verify` · `/verify/:credentialId` · `/privacy` · `/terms` · `/ai-and-fairness` · `/demo` (only when `VITE_DEMO_MODE=true`) · `*` (404).

### 7.1 Global header (`Header`, `MobileMenu`)

- Sticky, white (a solid or very light blur surface, **no gradient**), hairline bottom border, height 64 px. Skip-to-content link is the first focusable element.
- Left: `Logo`. Centre/right nav: **How it works · For students · Universities · Organizations · Pricing** (anchors on Home, routes elsewhere). Right: **Log in** (a popover on desktop listing *Student · University · Organization*, each linking to that portal's `/login`) and a primary button **Get started** → `/get-started`.
- **Hamburger menu (required, built now):** below **768 px** show a ☰ icon button (44×44 px minimum, `aria-expanded`, `aria-controls`, accessible label). Desktop links are hidden below `md`. Tapping toggles a **full-width panel** with every nav link, then a "Log in" group with the three portal links, then the primary button. Behaviors: ✕ replaces ☰ while open; tapping any link closes it; tapping outside closes it; **Escape closes it and returns focus to the button**; smooth open/close transition (height + opacity, 220 ms); focus is trapped while open; body scroll is locked while open; the menu closes on route change and on resize ≥ 768 px.
- Never link to or mention the admin portal.

### 7.2 Home `/` (this is the page that sells the product)

**Hero** (mesh wash behind; text left-aligned; the scorecard is **real UI, not a screenshot**):

```
┌──────────────────────────────────────────────────────────────────────┐
│  Hired for what you can do.            ┌───────────────────────────┐ │
│                                        │   Sample scorecard        │ │
│  Build a real project, get it          │      ◯ 72  Proof Ring     │ │
│  reviewed in minutes, and carry a      │   React · Verified        │ │
│  skill scorecard employers can check.  │ ─ Correctness     ▇▇▇▇▇▁  │ │
│                                        │   [README.md] [Video 1:42]│ │
│  [ Start as a student ]  [ I hire ]    │ ─ Code quality    ▇▇▇▇▁▁  │ │
│  Free for students.                    │ ─ Communication   ▇▇▇▇▇▁  │ │
│                                        └───────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

- H1 (display-xl, whole line in `ink`): **"Hired for what you can do."** Lead (body-l, moss, ≤ 2 sentences) as above.
- CTAs: primary **Start as a student** → `portalUrl('student', '/signup')`; secondary **I hire** → `/get-started?intent=organization`. Under them, a one-line reassurance in `small`: "Free for students. Employers see skills, not names, until you say yes."
- Sample scorecard content (fixture in `content/sample.ts`): React, **72**, **Proficient**, Verified; Correctness 74 ("Works as described; one edge case with an empty cart."), Code quality 66, Communication 78; chips `Live site`, `src/cart.tsx`, `README.md`, `Video 01:42`. Ring reveal plays once on load. Below 640 px the ring is 160 px and the evidence rows stack under it.
- Hero must render (text + final ring state) **without JavaScript animation** and reach first contentful paint under 2 s on a mid-range phone over 4G.

**Below the hero, in this order** (each with a clearly different layout; do not repeat the same card grid):

1. **How it works** — a true sequence, so numbering is allowed: **Build → Review → Verify → Get found.** Horizontal steps joined by a hairline on desktop, vertical on mobile. Each step: a small illustration made of palette circles/arcs, a heading, one sentence.
   - Build: "Pick a skill and get a real-world project at your level, or upload one you already built."
   - Review: "Submit your code, files and a 3-minute demo. Every score comes with the evidence behind it."
   - Verify: "Skills above 50 become Verified. Not happy with a result? Raise a ticket and a different reviewer decides."
   - Get found: "Organizations find your skills, not your name. You choose who gets to see more."
2. **For students** — split layout: left copy and three short points (project at your level; evidence-linked scorecard; one permanent profile that follows you across universities, plus a downloadable resume with a QR code); right: `SkillGraphPreview` (graph ⇄ list) and a small resume-with-QR illustration. CTA: **Create your profile**.
   - Include the line: "Your university isn't on AptoHire yet? Your scores and job matches are not affected."
3. **For universities** — wide, table-led layout: a **sample roster table** (fictional Uni A: Aman · incoming; Ajay · year 1; Aarav · year 2; Akshit · year 3; Aarti · dropout; Avi · alumnus) with columns *Student, Status, Verification, Verified skills, Placement* (placement values only **Placed / Not placed / Unknown**), labelled "Sample data". Under it, a plain stat row with hairline dividers (no cards) and an **Unmatched demand** list showing skill, number of jobs, and reason counts, **never names**. Copy: "See how your students are doing, which skills employers ask for that you don't yet supply, and where placements stand." CTAs: **Apply for university access** → `/apply/university`; secondary link "University sign in".
4. **For organizations** — `CandidateCardPreview` list (3 sample candidates with different aliases, e.g. K7F2, M3Q8, T9R4, and different match words: *Strong match*, *Strong match*, *Good match*) beside copy: "Post a job, read the skills it needs, and see ranked candidates with the reasons they match. Ask for access when you're ready; the student decides." CTAs: **Request organization access** → `/apply/organization`; secondary "Organization sign in".
5. **Privacy promise** — a short, calm band (white, hairline top/bottom): headline **"Employers see skills, not names, until you say yes."** with a three-step diagram: **Request → You decide → Reveal**, and a line "Withdraw access any time from Sharing." Use this section to describe the behaviour precisely: before you accept, organizations don't receive your name, email, photo, university name or files. Link to `/ai-and-fairness` and `/privacy`.
   - `// TODO(decision):` the product brand phrase for blind matching is "zero-knowledge anonymized matchmaking". Keep it as one editable constant `BRAND_BLIND_MATCHING_LABEL` in `content/copy.ts`, used at most once on the page (as a feature label), always followed by the plain-language explanation above.
6. **FAQ** — Radix accordion (one open at a time, keyboard accessible). Questions and grounded answers only:
   - *What makes a skill Verified?* A review score above 50. Scores from 40 to 60 are also checked by a person before they are shared with organizations.
   - *Who sees my demo video and files?* Only you, plus reviewers when needed. Organizations never see raw files before you accept a request, and only the parts you choose to share afterwards.
   - *What if I disagree with my score?* Raise a ticket from the scorecard. A different reviewer looks at it and records the decision.
   - *Do I need my university to be on AptoHire?* No. You can register with a university email, with a student ID check, or without either. Scores and job access are the same.
   - *Does AI decide everything?* AI does the first review. Identity, consent, permissions and verification status follow fixed rules, and borderline results go to a person. Read the AI and fairness statement.
   - *Is it free?* Free for students. See pricing for universities and organizations.
   - *Is this real data?* This is a prototype. All profiles, universities and organizations shown are fictional. (Show only when not production.)
7. **Closing call to action** (the only centred block) — "Show what you can do." Three buttons: **I'm a student** (primary), **I'm a university**, **I'm hiring**.

**Footer:** logo, one-line description, columns *Product* (How it works, Pricing, Verify a credential), *Portals* (Student, University, Organization — **no admin**), *Legal* (Privacy, Terms, AI and fairness), support email, "© AptoHire". No social icons unless real URLs are supplied via env (don't add placeholders).

### 7.3 Pricing `/pricing` (static; do not invent figures)

Three plan panels with different emphasis (not three identical cards): **Students — Free**; **Universities — Pilot (free) then "Talk to us"**, includes roster upload, student list, analytics, placement view, unmatched-demand alerts; **Organizations — Pilot (free for early partners) then "Talk to us"**, includes job posts, ranked blind candidates, access requests, pipeline. Prices come from `content/pricing.ts`; use `price: null` to render "Talk to us". `// TODO(decision):` real prices. Add a small comparison table (rows = capabilities, columns = the three audiences; checks always paired with the word "Included") that scrolls horizontally inside its own container on mobile. CTA per plan to the matching `/apply/...` or student sign-up. Footer note: "AptoHire is a prototype. Plans shown here are not final."

### 7.4 Get started `/get-started`

"Where do you want to start?" Three panels (student, university, organization), each with: who it is for in one sentence, **Sign in** (portal `/login`) and the right create path — Student → portal `/signup`; University → **Apply for access** (`/apply/university`); Organization → **Request access** (`/apply/organization`). `?intent=` highlights and focuses the matching panel. Never display admin.

### 7.5 Apply for university access `/apply/university` and organization access `/apply/organization`

- Two-column on desktop (form left, "What happens next" and a privacy note right), single column on mobile with the explainer first and collapsed into a short list.
- Use the fields from section 4.2 grouped into logical fieldsets (Institution, Contact, Anything else). Labels always visible **above** inputs, helper text 14 px, required marked in words ("required"), errors with an icon, linked through `aria-describedby`, announced with `aria-live="polite"`, focus moves to the first invalid field on submit.
- Consent checkbox text: "I agree to AptoHire contacting me about this application and to the privacy notice." (link). The honeypot field `hp` is visually hidden, `tabIndex={-1}`, `autoComplete="off"`, `aria-hidden`. Add a client-side minimum fill time of 3 seconds before accepting a submit (quietly, no message needed beyond normal validation).
- Submit button label: **Send application** (university) / **Request access** (organization). While submitting, keep the label and fixed width, show a spinner, and disable repeat submits. On success show the confirmation panel (4.3). On `429` show "Too many attempts. Try again in a few minutes." On `503` show "We couldn't send this right now. Your answers are still here. Try again." and keep the form values.
- If `VITE_TURNSTILE_SITE_KEY` is set, load the Turnstile widget lazily (only on these two pages) and attach `captcha_token`; if empty, skip it silently.
- Do not store form values in localStorage or log them.

### 7.6 Public verification `/verify` and `/verify/:credentialId`

- `/verify`: a short page, **"Check a credential"**, one input labelled "Credential ID" (JetBrains Mono, paste friendly, trims whitespace, accepts a full pasted `/verify/...` URL and extracts the ID), button **Check credential**. Helper: "Find the ID under the QR code on a student's resume." Invalid format → inline error "Credential IDs are 18 characters, 0–9 and a–f."
- `/verify/:credentialId` (this is the page a QR code opens; design it for **phones first**). States, each fully designed:
  1. **Loading:** skeleton matching the final layout.
  2. **Verified (normal):** header strip with the seal and "Verified on AptoHire", the credential ID in mono with a copy button, issued and last-updated dates; the student's display name if returned (otherwise "A verified AptoHire student"); the affiliation chip if returned; then a list of skills, each as a row with a 96 px `ProofRing` (when `score` is returned) or a level chip (when not), the skill name, level, "Verified on 12 Oct 2026". Below: **"How to read this"** (3 short lines: what Verified means, what the levels are, that scores come from reviewed projects with evidence) and a plain statement of what the student chose to share ("This student shares: skills and levels").
  3. **Nothing shared:** empty `skills` → "This student hasn't shared any skills on this page." with the credential ID still shown.
  4. **Unverified university:** dashed chip plus "This university isn't confirmed on AptoHire yet. Skill scores are not affected."
  5. **Not found / invalid ID:** same generic message for both: "We couldn't find this credential. Check the ID and try again." with a link to `/verify`. Never reveal whether an ID format was valid.
  6. **Rate limited:** "Too many checks from this connection. Try again in a minute." (use `Retry-After` if present.)
  7. **Service unavailable:** error with a **Try again** button.
- Page head: `noindex, nofollow`; this route must send `Referrer-Policy: no-referrer` (section 11). Print stylesheet: clean single-column, no nav, shows the credential ID and URL.
- Ring reveal plays once per page load for the first skill only; rows below render statically.

### 7.7 Legal pages `/privacy`, `/terms`, `/ai-and-fairness`

Long-form reading layout: single column ≤ 68 characters, a sticky "On this page" list on desktop that becomes a collapsible list on mobile, headings with anchor links. Each page shows a quiet banner: **"Prototype draft. Not yet reviewed by a lawyer."** `// TODO(decision):` swap for approved text later (keep text in `content/legal/*.tsx`).

- **Privacy:** what the site collects (this root site sets no tracking cookies and uses no analytics; forms send only what you type), how applications are used, student data principles (minimal data, student ID numbers are stored hashed, no ID card images, video and files are private, organizations see nothing identifying until a student accepts, access can be withdrawn), retention in plain words (drafts are deleted after 30 days; raw videos after 90 days while scores and evidence pointers remain), the fact that demo environments contain only fictional data, contact email.
- **Terms:** short, plain-language prototype terms (acceptable use, accuracy of fictional demo data, no guarantees of outcomes, "scores are a review of submitted work, not a guarantee of ability or of being hired").
- **AI and fairness:** how reviews work in plain words: the work (code, running app, documents) is judged first; the demo video is supporting evidence with a capped weight; **accent, speaking pace, fluency, camera quality and background are not scored**; disclosing AI use is not penalised; missing evidence lowers confidence instead of lowering the score; scores from 40 to 60 are checked by a person; students can appeal; consent, identity and permissions are rule-based, not AI.

### 7.8 404 and error pages

Calm 404 with the seal mark, one sentence ("That page doesn't exist."), and buttons **Go home** and **Check a credential**. Global `ErrorBoundary` with a **Reload page** button; never show stack traces.

### 7.9 `/demo` and prototype notice (only when `VITE_DEMO_MODE=true`)

- A slim `PrototypeNotice` bar above the header on every page: "Prototype. All people, universities and organizations shown are fictional." Dismissible per session (`sessionStorage`, wrapped in try/catch).
- `/demo`: "Demo guide" page for presentations. Shows the **5–7 minute demo script** (8 steps below), links that open each portal in a new tab, the demo account emails (**emails only, never passwords**; say "The shared demo password is in the project's .env.example"), and the sample credential IDs from section 4.4 as clickable links. This page and its code must be **tree-shaken/excluded from production builds** (dynamic import behind `import.meta.env.VITE_DEMO_MODE === 'true'`, plus a build assertion).

Demo script: (1) Admin approves University C. (2) Aarav (Uni A) signs in on the student site, asks for a React project. (3) Aarav submits a project and watches the review, then sees the scorecard. (4) Organization A posts a job and sees anonymous ranked cards. (5) Organization A requests access; Aarav accepts; identity is revealed. (6) University A sees analytics and an unmatched-skill alert. (7) Aarav downloads the resume and scans the QR to open the public verification page. (8) Show Avi and Niya: scores stay when they change university.

Demo accounts (fictional): `admin@demo.aptohire.test` (do not link the admin site), `aarav@unia.edu.test`, `naman@demo.aptohire.test`, `avi@demo.aptohire.test`, `tpo@unia.edu.test`, `recruiter@orga.demo.test`, `recruiter@orgb.demo.test`.

### 7.10 Privacy notice bar (F-ID-06)

A small, non-blocking bar at the bottom: "This site doesn't use tracking cookies. It remembers only whether you dismissed this notice." **Got it** button, link to `/privacy`. Stores `aptohire-notice-v1` in `localStorage` (try/catch; render correctly when storage fails). Must not overlap the mobile menu or cover form submit buttons (add bottom safe-area padding; hide while a form field is focused on mobile).

---

## 8. Quality bars

### 8.1 Mobile responsiveness (treat as built-in requirements; do not leave for later)

Breakpoints: **Mobile < 768 (base styles) · Tablet 768–1023 · Desktop ≥ 1024.** In Tailwind: `md` = 768, `lg` = 1024, `xl` = 1280; `sm` (640) may be used for small refinements.

- `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">` in `index.html`; keep fixed/sticky elements clear of notches with `env(safe-area-inset-*)`.
- **No horizontal scroll at any width from 360 px up.** Fix root causes (no fixed widths, no negative margins that overflow); never use `overflow-x: hidden` on `body` as a patch. Wide content (tables, code, diagrams) scrolls **inside its own** `overflow-x: auto` container.
- Multi-column layouts collapse to one column below 768 px. Flex rows wrap or stack. Section padding on mobile ≥ 40 px top/bottom (design rhythm is 72 px; never less than 40). Nothing touches the screen edge; panels have internal padding.
- Body text ≥ 16 px; small text ≥ 14 px; line-height ≥ 1.4; headings use the fluid tokens in 5.3.
- **Touch targets ≥ 44×44 px** for every button, link used as a button, icon-only button, accordion trigger, checkbox/radio hit area and chip that is clickable.
- **Forms:** inputs 100% width on mobile, **font-size ≥ 16 px** (prevents iOS zoom), labels above inputs, form buttons full width on mobile, correct `inputMode`, `autocomplete` and `type` attributes (`email`, `tel`, `url`).
- Images and SVG: `max-width: 100%; height: auto`; set `width`/`height` or `viewBox` to avoid layout shift. There is no photography.
- Tables: scroll inside a wrapper, or convert to a labelled card list below 640 px using `data-label` (do this for the sample roster on Home).
- Verify at **360, 375, 390, 768, 1024, 1440 px**.

### 8.2 Accessibility (WCAG 2.2 AA target)

Landmarks (`header`, `nav`, `main`, `footer`), one `h1` per page, logical heading order, skip link, visible **2 px `verdant-700` focus outline with 2 px offset plus a 4 px white halo** (never remove outlines), all interactive elements keyboard operable, dialogs/popovers/menus trap and restore focus, form errors announced, colour never the only signal, `aria-live="polite"` for async status, reduced-motion respected, `lang="en"`, meaningful `alt`/`aria-label` (decorative SVGs `aria-hidden`). `@axe-core/playwright` runs on every route with **zero serious or critical violations**.

### 8.3 Performance

Targets on the built site (mobile, throttled): **Lighthouse ≥ 95 for Performance, Accessibility, Best Practices and SEO**; LCP < 2.5 s, CLS < 0.05, TBT < 200 ms; initial JS for `/` ≤ **170 KB gzipped**. Techniques: code-split every non-home route, lazy-load below-the-fold sections and the Turnstile script, preload only the two font files above the fold, `font-display: swap`, no layout shift from fonts (use `size-adjust` fallbacks or accept swap), SVG only, no third-party scripts by default.

### 8.4 Browser support

Latest two versions of Chrome, Edge, Safari, Firefox.

---

## 9. SEO and sharing

- Per-page `<title>` and meta description via `react-helmet-async` (Home: "AptoHire — Hired for what you can do"), canonical URLs built from `VITE_SITE_URL`, Open Graph and Twitter card tags using `/og-image.png`, `theme-color` `#0B7F49`, `site.webmanifest`.
- `public/robots.txt`: allow all except `/verify/` and `/demo`; reference the sitemap. `public/sitemap.xml`: `/`, `/pricing`, `/get-started`, `/apply/university`, `/apply/organization`, `/verify`, `/privacy`, `/terms`, `/ai-and-fairness` (no credential URLs, no demo).
- JSON-LD `Organization` on Home (name, url, logo only; no invented details).
- **Prerendering:** a single-page app is acceptable for the prototype. `// TODO(decision):` note in README that adding static prerendering for the marketing routes is the next SEO step.

---

## 10. Security and privacy rules (never break)

1. **No secrets in this repo or bundle.** Only `VITE_*` values; never the Supabase service-role/secret key. `.env*` (except `.env.example`) is git-ignored.
2. **Never trust or echo the client:** treat `public-verify` and `submit-application` responses as untrusted input; Zod-parse everything; render API strings as text (React escaping) and never with `dangerouslySetInnerHTML`.
3. **No PII in logs, analytics, error messages, URLs, or storage.** The only storage keys are the notice dismissal and the prototype-notice dismissal.
4. **No third-party scripts** except the optional Turnstile widget (loaded only on the two apply pages when a site key is set).
5. **Verification page leaks nothing:** identical "not found" copy for missing and malformed IDs; honour whatever the API omits; `noindex`; `Referrer-Policy: no-referrer`; no autocomplete suggestions of IDs.
6. **External links** use `rel="noopener noreferrer"`. Links to other portals are built only by `portalUrl()`.
7. **Demo-only code** (`/demo`, prototype notice, mocks) is excluded or disabled in production; the build fails if `VITE_USE_MOCKS` or `VITE_DEMO_MODE` is true when `VITE_APP_ENV=production`.
8. Run `npm audit` and note results in the report; pin dependency versions (exact versions in `package.json`).

---

## 11. Deployment (Vercel, separate project for the root)

`vercel.json` (generate the CSP in `scripts/` if you need env-based values; otherwise write it directly):

- Rewrites: everything not a file → `/index.html`.
- Headers on all routes: `Content-Security-Policy` (`default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self'; connect-src 'self' https://*.supabase.co; frame-ancestors 'none'; base-uri 'self'; form-action 'self'; object-src 'none'` — add `https://challenges.cloudflare.com` to `script-src` and `frame-src` **only if** Turnstile is enabled), `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy: camera=(), microphone=(), geolocation=()`, `X-Frame-Options: DENY`, `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`, `Cross-Origin-Opener-Policy: same-origin`.
- Route override: `/verify/*` → `Referrer-Policy: no-referrer` and `X-Robots-Tag: noindex, nofollow`.
- Caching: hashed assets `immutable, max-age=31536000`; `index.html` `no-cache`.
- README must explain: deploy this repo as **its own Vercel project**, attach `aptohire.com` and `www.aptohire.com` (redirect www → apex), set the `VITE_*` variables, and that the other four portals are separate Vercel projects on their own subdomains. Mention that Vercel's free Hobby plan is for non-commercial demos.

---

## 12. Testing

| Level | Cover |
|---|---|
| Unit (Vitest) | `scoring.ts` (levels at 0, 50, 51, 69, 70, 84, 85, 100; verified only above 50), ProofRing caption/aria builder, alias glyph determinism, credential-ID parsing from pasted URLs, Zod schemas (valid and invalid payloads), `portalUrl()`, env parser, API client (all outcomes) |
| Component (RTL) | MobileMenu behaviors (open, close on link, outside click, Escape + focus return, closes on resize), forms (validation, honeypot, double-submit guard, field-error mapping), verify page for every state in 7.6, accordion keyboard use |
| E2E (Playwright, mock mode) | Home renders hero and ring; header nav and mobile menu at 390 px; chooser → each portal link has the correct URL; apply flows succeed and show reference; error paths (`fail@`, `rate@`); verify page for each fixture id; 404; **axe scan of every route**; keyboard-only pass through the hero CTAs and the Log in menu; no horizontal overflow check at 360, 390, 768, 1024, 1440 |
| Static | `typecheck`, `lint`, a script that greps `src/` and `dist/` for `service_role`, `sb_secret`, and hard-coded hex colors in components and fails the build |

---

## 13. Components checklist (build all, each with loading/empty/error states where applicable)

`Logo, SealMark, Button (primary / accent / secondary / ghost / danger; height 44 mobile, 40 desktop; label 15/600), Input, Textarea, Select, Checkbox, Field (label + helper + error), Toast, Accordion, Popover, Dialog, Sheet (mobile menu may use it), StatusChip, AliasGlyph, ProofRing, EvidenceRow, EvidenceChip, ScorecardPanel, CandidateCardPreview, SkillGraphPreview, SampleRosterTable, StatRow (plain figures with hairline dividers), PricingPanel, ComparisonTable, CopyButton, Skeleton, EmptyState (palette circles on mesh, one sentence, one button).`

Button rules: the **accent** (orange) button is used for the single most energetic action on a page and always has `ink` text. Loading keeps the label and fixed width.

---

## 14. Writing the README

`README.md` must be beginner-friendly (short sentences, numbered steps): what this project is, how it relates to the other four projects and the shared database, prerequisites, install, copy `.env.example`, run with mocks (`VITE_USE_MOCKS=true`), run against a real Supabase project, commands table, folder tour, how to deploy to Vercel, how to edit copy (`src/content`) and colours (`src/styles/tokens.css`), common problems, glossary. State clearly: **this site never logs anyone in and never needs the service-role key.**

---

## 15. Build order (finish each phase, then run the checks)

1. **Foundations:** Vite/TS/Tailwind config, tokens, fonts, `env.ts`, `portals.ts`, router shell, `PageShell`, `Header` + `MobileMenu`, `Footer`, SkipLink, ESLint/Prettier, Vitest/Playwright setup.
2. **Design primitives:** Button, Field/Input family, Toast, Accordion, Popover, StatusChip, Logo/SealMark, EmptyState, Skeleton.
3. **Proof components:** `scoring.ts` + tests, `ProofRing` (all four states + reveal), `EvidenceRow/Chip` with popovers, `ScorecardPanel`, `AliasGlyph`, `CandidateCardPreview`, `SkillGraphPreview`.
4. **Home page** — all sections, sample fixtures, anchors, responsive pass.
5. **API layer + mocks:** schemas, `api.ts`, fixtures, tests.
6. **Verify:** `/verify`, `/verify/:credentialId` with every state, print styles.
7. **Forms:** `/apply/university`, `/apply/organization`, confirmation panel, Turnstile (optional).
8. **Remaining pages:** Pricing, Get started, legal pages, 404, notice bar, demo page + prototype notice.
9. **SEO and deployment files:** meta, sitemap, robots, OG image, `vercel.json`, build assertions.
10. **Hardening:** axe, Lighthouse, overflow checks at all widths, keyboard pass, secret/hex greps, dependency audit, README.

---

## 16. Acceptance checklist (all must be true)

- [ ] `npm run typecheck`, `lint`, `test`, `build`, `e2e` all pass; no `any`; no `console.log` left
- [ ] Runs fully with `VITE_USE_MOCKS=true` and no network
- [ ] Hero shows the real-UI sample scorecard; the ring reveal plays once and respects reduced motion
- [ ] At most one gradient surface per viewport; none on buttons or text blocks; no all-caps tracked labels; sentence case throughout
- [ ] Primary button labels pass contrast (white on `verdant-700`; `ink` on `ember-500`); no small text in `verdant-600` / `ember-500`
- [ ] Every score shown is paired with openable (sample) evidence
- [ ] Candidate previews show **no** name, photo, university name or file links; only a university category
- [ ] Hamburger menu works as specified (☰/✕, closes on link, outside click, Escape, resize; focus trapped and restored)
- [ ] No horizontal scroll at 360, 375, 390, 768, 1024, 1440 px; tables scroll inside their container
- [ ] All touch targets ≥ 44×44 px on mobile; inputs ≥ 16 px; labels above inputs; form buttons full width on mobile
- [ ] All 7 states of the verify page implemented and tested; renders only API-returned fields; `noindex`; `no-referrer`
- [ ] Both application forms validate, map server field errors, block double-submit, and show the confirmation panel with reference ID
- [ ] No link, route or text anywhere refers to the admin portal
- [ ] No fabricated statistics, testimonials, logos or prices; samples are labelled "Sample"
- [ ] Axe: zero serious/critical violations on every route; Lighthouse ≥ 95 in all four categories; home JS ≤ 170 KB gzipped
- [ ] No secrets in repo/bundle; production build fails if mocks or demo mode are on
- [ ] README complete; `docs/PLAN.md` written; all `TODO(decision)` items listed in the report

---

## 17. Final report (output exactly this at the end)

### ✅ Root website build report

**Stack used:** [versions of Vite, React, Tailwind, Router, etc.]

**Routes built:** [list]

**Components built:** [list]

**Contracts implemented against (mock + real):** `public-verify`, `submit-application`

**Decisions made (`TODO(decision)`):** [each item and the default you chose]

**Quality results:**
- Lighthouse (mobile): Performance [x] · Accessibility [x] · Best Practices [x] · SEO [x]
- Home route JS (gzipped): [x KB]
- Axe violations (serious/critical): [0 per route]
- Overflow check at 360 / 375 / 390 / 768 / 1024 / 1440: [pass/fail]

**Hamburger menu:** [Built in `Header`/`MobileMenu` — behaviors verified: …]

**Files created:** [grouped list]

**What the backend project must provide:** [the endpoints and behaviors in section 4, plus anything you discovered]

**What to check manually:**
- Chrome DevTools → Device Toolbar (Ctrl+Shift+M) at 375, 390, 768, 1024 and 1440 px
- Tab through the whole home page with the keyboard only
- Turn on "reduce motion" in your OS and reload: the ring should appear in its final state instantly
- Open `/verify/a1b2c3d4e5f6a7b8c9` (full), `/verify/b2c3d4e5f6a7b8c9d0` (levels only), `/verify/e5f6a7b8c9d0e1f2a3` (rate limited), `/verify/0000000000000000aa` (not found)
- Submit both application forms with `fail@demo.test` and `rate@demo.test` to see the error states
