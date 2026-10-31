---
date: 30 October 2026
subtitle: Interactive technical explainer
title: AptoHire --- AI Agent & Workflow Architecture
version: 1.0
---

::: {align="center"}
# APTOHIRE / AI SYSTEMS

### **Proof → Skill Signal → Better Match**

**A centralized, n8n-orchestrated workflow for evidence-backed skills
and consent-first talent discovery.**

`MULTIMODAL AUDIT` · `PREDICTIVE BLIND MATCHING` · `PROFILE UPDATE`
:::

> **Status:** Based on AptoHire's idea document and prototype PRD. This
> describes the intended architecture; verify each workflow in the
> deployed product before calling it live or production-ready.

------------------------------------------------------------------------

## 01 --- Architecture at a glance

``` mermaid
flowchart TB
  subgraph USERS["AptoHire portals"]
    S["Student"] 
    O["Organization / Recruiter"]
    U["University"]
  end
  subgraph N8N["n8n · Workflow orchestration"]
    W["Router · validation · conditions · retries"]
    A1["01 · Multi-Model Proof Audit Agent"]
    A2["02 · Predictive Blind Matching Agent"]
    A3["03 · Profile Update Agent"]
    P["AI Project Recommendation Model"]
  end
  subgraph DATA["AptoHire data layer"]
    E["Project evidence: video · audio · script<br/>documents · code · links"]
    K["Skill graph / scorecard"]
    J["Role requirements"]
    PR["Student profile / resume"]
    L["Workflow & audit records"]
  end
  S -->|"Submit evidence"| E --> W --> A1
  A1 -->|"Scores · rationale · evidence refs"| K --> A2
  O -->|"Role / skills"| J --> A2
  A2 -->|"Anonymous ranked candidates + reasons"| O
  S -->|"Target skill / learning goal"| P -->|"Project brief"| S
  S -->|"Explicit permission"| A3 <--> PR
  A1 -.-> L
  A2 -.-> L
  A3 -.-> L
  P -.-> L
```

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}How to read this`</strong>`{=html}
```{=html}
</summary>
```
-   **n8n orchestrates** triggers, branching, API/model calls, retries
    and notifications; it does not guarantee that model output is
    correct.
-   **Agents are task-specific** and should have defined inputs and
    structured outputs.
-   **The application/backend owns permissions and persistence**; AI
    proposes results, while validation and access rules control what is
    saved or revealed.
-   The project recommendation model is a supporting capability from the
    idea document, separate from the three headline agents.

```{=html}
</details>
```

------------------------------------------------------------------------

## 02 --- The three agents

  -----------------------------------------------------------------------
  Agent                   What it does            Main output
  ----------------------- ----------------------- -----------------------
  **01 · Multi-Model      Reviews submitted       Rubric-based skill
  Proof Audit Agent**     project evidence across scores, rationale,
                          audio, visuals,         evidence references and
                          script/transcript and   uncertainty flags.
                          related artefacts.      

  **02 · Predictive Blind Compares a role with    Anonymized shortlist,
  Matching Agent**        verified student skill  match reasons and
                          signals, ranking        missing-skill signals.
                          relevant candidates     
                          while limiting identity 
                          exposure initially.     

  **03 · Profile Update   Helps maintain a        Proposed profile
  Agent**                 student's digital       changes, refreshed
                          profile/resume and      project/skill details
                          lifecycle/affiliation   and sync status.
                          information, using      
                          external sources only   
                          with permission.        
  -----------------------------------------------------------------------

### Agent 01 --- Multi-Model Proof Audit Agent

``` mermaid
flowchart LR
  I["Project submission"] --> V["Visual audit<br/>demo video / artefacts"]
  I --> AU["Audio audit<br/>spoken explanation / audio"]
  I --> T["Script audit<br/>script / transcript / description"]
  I --> X["Cross-check<br/>docs · code · GitHub/live link"]
  V --> F["Evidence fusion + rubric"]
  AU --> F
  T --> F
  X --> F
  F --> Q{"Clear and sufficient?"}
  Q -->|"Yes"| SC["Skill scorecard<br/>score + rationale"]
  Q -->|"Borderline / conflict"| H["Human review / appeal"]
  H --> SC
```

**Purpose:** behave like a project reviewer, combining several evidence
channels rather than trusting a résumé claim or one video alone.

**Suggested output fields:** `skill`, `score`, `rubric_version`,
`rationale`, `evidence_refs`, `confidence`, `flags`, `review_status`.

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Scoring & trust safeguards`</strong>`{=html}
```{=html}
</summary>
```
-   The idea document mentions a score **above 50** for a verified
    skill. Treat this as a proposed threshold until calibrated.
-   Keep evidence references and rubric version so students can
    understand and appeal a result.
-   Send borderline or conflicting cases to a human reviewer. The
    reviewed document's 40--60 band is an example, not a validated rule.
-   Measure false passes, false fails, accessibility and agreement with
    human reviewers.

```{=html}
</details>
```
### Agent 02 --- Predictive Blind Matching Agent

``` mermaid
flowchart LR
  JOB["Organization defines<br/>role / skills / level"] --> N["Normalize role criteria"]
  SK["Verified skill graph<br/>scorecard + evidence strength"] --> M["Structured skill match"]
  N --> M --> R["Rank by relevance<br/>explain why"]
  R --> B["Anonymized candidate cards"]
  B --> ASK["Organization requests access"]
  ASK --> C{"Student consent?"}
  C -->|"Accept"| SHOW["Reveal approved fields only"]
  C -->|"Decline / no response"| HIDE["Identity stays hidden"]
```

**Purpose:** help organizations discover students whose evidence-backed
skills align with a role. Describe this as **relevance matching**, not a
guaranteed "best hire" prediction, until hiring-outcome data validates
it.

**Signals:** required skills, verified level, evidence quality, project
relevance and missing requirements. Start with transparent structured
matching; add vector/semantic search only where tested and useful.

### Agent 03 --- Profile Update Agent

``` mermaid
flowchart LR
  TR["Profile event / source update"] --> C{"Permission for<br/>this source?"}
  C -->|"No"| STOP["Do not fetch or update"]
  C -->|"Yes"| F["Fetch / import allowed data"]
  F --> D["Normalize + compare changes"]
  D --> V["Validate / flag conflicts"]
  V --> P["Preview proposed changes"]
  P --> A{"Approval rule?"}
  A -->|"Approved"| U["Update profile / resume"]
  A -->|"Confirm first"| R["Ask student to confirm"]
  U --> L["Audit log + sync status"]
```

**Purpose:** reduce repetitive profile maintenance and support one
persistent student record across graduation, alumni status, dropout or a
new university affiliation.

**Boundary:** never invent skills, projects or experience. Use explicit,
revocable permission per source. Prefer approved APIs/OAuth where
available; for LinkedIn, use a student-provided data export unless an
authorized integration is available. Do not rely on unauthorized
scraping.

------------------------------------------------------------------------

## 03 --- Supporting capability: AI project recommendation

``` mermaid
flowchart LR
  S["Student enters target skill,<br/>level and interests"] --> P["Project Recommendation Model"]
  LIB["Curated project templates<br/>difficulty + skill tags"] --> P
  P --> OUT["Personalized project brief<br/>goals · constraints · deliverables"]
  OUT --> BUILD["Student builds project"]
  BUILD --> SUB["Submit to Proof Audit Agent"]
```

This model helps a student move from a learning goal to a
level-appropriate, real-world project. Curated templates with controlled
variations can reduce repeated/generic assignments and make outcomes
easier to evaluate.

------------------------------------------------------------------------

## 04 --- End-to-end workflow: proof to opportunity

``` mermaid
sequenceDiagram
  actor Student
  participant App as AptoHire App
  participant N8N as n8n Workflow
  participant Audit as Proof Audit Agent
  participant Store as Skill/Profile Store
  actor Recruiter
  participant Match as Blind Matching Agent

  Student->>App: Submit project + evidence
  App->>N8N: Validated audit request
  N8N->>Audit: Audio + visual + script + artefacts
  Audit-->>N8N: Scores, rationale, flags
  N8N->>Store: Save reviewed result / scorecard
  Recruiter->>App: Post role / required skills
  App->>N8N: Matching request
  N8N->>Match: Role criteria + eligible skill signals
  Match-->>N8N: Ranked anonymous matches + reasons
  N8N-->>App: Candidate cards
  Recruiter->>App: Request candidate access
  App-->>Student: Ask for scoped consent
  Student-->>App: Accept or decline
  App-->>Recruiter: Reveal approved fields only if accepted
```

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Recommended n8n workflow pattern`</strong>`{=html}
```{=html}
</summary>
```
1.  **Trigger** --- webhook or application event.
2.  **Validate** --- authenticate request; check fields, file references
    and consent.
3.  **Route** --- choose audit, matching, profile sync or project
    recommendation.
4.  **Execute** --- call model/API steps with bounded inputs and
    structured outputs.
5.  **Validate output** --- parse schema, check score ranges and safety
    flags.
6.  **Branch** --- save, retry, use rules/mock fallback, or queue human
    review.
7.  **Persist** --- write through a trusted backend/API, not directly
    from a public client into privileged tables.
8.  **Observe** --- record run ID, status, latency, provider/model,
    error category and cost estimate.
9.  **Notify** --- return status to the app and notify relevant users.

Use signed webhooks, bounded retries, timeouts, idempotency keys and a
kill switch for costly model calls.
```{=html}
</details>
```

------------------------------------------------------------------------

## 05 --- Data boundaries & responsible AI

``` mermaid
flowchart TB
  IN["Evidence · profile · role criteria"] --> MIN["Minimize data + verify permission"]
  MIN --> RUN["Task-specific AI workflow"]
  RUN --> CHECK["Validate schema, evidence and flags"]
  CHECK --> HUMAN["Human review for appeals / uncertainty"]
  CHECK --> SAVE["Save scoped, explainable result"]
  HUMAN --> SAVE
  SAVE --> ACCESS["Role-based access + consent-gated reveal"]
  ACCESS --> LOG["Audit trail + monitoring"]
```

  -----------------------------------------------------------------------
  Control                             Rule
  ----------------------------------- -----------------------------------
  **Consent**                         External profile sync and recruiter
                                      identity reveal require scoped
                                      permission; consent should be
                                      revocable.

  **Privacy**                         Pass only task-relevant fields;
                                      keep identity hidden during initial
                                      matching where feasible.

  **Evidence**                        Scores reference evidence and
                                      rubric criteria, not unsupported
                                      model assertions.

  **Human oversight**                 AI supports assessment; it does not
                                      make final hiring decisions.
                                      Appeals and uncertain audits need a
                                      human path.

  **Security**                        Keep provider secrets server-side;
                                      validate signed webhooks; enforce
                                      role-based authorization and
                                      database row-level security.

  **Observability**                   Log status, latency and errors
                                      without unnecessarily copying raw
                                      sensitive content into logs.

  **Fallbacks**                       Use rules/mock fallback when
                                      providers fail and label fallback
                                      results clearly.

  **Fairness**                        Blind matching can reduce some bias
                                      but cannot eliminate it. Monitor
                                      ranking and audit errors.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 06 --- Agent vs. orchestration: quick explainer

```{=html}
<details open>
```
```{=html}
<summary>
```
`<strong>`{=html}Technical roles`</strong>`{=html}
```{=html}
</summary>
```
-   **AI model / agent task:** interprets evidence, proposes a project
    brief, or evaluates role-to-skill relevance.
-   **n8n workflow:** coordinates triggers, API calls, conditions,
    retries, transformations and notifications.
-   **Backend/application:** enforces consent, authorization,
    validation, persistence and user-facing status.
-   **Database / skill graph:** stores profiles, affiliations, project
    references, scores and workflow metadata.
-   **Human reviewer:** resolves appeals and uncertain cases and retains
    oversight over consequential decisions.

**Design principle:** one observable orchestration layer with modular
agent steps is easier to debug, secure and cost-control than many
unbounded autonomous agents calling one another.
```{=html}
</details>
```

------------------------------------------------------------------------

## 07 --- Future extension: AI interviewer

``` mermaid
flowchart LR
  ROLE["Role-specific rubric"] --> Q["Structured question generation"]
  Q --> RESP["Student response<br/>text / audio / coding task"]
  RESP --> EVAL["Rubric-based evaluation"]
  EVAL --> FB["Evidence-linked feedback<br/>uncertainty flags"]
  FB --> HUMAN["Human review / recruiter decision"]
```

**Roadmap only:** the AI interviewer is a future capability, not one of
the three headline agents in this report. It should use role-relevant
questions, consistent rubrics, accessibility options and human
oversight---not automated rejection based only on an AI score.

------------------------------------------------------------------------

## 08 --- Implementation checklist

-   [ ] Define input/output schemas for each workflow.
-   [ ] Add authentication, consent checks and signed webhooks before
    model calls.
-   [ ] Keep AI provider credentials server-side.
-   [ ] Version prompts, models and audit rubrics.
-   [ ] Add output validation, retries, timeouts and a non-AI fallback.
-   [ ] Store run status and evidence references; avoid logging raw
    sensitive content.
-   [ ] Calibrate audit thresholds against human-rated projects.
-   [ ] Test matching relevance, anonymization and consent-gated reveal.
-   [ ] Test profile-sync revocation, conflicts and change previews.
-   [ ] Label mock outputs clearly; verify live functionality before
    claiming it is operational.

------------------------------------------------------------------------

## The one-line story

> **AptoHire turns project evidence into explainable skill signals,
> connects those signals to relevant opportunities through
> privacy-conscious matching, and helps keep a student's professional
> profile current---with n8n coordinating workflows and humans retaining
> oversight.**

`<sub>`{=html}Source basis: AptoHire idea document, reviewed idea
document, and PRD (prototype/demo scope). "Recommended" controls are
technical guidance; live status must be checked against the deployed
workflows.`</sub>`{=html}
