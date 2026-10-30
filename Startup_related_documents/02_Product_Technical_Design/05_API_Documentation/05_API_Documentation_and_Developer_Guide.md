# AptoHire — API Documentation & Developer Guide
**Version:** 1.0 · **Date:** 30 October 2026 · **Source of truth:** `FROZEN-CONTRACTS.md`

> **Contract warning:** This guide is an orientation layer. Request/response shapes, enum values, field bounds, status codes and error codes in the supplied `FROZEN-CONTRACTS.md` are binding. Do not change them by paraphrase or implementation preference. Include the full frozen file in the backend repository and keep this guide synchronized with it.

## Navigation
- [1. Base URL and authentication](#1-base-url-and-authentication)
- [2. Common error contract](#2-common-error-contract)
- [3. Anonymous root endpoints](#3-anonymous-root-endpoints)
- [4. Student portal functions](#4-student-portal-functions)
- [5. University portal functions](#5-university-portal-functions)
- [6. Organization and admin functions](#6-organization-and-admin-functions)
- [7. Shared data contracts](#7-shared-data-contracts)
- [8. Security requirements for implementers](#8-security-requirements-for-implementers)
- [9. Local development and testing](#9-local-development-and-testing)
- [10. Contract change process](#10-contract-change-process)
- [Appendix A: Frozen source](#appendix-a-frozen-source)

## 1. Base URL and authentication

Base URL: `${SUPABASE_URL}/functions/v1` (frontend environment variable is described as `${VITE_SUPABASE_URL}/functions/v1`).

| Endpoint class | Authentication | Rules |
|---|---|---|
| `public-verify` | Public anon key in `apikey` and `Authorization: Bearer <anon key>` | Anonymous, rate-limited and cached; return only permitted public fields. |
| `submit-application` | Public anon key in both headers | Anonymous, validated, rate-limited and idempotent; must not create a login. |
| All other functions | Signed-in user's JWT | Authenticate and authorize explicitly on the server. |
| `admin-*` functions | Signed-in admin JWT with `aal2` | MFA step-up required; audit sensitive actions. |

A successful login/session does not grant arbitrary portal access. Roles are server-controlled and must not be self-assigned by users. CORS is not authorization.

## 2. Common error contract

The frozen contract specifies this shape:

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Human-readable safe message",
    "fields": { "email": "Field-level message" }
  }
}
```

`fields` is optional. Frontends map stable codes to calm copy and must not display raw database or model-provider errors.

| Code | HTTP | Meaning / handling |
|---|---:|---|
| `VALIDATION_FAILED` | 422 | Show field errors when supplied. |
| `UNAUTHENTICATED` | 401 | Re-authenticate or sign in. |
| `FORBIDDEN` | 403 | Explain insufficient permission without leaking protected data. |
| `NOT_FOUND` | 404 | Render not-found state. |
| `CONFLICT` | 409 | Resolve state conflict. |
| `ROLE_ALREADY_SET` | 409 | Do not silently change role. |
| `ACCOUNT_EXISTS` | 409 | Use safe account-recovery copy. |
| `RATE_LIMITED` | 429 | Honor `Retry-After`. |
| `QUOTA_EXCEEDED` | 429 | Show retry/usage state. |
| `KILL_SWITCH` | 503 | AI/workflow temporarily disabled. |
| `SERVICE_UNAVAILABLE` | 503 | Safe retry state. |
| `SUBSCRIPTION_INACTIVE` | 402 | Subscription/demo gate state. |
| `NO_TEMPLATE` | 422 | No suitable project template. |
| `VIDEO_REQUIRED` | 422 | Required video missing. |
| `VIDEO_TOO_LONG` | 422 | Video exceeds allowed duration. |
| `MISSING_SKILLS` | 422 | Required target skill information missing. |
| `GRANT_EXPIRED` | 403 | Consent grant expired; protected fields stay hidden. |
| `MFA_REQUIRED` | 403 | Complete MFA step-up. |

## 3. Anonymous root endpoints

### 3.1 `GET /public-verify?id={credentialId}`

- Credential ID format: exactly 18 lowercase hexadecimal characters (`^[a-f0-9]{18}$`).
- Invalid client-side format: show “not found” without making a network call.
- Success `200`: `{ "credential": { ... } }`; fields such as `display_name`, `affiliation`, and skill `score` are optional and only returned if permitted.
- `skills` may be empty; never label unverified skills as verified.
- Errors: `404 NOT_FOUND`, `429 RATE_LIMITED` with `Retry-After`, or `500/503 SERVICE_UNAVAILABLE`.
- Render only fields actually returned. Parse response with the client Zod schema; unknown fields are dropped per the frozen contract.

### 3.2 `POST /submit-application`

Header: `Idempotency-Key: <crypto.randomUUID()>`; use one key per form instance and regenerate only after successful submit.

**University payload** is a discriminated union member with `type: "university"` and fields:
`institution_name` (2–120 chars), optional `short_name` (≤30), `kind` (`university|college|school`), `tier` (`tier1|tier2|tier3|other`), `city`, `state`, optional `email_domains[]`, `student_count_band` (`<500|500-2000|2000-5000|5000+`), optional `departments` (≤200), `contact_name`, `contact_role`, `contact_email`, optional `contact_phone`, optional `message` (≤1000), literal `consent: true`, required honeypot `hp: ""`, optional `captcha_token`.

**Organization payload** uses `type: "organization"` and fields:
`org_name` (2–120 chars), `org_kind` (`corporate|startup|agency|other`), `size_band` (`1-10|11-50|51-250|251-1000|1000+`), HTTPS `website`, `hiring_focus[]` (`web_frontend|backend|data|management_commerce|other`), `contact_name`, `contact_role`, `work_email` (common free-mail domains rejected with friendly message), optional `message`, literal `consent: true`, required honeypot `hp: ""`, optional `captcha_token`.

Success: `202 { "reference": "AH-7K2Q9F", "status": "received" }` (reference format illustrative in source). Errors include `422 VALIDATION_FAILED`, `429 RATE_LIMITED`, `503 SERVICE_UNAVAILABLE`. Backend must create a pending application, verify CAPTCHA if configured, rate-limit by IP/email, never create a login, and allow admin approval/rejection.

## 4. Student portal functions

All use the student JWT unless otherwise specified.

| Function | Request → response | Notes |
|---|---|---|
| `complete-signup` | `{ portal: "student" }` → `204` | Sets role once; idempotent. |
| `register-student` | `{ full_name, program_level: "ug"|"pg", year?, choice: {kind:"detect"} \| {kind:"student_id", university_id, student_id_number} \| {kind:"unregistered", university_name} }` → `{ student_id, outcome, affiliation?, matched_existing_person }` | Raw student ID travels only over TLS to this function; never log it, keep it in post-submit state or put it in URL. Frozen contract also has backend-delta requirements; reconcile them explicitly. |
| `confirm-affiliation` | `{ proposal_id }` or `{ new: { university_id?, university_name_text?, student_id_number?, program_level, year?, status } }` → `{ affiliation }` | Close old affiliation and open new one; preserve scores. |
| `assign-project` | `{ target: {skill_slug} \| {role_text}, level?: "Competent"|"Proficient"|"Advanced" }` → `201 { assignment_id, brief: ProjectBrief }` | Errors include `NO_TEMPLATE`, `QUOTA_EXCEEDED`, `KILL_SWITCH`, `VALIDATION_FAILED`. |
| `submit-project` | `{ submission_id, confirm: true }` → `202 { run_id, status, mode }` | Requires video and target skills per frozen contract; retries failed submissions. |
| `respond-access` | `{ request_id, decision: "accept"|"decline", granted_scope?: Scope[], duration_days?: 7|30|90 }` → `{ status, expires_at? }` | Enforce scope and duration server-side. |
| `withdraw-access` | `{ request_id }` → `{ status: "withdrawn" }` | Revoke access immediately. |
| `profile-sync` | `{ action:"github_start" }` → `{ authorize_url }`; or `{ action:"github_sync"|"linkedin_parse"|"revoke", source_id?, text?, file_path?, delete_data? }` → `{ proposals?, last_synced_at }` | User review of proposals; GitHub is read-only; LinkedIn source is export upload in supplied architecture. |
| `export-my-data` | `{}` → `{ signed_url }` | Marked “backend to add” in frozen contract. |
| `delete-my-account` | `{ password, confirm:"DELETE" }` → `204` | Marked “backend to add”; deletion/retention semantics need verification. |

### Student shared types

```ts
type Scope = 'profile' | 'scorecard' | 'projects' | 'contact';

type ProjectBrief = {
  title: string; scenario: string; level: 'Competent'|'Proficient'|'Advanced';
  target_skills: string[]; deliverables: string[]; constraints: string[];
  suggested_stack?: string[]; estimated_hours: number;
  rubric_preview: { criterion: string; weight: number }[];
  video_guidance: string; originality_questions: string[];
};

type EvidenceItem = {
  ref: string; kind: 'video'|'file'|'repo'|'doc'|'live';
  label: string; pointer: string; excerpt?: string
};

type AuditCriterion = {
  name: string; weight: number; score: number | null;
  rationale: string; evidence_refs: string[]; insufficient_evidence?: boolean
};

type Audit = {
  id: string; submission_id: string; created_at: string; overall: number;
  confidence: number; summary: string; next_steps: string[]; flags: string[];
  borderline: boolean; review_status: 'none'|'pending'|'confirmed'|'adjusted';
  per_skill: { skill_id: string; score: number; criteria: AuditCriterion[] }[];
  evidence: EvidenceItem[]; transcript?: { t: number; text: string }[]
};

type ProfileProposal = {
  source: 'github'|'linkedin_export';
  changes: { field: 'headline'|'skills_interest'|'experience'|'education'|'links'|'project_candidate';
    action: 'add'|'update'; value: unknown; evidence: string }[]
};
```

Evidence pointers: video pointer is seconds as string; file/repo pointer is `path#Lstart-Lend`. Confidence labels from frozen contract: `>=0.75` High, `>=0.5` Medium, otherwise Low; thresholds are noted as a decision point in the source.

## 5. University portal functions

All use the university staff JWT.

| Function / view | Contract summary |
|---|---|
| `complete-signup` | `{ portal:"university", application_ref? }` → `204`; role is set once and idempotently. Attach user only when verified email matches the application's contact email and application reference rules are satisfied. |
| `roster-import` | `{ university_id, source: { csv_path } \| { rows }, dry_run? }` → `ImportReport`; supports dry run, real import and one-row additions. |

`ImportReport`:
```ts
type LifeStatus = 'incoming'|'enrolled'|'dropout'|'alumni';
type CanonicalRow = {
  full_name: string; email?: string; student_id_number?: string;
  program?: string; program_level: 'ug'|'pg'; year?: number; status: LifeStatus;
};
type ImportReport = {
  total: number; valid: number;
  invalid: { row: number; field: string; reason: string }[];
  duplicates: number; created: number; updated: number;
  matched_existing: number; // count only, never identify matched rows
  proposals_created: number; // count only
};
```

Canonical CSV header: `full_name,email,student_id_number,program,program_level,year,status`. Client must map columns, drop all unmapped columns before upload and re-serialize. Raw student ID travels only to the import function over TLS and must never be logged.

Additional views and constraints specified by backend-delta prompts include `my_university_v`, `university_students_v` (including stable `roster_entry_id` and nullable `student_id` for unclaimed rows), and `university_student_skills_v` restricted to verified, share-approved and non-pending skills for the caller's own students. Treat these as implementation requirements from the supplied delta prompt; verify actual existence before claiming availability.

## 6. Organization and admin functions

The frozen contract contains further organization/admin functions and their exact schemas. Because these contracts are binding and lengthy, the complete `FROZEN-CONTRACTS.md` should remain checked into the same docs directory and be treated as the definitive endpoint reference. Do not infer a request schema from the function name.

Important invariants:
- Organization queries use identity-free candidate projections. No direct identity table access.
- `reveal-student` is the only identity path and must check a valid consent grant and audit the event.
- Admin functions require `aal2` and are audited.
- Appeals should be reviewed by a different reviewer than the original audit reviewer.
- Application config supplies the pass mark and borderline range; do not hard-code them.
- Backend-delta prompt additions are binding; record implementation gaps in a delta audit.

## 7. Shared data contracts

The common error envelope is specified above. Key data types include `ProjectBrief`, `EvidenceItem`, `AuditCriterion`, `Audit`, `ProfileProposal`, `Scope`, `CanonicalRow` and `ImportReport`. The frozen contract also defines exact root verification response fields, application forms, role flows, and additional organization/admin schemas.

Do not loosen client schemas or make optional fields mandatory unless the frozen contract explicitly says so. Optional public credential fields must be rendered only if returned.

## 8. Security requirements for implementers

- Validate requests server-side even when frontend uses Zod.
- Keep service-role keys and provider secrets out of browser bundles.
- Enforce RLS and explicit function authorization; CORS is not authorization.
- Validate role and `aal2` for admin actions on every request.
- Rate-limit anonymous endpoints, AI calls and expensive actions; honor `Retry-After`.
- Verify idempotency keys and avoid duplicate application creation.
- Never log raw student ID numbers, passwords, tokens, secrets or full evidence payloads.
- Ensure storage access is scoped and signed URLs expire.
- Audit reveal, consent changes, approvals, appeal decisions and other sensitive operations.
- Keep mock data fictional. Never use mock mode for real candidate data.
- Do not send repositories, student data or secrets to third-party scanners.
- Use safe error copy; do not expose raw stack traces or provider responses.

## 9. Local development and testing

The supplied README says portals default to mock mode and uses fictional data. It describes Node.js 20.19+ as the baseline. Verify each repository's own scripts before running commands; do not assume identical scripts across five repositories.

Suggested verification sequence:
1. Install dependencies and run each portal's existing unit/lint/type-check scripts.
2. Run Vitest unit/integration tests and Playwright end-to-end tests where present.
3. Run Supabase migrations and database tests in a local supported environment.
4. Run Edge Function type-check/lint/test commands where configured.
5. Verify contract conformance for every frozen endpoint.
6. Test negative access cases: cross-student, cross-university, cross-organization, no-consent reveal, expired grant, withdrawn grant, role escalation and admin without MFA.
7. Test error handling, rate limits, idempotency, kill switch and provider outage.
8. Report each check as **Verified** (executed), **Inferred** (from code/config), **Not run**, or **Manual dashboard step**.

No test results are claimed by this document.

## 10. Contract change process

1. Identify the consumer and reason for change.
2. Propose a versioned change with backward-compatibility impact.
3. Update frozen contract and portal schemas together; do not silently rename fields or enums.
4. Update backend contract export and client adapters.
5. Add positive and negative contract tests.
6. Review security implications, particularly identity, consent, role and MFA behavior.
7. Record the migration and rollout plan; maintain compatibility or explicitly coordinate a breaking release.

## 11. Required companion files

Keep these files in the backend docs directory:
- `FROZEN-CONTRACTS.md` — exact request/response source of truth.
- `00-aptohire-backend-prompt.md` and backend delta prompt — implementation/security additions.
- Product plan files: `prd.md`, `features.md`, `architecture.md`, `design.md`, `agent.md`.
- `CONTRACT.md` and `DELTA_AUDIT.md` generated/maintained by the backend implementation process.

## Appendix A: Frozen source

For the complete binding API specification, refer to the supplied `FROZEN-CONTRACTS.md`. This document is intentionally preserved as a separate file rather than copied into a rewritten guide, to avoid accidental contract drift.
