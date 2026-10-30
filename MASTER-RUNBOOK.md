# AptoHire Master Runbook

One checklist to take AptoHire from empty folders to a working, demo-ready system.
Everything here is **fictional demo data**. This is a **prototype**.

---

## 1. The big picture

AptoHire is **five websites + one shared backend**. Each is its own folder, its own Git repository, and (later) its own Vercel project.

```
aptohire/                      ← parent folder (optional, but handy)
├─ aptohire-backend/           ← database, security rules, server functions   (prompt 00 / 00b)
├─ aptohire-root/              ← aptohire.com                port 5173        (prompt 01)
├─ aptohire-student/           ← student.aptohire.com        port 5174        (prompt 02)
├─ aptohire-university/        ← university.aptohire.com     port 5175        (prompt 03)
├─ aptohire-organization/      ← organization.aptohire.com   port 5176        (prompt 04)
└─ aptohire-admin/             ← admin.aptohire.com          port 5177        (prompt 05)
```

Two modes for every website:

| Mode | How | When to use |
|---|---|---|
| **Mock** | `VITE_USE_MOCKS=true` (no backend, no internet) | Building the UI, quick demos, tests |
| **Real** | `VITE_USE_MOCKS=false` + Supabase URL and anon key | Real end-to-end demo across all portals |

---

## 2. Before you start (one-time)

- [ ] **Node.js LTS** (`node --version`)
- [ ] **Git** (`git --version`)
- [ ] **VS Code** (one window per project folder)
- [ ] **Docker Desktop** running (needed for the local database; on Windows use the WSL 2 option)
- [ ] **Supabase CLI** (`npx supabase --version` works without installing)
- [ ] **Deno** (for server-function tests: https://deno.com)
- [ ] An AI coding tool (Claude Code, Cursor, etc.)
- [ ] Later, for deployment: a free **Supabase** account, a **Vercel** account, and your domain

Create the parent folder and the six empty project folders. Run `git init` inside each one.

---

## 3. Which prompt goes where

| Folder | Prompt file | What to paste |
|---|---|---|
| `aptohire-backend/` | `00-aptohire-backend-prompt.md` | **Already ran the first version?** Don't re-run it. Use `00b-aptohire-backend-delta-prompt.md` (see section 4) |
| `aptohire-root/` | `01-aptohire-root-prompt.md` | Whole file |
| `aptohire-student/` | `02-aptohire-student-prompt.md` | Whole file |
| `aptohire-university/` | `03-aptohire-university-prompt.md` | Whole file |
| `aptohire-organization/` | `04-aptohire-organization-prompt.md` | Whole file |
| `aptohire-admin/` | `05-aptohire-admin-prompt.md` | Whole file |

Before the backend prompt: copy your plan docs (`prd.md`, `features.md`, `architecture.md`, `design.md`, `agent.md`) into `aptohire-backend/docs/plan/`.

### How to feed a big prompt (important)

These prompts are long. Don't ask for everything in one go. Use this pattern:

1. Paste the full prompt.
2. Add: **"Do phases 1–3 only. Then stop and give me the report for those phases."**
3. Check the result (run the commands in section 6).
4. Continue: **"Continue with phases 4–6."** And so on until the last phase.
5. Commit to Git after every phase: `git add -A && git commit -m "phase N"`.

If the AI tool loses context, paste this and continue: *"Read `docs/PLAN.md` and the code so far. Continue from phase N of the prompt. Do not redo finished phases."*

---

## 4. Backend: what to do now (you already ran the first prompt)

1. Save `00-aptohire-backend-prompt.md` (the updated one) into `aptohire-backend/docs/prompts/` for reference.
2. Open `aptohire-backend/` in VS Code and paste **`00b-aptohire-backend-delta-prompt.md`**.
3. It will first write an audit (`docs/DELTA_AUDIT.md`: Present / Partial / Missing for each new requirement). **Read it** before saying "continue".
4. Then it adds only what's missing, in **new migration files** (it must never edit old ones), with tests.
5. If your first run did **not** finish all 13 phases: run the delta first, then say: *"Now continue the remaining phases of `docs/prompts/00-aptohire-backend-prompt.md` from phase N. Don't redo finished phases."*
6. When done, run section 6's backend checks, then:

```bash
npm run contract:export
```

7. Copy `contract-dist/CONTRACT.md` into the `docs/` folder of **each** portal project. The portals use it to match exact table, view and function names.

> If your first run produced something badly broken (not just incomplete), the safe reset is local-only: `supabase db reset`. Never reset a remote project.

---

## 5. Recommended build order

1. **Backend** (or at least phases 1–5 of it) so the contract exists.
2. **Root** (`01`) and **Student** (`02`): the core story. Build in mock mode first.
3. **University** (`03`) and **Organization** (`04`).
4. **Admin** (`05`).
5. Switch every portal to real mode and run the end-to-end demo (section 8).
6. Deploy (section 10).

You can build the portals in parallel in separate windows; they run on mock data until you connect them.

---

## 6. Run everything on your computer

### 6.1 Backend (real database, local)

```bash
cd aptohire-backend
cp .env.example .env.local          # Windows PowerShell: copy .env.example .env.local
```

Fill `.env.local` (never commit it). Generate each secret with:

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Use a different value for `ID_HASH_PEPPER`, `ALIAS_PEPPER`, `RATE_LIMIT_PEPPER`, `CALLBACK_SECRET`. Set:

```
AGENT_MODE=mock
APP_ENV=local
ALLOWED_ORIGINS=http://localhost:5173,http://localhost:5174,http://localhost:5175,http://localhost:5176,http://localhost:5177
```

Then (Docker must be running):

```bash
npm run db:start      # starts the local Supabase
npm run db:reset      # applies migrations + demo data
npm run db:test       # security tests (RLS)
npm run fn:test       # function tests
npm run fn:serve      # keep this terminal open: serves the Edge Functions
npx supabase status   # shows API URL + anon key (copy these; never share the service_role key)
```

Handy local addresses: API `http://127.0.0.1:54321` · Studio (database viewer) `http://127.0.0.1:54323` · **Inbucket (catches sign-up emails) `http://127.0.0.1:54324`**.

### 6.2 Portals

For each portal folder:

```bash
npm install
cp .env.example .env.local
npm run dev
```

**Mock mode** (default in `.env.example`): nothing else to set. **Real mode:** edit `.env.local`:

```
VITE_USE_MOCKS=false
VITE_DEMO_MODE=true
VITE_SUPABASE_URL=http://127.0.0.1:54321
VITE_SUPABASE_ANON_KEY=<anon key from "supabase status">
VITE_DEMO_PASSWORD=<DEMO_PASSWORD from aptohire-backend/.env.example>
```

Stop and restart `npm run dev` after any `.env.local` change. Every portal also needs its sibling URLs, already pre-filled in `.env.example` (`VITE_ROOT_URL=http://localhost:5173`, `VITE_STUDENT_URL=http://localhost:5174`, …).

### 6.3 Start all six together

**Windows (PowerShell):** save as `start-all.ps1` in the parent folder, then run `.\start-all.ps1`.

```powershell
$root = $PSScriptRoot
Start-Process powershell -ArgumentList "-NoExit","-Command","cd '$root\aptohire-backend'; npm run db:start; npm run fn:serve"
foreach ($p in 'root','student','university','organization','admin') {
  Start-Process powershell -ArgumentList "-NoExit","-Command","cd '$root\aptohire-$p'; npm run dev"
}
```

**macOS / Linux (one terminal):** from the parent folder, after `npm run db:start` has run once in the backend:

```bash
npx concurrently -n fn,root,student,uni,org,admin \
  "cd aptohire-backend && npm run fn:serve" \
  "cd aptohire-root && npm run dev" \
  "cd aptohire-student && npm run dev" \
  "cd aptohire-university && npm run dev" \
  "cd aptohire-organization && npm run dev" \
  "cd aptohire-admin && npm run dev"
```

Open these in the browser:

| Site | Address |
|---|---|
| Root (marketing, verify, apply) | http://localhost:5173 |
| Student | http://localhost:5174 |
| University | http://localhost:5175 |
| Organization | http://localhost:5176 |
| Admin | http://localhost:5177 |

> Each site keeps its **own** sign-in. Being signed in on one doesn't sign you in on another. That's by design.

---

## 7. Demo accounts

Password for all: the `DEMO_PASSWORD` in the backend `.env.example` (mock mode: anything). Demo panels on each login page give one-click buttons.

| Portal | Email | Who |
|---|---|---|
| Admin | `admin@demo.aptohire.test` | Admin 1 (needs authenticator; mock code `123456`) |
| Admin | `reviewer2@demo.aptohire.test`, `reviewer3@demo.aptohire.test` | Other reviewers (for the "different reviewer" rule) |
| Student | `aarav@unia.edu.test` | Uni A, React Proficient, 4 consent states |
| Student | `naman@demo.aptohire.test` | Uni B, brand new |
| Student | `avi@demo.aptohire.test` | Alumnus; pending "confirm new university" |
| Student | `niya@demo.aptohire.test` | Unverified university |
| University | `tpo@unia.edu.test` | Uni A (rich data) |
| University | `tpo@unib.edu.test` / `tpo@unic.edu.test` | Uni B (thin) / Uni C (**pending**) |
| Organization | `recruiter@orga.demo.test` | Northwind Labs |
| Organization | `recruiter@orgb.demo.test` | Brightlane |
| Organization | `recruiter@orgc.demo.test` / `recruiter@orgd.demo.test` | Orbit Analytics (**pending**) / Cedar Works (**subscription inactive**) |

**Admin MFA in real mode:** there is no shortcut. Create the first admin with the backend script `scripts/create-admin.sql`, sign in at the admin site, scan the QR with an authenticator app (Google Authenticator, Authy, 1Password…), and enter the 6-digit code each time you sign in.

---

## 8. End-to-end demo script (about 7 minutes)

Use **real mode** with all six running. (In mock mode the portals can't talk to each other, so use each portal's built-in **demo tools** instead; they are noted in the last column.)

| # | Portal · account | Do this | You should see | Mock-mode shortcut |
|---|---|---|---|---|
| 1 | **Admin** · Admin 1 | Approvals → open **Uni C** → read the checks → **Approve** | Consequence text ("unlocks registration for 6 rostered students"), toast "University approved." | University portal Account → **Simulate admin approval** |
| 2 | **Student** · Aarav | Projects → **Get a project** → React → read the brief → **Start project** | A project brief with deliverables and a rubric preview | n/a |
| 3 | **Student** · Aarav | Fill the submission (title, description, small test video under 3:10, AI-use disclosure) → **Submit project** | Live progress steps; after about 15 seconds "Your React score is ready." with the Proof Ring and evidence chips | Same (backend mock agent / student mock) |
| 4 | **Organization** · Northwind | Jobs → open **Frontend Developer (React)** → Candidates | Ranked **anonymous** cards (alias, category, match words, lock row). **No names anywhere** | Same |
| 5 | **Organization → Student → Organization** | Northwind **Request access** to a candidate. In the student site (Aarav) → Sharing → **Review request** → choose scopes and 30 days → **Accept request**. Back in Northwind → Requests → **Accepted** → **View candidate** | The revealed profile with only the shared scopes, plus the expiry date | Org Account → Demo tools → **Simulate student response** |
| 6 | **University** · Uni A | Overview → funnel, skills, **Unmatched demand** (Data Visualization) → Students → open Aarav | Funnel with conversion %, reasons shown as counts, **no student names in demand views** | Same |
| 7 | **Student → Root** · Aarav | Scorecard → **Download resume** → scan the QR (or open the link) | Root site `/verify/<id>` showing only what Aarav chose to share | Same (link opens the root site) |
| 8 | **Student** · Avi, then Niya | Avi: Profile → Education → **Confirm** the new university. Niya: see the dashed **Unverified** chip | Scores unchanged; "Your scores and job matches are not affected." | Same |
| Bonus | **Admin** · Admin 1 | Reviews → **Naveen's React 52** → Confirm. Tickets → decide as a different reviewer. Settings → try the **kill switch** | Blind review (no identity), consequence preview, banner on every page | Same |

Reset between runs: backend `npm run db:reset` (real) or each portal's **Reset demo data** (mock).

---

## 9. Quick health checks (after each phase)

Run in every portal folder:

```bash
npm run typecheck && npm run lint && npm run test && npm run build
```

Run in the backend:

```bash
npm run db:reset && npm run db:test && npm run fn:test && npm run fn:check
```

Manual smoke test per portal (2 minutes):

- [ ] Page loads with no red errors in the browser console
- [ ] Resize to **375 px** wide: no sideways scrolling, menus work (hamburger or bottom bar)
- [ ] Tab through the page with the keyboard: you can always see where you are
- [ ] Sign in with a demo account; sign in with the **wrong** kind of account and see "Go to your portal"
- [ ] Turn on "reduce motion" in your operating system: nothing important animates

Privacy checks worth repeating before any demo:

- [ ] Organization candidate pages contain **no** names, emails, photos or university names before a student accepts (search the page with DevTools → Elements)
- [ ] University **Unmatched demand** shows counts only
- [ ] Admin review screens show **no** student identity
- [ ] No page ever shows a key, password or token

---

## 10. Putting it online (do this after the local demo works)

Do it in this order. **Only you run these commands; AI tools must not run them against a real project.**

1. **Supabase demo project:** create it (free), note the project ref, URL, anon key. Keep the `service_role` key secret.
2. **Dashboard settings:** Authentication → URL Configuration: add all five site URLs and the `localhost` ones; turn on email confirmation; password minimum length 10; enable TOTP MFA; set up email sending (the built-in sender has low limits); turn off APIs you don't use; check storage limits.
3. **Push the database:** in `aptohire-backend/`:
   ```bash
   npx supabase login
   npx supabase link --project-ref <your-project-ref>
   npx supabase db push
   ```
4. **Set function secrets** (from a file that is **not** committed) and deploy functions:
   ```bash
   npx supabase secrets set --env-file .env.production.local
   npx supabase functions deploy
   ```
   Use `APP_ENV=demo` (or `production` for a real launch: it disables seed/reset), new random peppers, and `ALLOWED_ORIGINS` with your five real URLs.
5. **Demo data (demo project only):** run the seed in the SQL Editor. **Never** seed a production project.
6. **Create the first admin** with `scripts/create-admin.sql`, then enrol the authenticator.
7. **Vercel:** create **five separate projects**, one per folder. Set each project's `VITE_*` variables (real Supabase URL + anon key, `VITE_USE_MOCKS=false`, `VITE_DEMO_MODE=false` for a real launch). Attach domains: `aptohire.com` (+ `www` redirect), `student.`, `university.`, `organization.`, `admin.`.
8. **Smoke test** every site over HTTPS (section 9), then run the demo script on the live URLs.
9. **Later:** run your `Security.md` audit prompt against the repos. Vercel's free Hobby plan is for non-commercial use; move to a paid plan before charging money or running a company pilot. Supabase free projects pause after about a week of inactivity (a daily keep-alive is in the backend).

> The admin site: don't advertise its address. Its protection is the password + authenticator, but a quiet address is still a good habit. Optionally add Vercel Deployment Protection or an IP allow-list in front of it.

---

## 11. Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| `supabase start` fails | Docker not running | Start Docker Desktop, wait until it says "running", retry |
| Port already in use | Another copy is running | Close the other terminal or change the port with `npm run dev -- --port <n>` (keep the portal's own port so cross-links work) |
| White page, red errors | `.env.local` typo or missing | Re-copy `.env.example`; restart `npm run dev` |
| "Invalid API key" | Wrong key | Use the **anon/publishable** key from `supabase status`, never `service_role` |
| Browser shows a CORS error | Origin not allowed | Add the portal's address to `ALLOWED_ORIGINS` in the backend `.env.local`; restart `fn:serve` |
| Sign-up "check your email" but nothing arrives | Local email is caught, not sent | Open Inbucket at http://127.0.0.1:54324 |
| "Go to your portal" screen | Signed in with the wrong role for this site | Sign out and use the right demo account |
| Pages empty / "permission denied" | Wrong role, missing seed, or admin not at MFA level | Re-run `npm run db:reset`; for admin finish MFA |
| Demo account missing in real mode | Backend seed doesn't have it yet | Run the backend delta prompt (section 4) and `db:reset` |
| Function returns 500 | Missing function secret or stack error | Read the `fn:serve` terminal; check `.env.local` names against `.env.example` |
| Review never finishes | Functions not running or `AGENT_MODE` wrong | Keep `fn:serve` running; set `AGENT_MODE=mock` for demos |
| Realtime updates don't arrive | Channel not joined | The apps fall back to polling; check the backend's Realtime publication tests |
| Video upload fails | Too long (over 3:10) or too large | Trim or compress; check the bucket size limit |
| Portal changes not showing | Old env or cache | Stop and restart `npm run dev`; hard refresh (Ctrl+Shift+R) |
| Mock and real data confusing you | Different `VITE_USE_MOCKS` per portal | Check each `.env.local`; mock data lives only in that browser tab/session |
| Lost authenticator (admin) | Device lost | Another admin resets the factor (backend RUNBOOK); never share codes |
| Supabase project paused | Free plan inactivity | Open the project and click **Restore** |

---

## 12. Golden rules

1. **Never commit** `.env*` files (only `.env.example`). **Never share** the `service_role` key or any pepper.
2. **Never edit an applied database migration.** Add a new one.
3. **Never let an AI tool run `db push`, `functions deploy` or `secrets set` against a real project.** You do those steps yourself.
4. **Never weaken security to make something work** (turning off RLS, widening a view, skipping MFA). If something is blocked, ask why.
5. **Use only fictional data** in demos.
6. **Commit after every phase.** Small commits make mistakes easy to undo.
7. **When the contract changes** (new view, renamed column), re-run `npm run contract:export` in the backend and re-copy `CONTRACT.md` into the portals.

---

## 13. Definition of done

- [ ] Backend: `db:reset`, `db:test`, `fn:test` all pass; `CONTRACT.md` exported and copied to the five portals
- [ ] Every portal: `typecheck`, `lint`, `test`, `build`, `e2e` pass; works at 375 px; keyboard-friendly
- [ ] The 8-step demo script runs end to end in real mode, twice in a row after a reset
- [ ] Privacy checks in section 9 pass
- [ ] Deployed demo (if wanted): five Vercel projects, five domains, Supabase configured, first admin enrolled with MFA
- [ ] `Security.md` audit run against the backend and fixes applied
