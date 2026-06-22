# PRD: ACME GLOBAL MEDIA — AI Content Workflow

| Field | Value |
|-------|-------|
| **Ticket** | [86e1ykv42](https://app.clickup.com/t/86e1ykv42) (PHASE-0.3) |
| **Status** | Draft — pending Ro review |
| **Client** | ACME GLOBAL MEDIA (fictional) |
| **Repository** | [rosariosm/fullstack-engineer-ai-content-workflow-challenge](https://github.com/rosariosm/fullstack-engineer-ai-content-workflow-challenge) |

---

## Objective

ACME GLOBAL MEDIA produces ads, micro-sites, and marketing materials across multiple languages. Today, drafting and localizing campaign content is slow and error-prone.

This product delivers a **web application** where teams can:

1. Organize work as **campaigns** with multiple **content pieces**.
2. Use **LLMs** (OpenAI and/or Anthropic) to generate drafts and translation/localization suggestions.
3. Run a **human-in-the-loop review workflow** before content is considered approved.
4. See **real-time updates** when any user changes campaign or content state.

**Training goals** (implementation context, not client-facing):

- Learn **Django** as the backend framework.
- Deliver via **harness-driven** incremental development (`AGENTS.md`, `feature_list.json` in Phase 1+).
- Practice AI-assisted SDLC with traceability to ClickUp.

**API choice:** **GraphQL only** (Strawberry on Django). Rationale: training stack requirement; single endpoint for dashboard queries; **GraphQL subscriptions** for real-time updates without a separate WebSocket layer.

---

## User Flow / Personas

### Personas

| Persona | Goals | Primary actions |
|---------|-------|-----------------|
| **Content Manager** | Create campaigns, add content pieces, trigger AI generation | Create/edit campaigns and pieces; request AI drafts and translations |
| **Reviewer** | Quality-check AI output before publication | Review, edit, approve, or reject content |
| **Admin** (optional, v1-lite) | Bootstrap demo environment | Manage locales/providers config (env-based in v1) |

Authentication is **out of scope for v1** unless required for demo; a single shared workspace is acceptable for the challenge submission.

### Primary flows

**F1 — Create campaign and content**

1. Content Manager opens the campaign dashboard.
2. Creates a campaign (name, description, target locales).
3. Adds one or more content pieces (type: headline, description, etc.; source locale).
4. Dashboard lists campaigns with piece counts and review status summary.

**F2 — Generate AI draft**

1. Content Manager selects a content piece in `Draft` state.
2. Triggers **Generate draft** (provider: OpenAI or Anthropic, configured server-side).
3. System calls the LLM, stores the suggestion, transitions state to `SuggestedByAI`.
4. Reviewer sees the new suggestion (real-time update on dashboard/detail view).

**F3 — Translate / localize**

1. Content Manager selects a piece with approved or draft source text.
2. Triggers **Translate** to a target locale.
3. System returns a localization suggestion; state becomes `SuggestedByAI` for the target variant.
4. Optional (P1): extract keywords, tone, or sentiment as structured metadata on the piece.

**F4 — Review workflow**

1. Reviewer opens a piece in `SuggestedByAI` or `Reviewed` state.
2. Reads AI suggestion alongside current text.
3. Actions:
   - **Approve** → `Approved`
   - **Reject** → `Rejected` (with optional reason)
   - **Edit and save** → `Reviewed` (human-edited body stored as current version)
4. State change is visible to all connected clients in real time.

**F5 — Query and monitor**

1. Any user opens the dashboard.
2. GraphQL query returns campaigns, nested content pieces, and review states.
3. Filters (P1): by campaign, review status, or locale.

---

## Entities & Use Cases

### Core entities

| Entity | Description | Key fields |
|--------|-------------|------------|
| **Campaign** | Container for a marketing initiative | `id`, `name`, `description`, `target_locales[]`, `created_at`, `updated_at` |
| **ContentPiece** | Single unit of content within a campaign | `id`, `campaign_id`, `type` (headline, description, …), `source_locale`, `current_text`, `review_status`, `updated_at` |
| **ContentSuggestion** | AI-generated or translated variant | `id`, `content_piece_id`, `provider`, `model`, `suggested_text`, `metadata` (keywords/tone/sentiment P1), `created_at` |
| **ReviewEvent** (optional audit) | History of status transitions | `id`, `content_piece_id`, `from_status`, `to_status`, `actor`, `note`, `created_at` |

### Review status (state machine)

```
Draft → SuggestedByAI → Reviewed → Approved
                    ↘           ↘ Rejected
         Rejected ← (from SuggestedByAI or Reviewed)
```

- **Draft:** Initial or reset state; no pending AI suggestion required.
- **SuggestedByAI:** AI output available for human review.
- **Reviewed:** Human edited text; not yet final approval.
- **Approved:** Accepted for use in the campaign.
- **Rejected:** Not accepted; may regenerate from Draft.

### Use cases

| ID | Use case | Actor |
|----|----------|-------|
| UC-1 | Create and list campaigns | Content Manager |
| UC-2 | Add, edit, and list content pieces under a campaign | Content Manager |
| UC-3 | Generate AI draft for a content piece | Content Manager |
| UC-4 | Request translation/localization to a target locale | Content Manager |
| UC-5 | Approve, reject, or edit AI-suggested content | Reviewer |
| UC-6 | Query campaigns with nested pieces and review states | Any user |
| UC-7 | Receive real-time updates when content or status changes | Any user |

---

## Acceptance Criteria

Criteria marked **P0** are required for challenge submission. **P1** items are nice-to-have / bonus.

### Backend (Django + Strawberry GraphQL)

- [ ] **P0** GraphQL API runs locally and connects to PostgreSQL.
- [ ] **P0** Mutations: create campaign, create content piece, update piece text, transition review status.
- [ ] **P0** Mutations: trigger AI draft generation (OpenAI or Anthropic SDK).
- [ ] **P0** Mutations: trigger translation/localization for a target locale.
- [ ] **P0** Queries: list campaigns; fetch campaign with nested content pieces and `review_status`.
- [ ] **P0** Review transitions enforce valid state machine paths (invalid transitions return a clear error).
- [ ] **P0** AI provider and API keys loaded from environment variables (documented in `.env.example`).
- [ ] **P1** Query filters by `review_status` and/or locale.
- [ ] **P1** Store structured metadata (keywords, tone, sentiment) on suggestions.
- [ ] **P1** Support switching or comparing OpenAI vs Anthropic for the same piece.

### Frontend (React)

- [ ] **P0** Campaign dashboard lists campaigns and summary of content/review states.
- [ ] **P0** UI to create a campaign and add content pieces.
- [ ] **P0** UI to trigger AI draft generation and show loading/error states.
- [ ] **P0** Review UI: view suggestion, edit text, approve, reject.
- [ ] **P0** Real-time updates when another client changes data (GraphQL subscription or equivalent).
- [ ] **P1** Campaign detail filters and status badges.

### Platform & delivery

- [ ] **P0** `docker compose` (or equivalent) starts backend, frontend, and PostgreSQL locally.
- [ ] **P0** Root `README.md` includes setup steps, tech decisions, and GraphQL-only justification.
- [ ] **P0** `docs/` contains this PRD and room for TRD/ADRs.
- [ ] **P1** Unit or integration tests for review state transitions and AI service boundary (mocked).
- [ ] **P1** GitHub Actions CI pipeline.
- [ ] **P1** LangChain for chained generate → translate → summarize.

### Harness (Phase 1+, referenced here for traceability)

- [ ] **P0** (Phase 1) `AGENTS.md` defines agent entry point and project conventions.
- [ ] **P1** (Phase 1) `feature_list.json` tracks incremental feature completion.

---

## Edge Cases

| Scenario | Expected behavior |
|----------|-------------------|
| LLM API timeout or rate limit | Mutation fails with user-visible error; piece stays in prior state; no partial corrupt text saved |
| LLM returns empty or blocked content | Treat as failure; surface message; do not transition to `SuggestedByAI` |
| Invalid review transition (e.g. `Approved` → `Draft` without explicit reset) | Reject with GraphQL error and stable error code/message |
| Concurrent edits on same piece | Last-write-wins on text fields for v1; subscription notifies all clients of latest state |
| Campaign with zero content pieces | Dashboard shows empty state; queries return empty list, not error |
| Unsupported or missing locale | Reject translation request with validation error |
| Rejected piece | Remains `Rejected` until user resets to `Draft` or triggers new generation |
| Subscription client disconnects | Client reconnects and refetches; no data loss on server |
| Missing API keys at startup | Backend fails fast with clear log message; documented in README |

---

## Integrations

| Integration | Role | Notes |
|-------------|------|-------|
| **PostgreSQL** | Primary datastore | Campaigns, pieces, suggestions, optional review events |
| **OpenAI API** | LLM provider | Draft generation, translation (configurable default) |
| **Anthropic API** | LLM provider | Alternative or primary provider via env config |
| **Strawberry GraphQL** | API layer | Queries, mutations, subscriptions |
| **React + GraphQL client** | Frontend | Apollo Client or urql with subscription support |
| **Docker Compose** | Local orchestration | Backend, frontend, database services |

**Out of scope for v1:** Kafka, Redis, Kubernetes, ArgoCD, production SSO, billing.

**Configuration:** Provider choice, model names, and API keys via environment variables only (no secrets in repo).

---

## Timeline

Phased plan for solo training delivery. Dates are indicative — adjust in ClickUp.

| Phase | Scope | Target |
|-------|--------|--------|
| **0 — Bootstrap** | Context, handshake, PRD, TRD, ADRs | Complete (`86e1ykuyu`, `86e1ykv0j`; PRD in progress) |
| **1 — Harness** | `AGENTS.md`, optional `feature_list.json`, repo layout (`backend/`, `frontend/`) | Week 1 |
| **2 — Backend core** | Django models, GraphQL schema, CRUD, review state machine | Week 2 |
| **3 — AI layer** | OpenAI/Anthropic service, generate + translate mutations | Week 3 |
| **4 — Frontend** | Dashboard, review UI, GraphQL client | Week 4 |
| **5 — Real-time + Docker** | Subscriptions, `compose.yml`, README setup | Week 5 |
| **6 — Polish** | Tests, CI, P1 items as time allows | Week 6+ |

**MVP cut (minimum submission):** Phases 1–5 P0 criteria only.

---

## Approvals

| Role | Name | Date | Status |
|------|------|------|--------|
| Product owner / trainee | Ro (Rosario Santa Marina) | _pending_ | Awaiting review of this draft |

---

## Related

| Artifact | Path / link | Status |
|----------|-------------|--------|
| Challenge brief | [README.md](../README.md) | Source requirements |
| Output handshake | `~/.nan-ai-workspace/.../artifact-destinations.md` | Approved (local) |
| TRD | `docs/trd.md` | PHASE-0.4 (`86e1ykv43` or next TRD ticket) |
| ADR-001 | `docs/adr/ADR-001.md` | PHASE-0.5 |
| ADR-002 | `docs/adr/ADR-002.md` | PHASE-0.6 |
| ADR-003 | `docs/adr/ADR-003.md` | PHASE-0.7 (harness engineering) |
| Harness entry | `AGENTS.md` | PHASE-1 |
| ClickUp ticket | [86e1ykv42](https://app.clickup.com/t/86e1ykv42) | In progress |
