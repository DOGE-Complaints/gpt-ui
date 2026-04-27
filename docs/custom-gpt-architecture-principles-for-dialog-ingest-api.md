# Custom GPT Architecture Principles: Dialog Data Collection, Validation, Normalization & API Submission

**DOGEstonia status:** This document is aligned to the active Issue chain (`validation -> safety -> issue-policy-gate -> issue-normalizer -> api-orchestrator`) and to the current handoff artifact `normalized_issue_payload`.

**Purpose:** This document extracts the **technical architecture** of the GPT UI instruction system so it can be reproduced in another project. Same technical task—dialog-driven data collection, validation, normalization, and submission to an external API—with a **different data model and business context**. It also provides a **theoretical base for Custom GPT Actions**: what Actions are, what OpenAPI must contain, production limits, authentication modes, and how that layer fits **on top of** the instruction stack (§11–§16).

**Source:** Derived from `GPT UI/instructions/` (root, base, bootstrap, instruction-modules-index, ingest-validation, ingest-deep-parsing, **issue-normalizer**, api-orchestrator, safety-compliance, **issue-policy-gate**) and reference docs (**issue-data-model**, **issue-api-methods-reference**, communication-presets-reference). Donor legacy materials were removed from active runtime surface. **Actions/OpenAPI theory** is aligned with [openai-custom-gpt-actions-official-guide.md](./openai-custom-gpt-actions-official-guide.md) (official URLs and limits) and [gpt-actions-bot-api-auth-mapping.md](./gpt-actions-bot-api-auth-mapping.md). In this project the canonical **Issue** model and HTTP SSOT live in `story-data-model.md` + `story-api-methods-reference.md` + Issues OpenAPI YAML.

**Audience:** Teams building a Custom GPT that collects structured data via dialogue, validates it, normalizes it to a canonical schema, and sends it to an external API.

**DOGEstonia (2026-04-21):** основная цепочка инструкций переведена на **Issue** (`issue-*`, `issue-policy-gate`); legacy stubs removed from active `instructions/` runtime surface.

---

## 1. High-Level Architecture

### 1.1 Layered Instruction Hierarchy

The system is a **strict priority stack**. Lower layers never override higher ones.

| Priority | Layer | Role |
|----------|--------|------|
| 1 | **Root Wrapper** | Identity, constraints, routing order, "orchestrator not decision-maker", no verbal claims about backend decisions |
| 2 | **Base Instruction** | Operational modes, mode detection, handoff rules, lifecycle/state machine, strict protocol (artifacts, stop-the-line) |
| 3 | **Safety & Compliance** | Non-negotiable safety/legal boundaries; can interrupt any flow |
| 4 | **Functional modules** | Ingest Validation, Deep Parsing, Normalizer, Policy Gate, API Orchestrator, Search Dialogue |
| 5 | **Conversational output** | Final wording and presentation |

**Principles:**
- One source of truth per concern (Root = behavior, Base = modes + protocol, Data Model = schema, API Reference = endpoints).
- No invented rules; no merging of modules; no bypassing defined workflows.
- Exactly **one functional module** active at any step; modules are activated **sequentially**, never simultaneously.
- Context does not leak between modules except via **explicit handoff** (named artifacts).

### 1.2 Pre-Routing: Communication Bootstrap

Before any functional routing, a **Communication Bootstrap** runs (once per session):

- **Input:** First user message (or explicit commands later).
- **Output:** `comm_context` with: dialog language, tone/style preset, verbosity level, transparency (comfort vs debug).
- **Persistence:** Implicit—by stating parameters in the conversation so the model can read them from history.
- **Commands:** e.g. `/lang`, `/style`, `/brief`, `/normal`, `/detailed`, `/debug` to update context anytime.

Bootstrap is **before** mode detection and module activation. All responses then apply `comm_context` (language, tone, length, whether to show technical artifacts).

**Replication:** Define your own 4–5 communication parameters and a small command set; keep bootstrap in a dedicated instruction file referenced from Root.

---

## 2. Operational Modes

### 2.1 Mode Set

The system operates in **exactly one mode at a time**:

| Mode | Purpose | Delegation |
|------|---------|------------|
| **INGEST** | Create/update entity (collect, validate, normalize, submit) | Ingest Validation → (Deep Parsing) → Safety → (Policy Gate) → Normalizer → API Orchestrator |
| **SEARCH** | Read-only search/query | Search Dialogue → API Orchestrator |
| **HELP** | How/why/what, onboarding | Handled in Base (no delegation) |
| **POLICY** | Privacy, rules, principles | Handled in Base (no delegation) |

**Rules:**
- One user input → one mode; if ambiguous, ask before proceeding.
- No mixing modes in one step; if the user switches intent, acknowledge and switch mode.
- Mode persists until task completion or explicit change.

### 2.2 Mode Detection (Base Instruction)

- Analyze intent (verbs, keywords, patterns).
- Classify into exactly one mode.
- If ambiguous: ask one clarification question; do not guess.
- Activate mode and delegate to the corresponding module; Base does **not** parse, validate, or call APIs.

**Replication:** Keep mode set and detection in Base; only the labels and triggers are domain-specific (e.g. "add" vs "create event").

---

## 3. INGEST Flow: End-to-End Pipeline

The INGEST flow is the core technical pattern for **dialog data collection → validation → normalization → API**.

### 3.1 Pipeline Stages (Order)

1. **Input classification** (Base): Dialogue vs non-dialogue (e.g. link, image, PDF, long text).
2. **Deep Parsing** (optional): For non-dialogue only; produces structured extraction + confidence + ambiguities.
3. **Ingest Validation**: Validates against the data model; resolves missing/invalid fields via dialogue; produces validation report.
4. **Safety & Compliance**: At defined check points; can allow / redact / block.
5. **Policy Gate** (optional): Evaluates eligibility against policy; does not call API.
6. **Normalizer**: Transforms validated data into **canonical schema** for the API.
7. **API Orchestrator**: Sole module that calls the external API; reports backend response verbatim.

**Principles:**
- **Single responsibility per module:** e.g. Validation does not call API; Normalizer does not ask questions; API Orchestrator does not parse or validate.
- **Backend as source of truth:** Status and success come only from API responses; GPT must not claim outcomes the backend has not confirmed.
- **Explicit handoffs:** Each stage consumes and produces named artifacts (see below).

### 3.2 Input Types and Routing

- **Dialogue:** Short Q&A; Validation extracts and validates from conversation; no Deep Parsing.
- **Non-dialogue:** Link, image, PDF, long paste; Validation **activates Deep Parsing first**; validation runs on the parsing artifact.
- **One entity per input:** If multiple entities are detected (e.g. multiple images), reject and ask for one at a time.

**Replication:** Keep the same split (dialogue vs non-dialogue) and the rule “one entity per input”; only the entity name and detection cues are domain-specific.

---

## 4. Strict Protocol Mode (Discipline for INGEST)

To avoid “almost ready” or silent skips, INGEST can run in **Strict Protocol Mode**:

- **Mandatory artifacts:** Each stage produces a **versioned JSON artifact** before the next stage runs.
- **Stop-the-line rules:** Progression is blocked until conditions are met (e.g. no missing required fields, no safety block).
- **Review-first default:** Default target is “ready for review” (all required fields for that level); “draft only” only if the user explicitly asks.
- **Batch field requests:** Ask for **all** missing required fields in one go (with enum options where applicable), not one-by-one.
- **Deep Parsing pre-step:** For non-dialogue, do **not** ask clarifying questions until a `deep_parsing_artifact` exists; then ask from ambiguities/missing fields in batch.
- **Early enum extraction:** Deep Parsing tries to extract enums (e.g. format) early; if confidence is low, put in ambiguities and let Validation ask in batch.
- **No verbal claims about backend/gate:** GPT says “ready for submission”, “local validation PASS/FAIL”, “awaiting backend confirmation”—not “approved” or “published” until the API says so.
- **Audit trail:** After each step, record step name, artifact id, passed/blocked, next allowed steps, timestamp (e.g. in a small JSON block).

**Replication:** Keep the same protocol (artifacts, stop-the-line, batch requests, no verbal backend claims, audit trail); only artifact field names and readiness levels are schema-specific.

---

## 5. Artifacts (Contract Between Stages)

Artifacts are the **only** way stages pass data forward. Each has an id, version, and timestamp.

### 5.1 Canonical Artifact Set (Strict Protocol)

| Artifact | Produced by | Purpose |
|----------|-------------|--------|
| **deep_parsing_artifact** | Deep Parsing | Extracted fields, confidence, ambiguities, conflicts, PII flags, missing required fields (for non-dialogue) |
| **ingest_validation_report** | Ingest Validation | Readiness level, required-field status, missing/invalid fields, stop_the_line.blocked |
| **safety_compliance_report** | Safety & Compliance | decision (allow/redact/block), redactions, reasons, check_point |
| **gate_request_package** | Validation (before Gate) | validated_payload + refs to validation and safety reports (no gate decision inside) |
| **policy_gate_result** | Policy Gate | approval/rejection/clarification + reasons |
| **normalized_issue_payload** | Normalizer | canonical_payload + normalization_metadata (refs to validation/safety/gate) |
| **api_response** | Backend | Actual response; GPT reports it verbatim |

**Principles:**
- Each artifact has a **stable schema** (versioned).
- References between artifacts (e.g. validation_report_ref) link the chain.
- Stop-the-line: if e.g. `missing_required_fields[]` is not empty, do not advance to Normalizer/Gate/API.

**Replication:** Define one artifact per stage for your pipeline; keep the same roles (extraction → validation → safety → gate → normalization → API). Name the canonical output as `normalized_<entity>_payload` and keep report references explicit in normalization metadata.

---

## 6. Safety & Compliance Integration

Safety is **always on** and can interrupt any flow.

- **Activation points:** e.g. raw input, after extraction, after validation, after normalization (before API).
- At each point, produce a **safety_compliance_report** (decision: allow / redact / block).
- **Block:** Halt workflow; do not call API; explain to user.
- **Redact:** Require user to edit; re-run safety at same check point until allow or block.
- **Allow:** Proceed.

**Replication:** Keep the same activation points and report shape; only the rules (prohibited content, minors, etc.) are project-specific.

---

## 7. Policy Gate (Optional Admission Layer)

A **Policy Gate** sits between “validated + safe” and “normalize + API” when the business requires an eligibility check.

- **Input:** Validated payload + refs to validation and safety reports.
- **Output:** Admit / reject / needs_clarification + reasons and principle references.
- **Role:** Gate evaluates **eligibility** only; it does not call the API or decide final publication.
- **Single source of truth:** A policy document (e.g. PDF); Gate only evaluates against it, does not invent rules.

**Replication:** If your project has an admission policy, add one Gate module with the same contract (input package, output result with reasons). If not, skip Gate and go Validation → Normalizer → API.

---

## 8. Normalizer and API Orchestrator

### 8.1 Normalizer

- **Input:** Validated payload and policy result (`policy_gate_result`) on strict chain.
- **Output:** One canonical JSON payload (`normalized_issue_payload`) matching the **backend schema** (single source of truth: data model doc).
- **Responsibilities:** Normalize formats, enums, conditional fields, dates; no parsing, no questions, no API calls.
- **Failure:** If normalization fails, do not pass data to API Orchestrator; return to Validation.

**Replication:** One Normalizer per entity type; one canonical schema document (your “activity-data-model” equivalent).

### 8.2 API Orchestrator

- **Sole caller** of the external API for create/update/submit/search/etc.
- **Input:** Normalized payload (or search query from Search Dialogue).
- **Responsibilities:** Map to API methods, handle errors, retries, and translate system errors to user-facing messages.
- **Does not:** Parse input, validate, normalize, or decide policy.
- **Source of truth:** [`story-api-methods-reference.md`](../instructions/story-api-methods-reference.md) + imported Actions contract (from [`custom-gpt-story-intake-actions.openapi.yaml`](./custom-gpt-story-intake-actions.openapi.yaml)).

**Replication:** One API Orchestrator instruction; one api-methods-reference (or equivalent) as single source of truth. Same principle: “backend response is truth; report it verbatim.”

---

## 9. Data Model and References as Single Source of Truth

- **Data model:** One document that defines the full canonical schema (fields, enums, conditionals, required-by-stage). All parsing, validation, and normalization align with it.
- **API reference:** One document that defines endpoints, methods, request/response, errors. Only the API Orchestrator implements it.
- **Policy (if any):** One document for the Gate; Gate only evaluates against it.

**Replication:** In the other project, maintain: (1) entity data model, (2) API reference, (3) optional policy doc. Instructions only reference these; they do not redefine schema or API.

---

## 10. File and Module Layout (Reusable Structure)

Suggested instruction file set for the other project:

| File | Content |
|------|--------|
| **root.md** | Identity, hierarchy, “orchestrator not decision-maker”, no verbal backend claims, Bootstrap reference, module index reference |
| **base.md** | Modes, mode detection, input type (dialogue vs non-dialogue), strict protocol (artifacts, stop-the-line, batch requests), lifecycle/state machine, handoff rules |
| **bootstrap.md** | Communication parameters, detection algorithm, commands, integration with routing |
| **instruction-modules-index.md** | Short list of all modules, file names, one-line role |
| **communication-presets-reference.md** | Tone/verbosity/transparency presets and command mapping (if you keep bootstrap) |
| **ingest-validation.md** | Validation rules, readiness levels, batch missing-field requests, integration with Deep Parsing and Normalizer, output contract (validation report) |
| **ingest-deep-parsing.md** | Non-dialogue extraction, confidence, ambiguities, output contract (deep_parsing_artifact) |
| **entity-normalizer.md** | Canonical schema, normalization rules, input/output contract |
| **api-orchestrator.md** | When activated, input contract, API reference pointer, error handling, no invention of state |
| **safety-compliance.md** | Check points, report structure, allow/redact/block rules |
| **policy-gate.md** | (Optional) When activated, input package, policy doc reference, output result |
| **search flow contract** | (Optional) Query construction/handoff only when search operations exist in OpenAPI |
| **entity-data-model.md** | Canonical entity schema (your “activity-data-model”) |
| **story-api-methods-reference.md** | Issue HTTP SSOT for orchestrator behavior; keep in lockstep with Issues OpenAPI YAML |

Keep **Root** small (principle + references); put algorithms in **Base** and in each module file.

---

## 11. Custom GPT Actions: platform model (theory)

### 11.1 What Actions are

- A **Custom GPT** can attach **external REST APIs** through **Actions**.
- Mechanically, Actions build on **function calling**: the model (1) selects an operation, (2) fills **JSON arguments** from the user message and conversation context, (3) the **ChatGPT platform** executes a real **HTTP request** to your server and injects the response back into the chat.
- **OpenAPI 3.x** (JSON or YAML) is the **machine-readable contract**: it lists **paths**, **methods**, **parameters**, **request bodies**, **responses**, and **security**. The model relies heavily on **`operationId`**, **`summary`**, and **`description`** to decide **which** call to make and **how** to populate arguments—ambiguous or missing descriptions are a common cause of wrong or missing tool calls.

### 11.2 Apps vs Actions (product constraint)

- In a single GPT, OpenAI allows **either** first-party **Apps** **or** **Actions**—**not both** at the same time. Plan product UX accordingly.

### 11.3 Configuration surface in the GPT editor (conceptual)

1. **Actions** — create or edit an action set.  
2. **Authentication** — **None**, **API Key** (Basic / Bearer / custom header name), or **OAuth** (user login flow).  
3. **Schema** — paste, import from URL, or start from examples; must be valid **OpenAPI 3.x**.  
4. **Preview / Test** — per-operation test with request/response visibility.  
5. **Privacy policy URL** — required for **published** GPTs that use Actions.

**Canonical local compilation of official OpenAI points (URLs, limits, auth details):** [openai-custom-gpt-actions-official-guide.md](./openai-custom-gpt-actions-official-guide.md). Treat that file as the **citable** mirror of Help Center + developers.openai.com; this section is the **conceptual** bridge into your instruction architecture.

---

## 12. OpenAPI schema in Actions: structure and requirements

### 12.1 Elements the platform expects

| Element | Role |
|---------|------|
| **`openapi`** | Declares OpenAPI major version (3.x). |
| **`info`** | `title`, `version`; optional `description`. |
| **`servers`** | One or more **absolute** base URLs (`https://host…`). All paths in `paths` are resolved relative to the chosen server. |
| **`paths`** | For each path: supported **methods** (`get`, `post`, …), **parameters**, **requestBody**, **responses** (`200`, `4xx`, etc.). |
| **`operationId`** | Unique, stable name per operation—**Instructions** (e.g. API Orchestrator) should refer to these identifiers so the model’s behavior matches the schema. |
| **`summary` / `description`** | Primary natural-language signal for tool selection and argument filling. |

### 12.2 Production limits and ergonomics (verify on OpenAI’s current page)

Typical constraints documented by OpenAI (see [openai-custom-gpt-actions-official-guide.md](./openai-custom-gpt-actions-official-guide.md) §7):

- **TLS:** HTTPS, **TLS 1.2+**, port **443**, publicly trusted certificate.  
- **Timeout:** on the order of **45 seconds** round-trip.  
- **Payload size:** request and response bodies each up to roughly **100k characters** (text-oriented).  
- **Description length caps** for operation and parameter descriptions in the schema (exact numbers can change—sync from the official Production doc).  
- **Debugging:** reproduce the same URL, headers, and body in **Postman** (or equivalent) before blaming the model; use the editor’s **Test** on each operation.

### 12.3 `security`, `components.securitySchemes`, and common mistakes

- Declare **`components.securitySchemes`** (e.g. `type: http`, `scheme: bearer` for **API Key → Bearer** in the GPT UI).  
- Apply **`security`** on **every operation** that must send the credential. Omitting `security` on a protected operation causes **401** even if Authentication is configured in GPT UI, because the client may treat the operation as unsecured.  
- For DOGEstonia Issue Actions, keep operation-level security consistent between OpenAPI YAML and `story-api-methods-reference.md`; bearer policy details are in [gpt-actions-bot-api-auth-mapping.md](./gpt-actions-bot-api-auth-mapping.md).

### 12.4 Extension: `x-openai-isConsequential`

- Optional OpenAPI field **`x-openai-isConsequential`**: if **`true`**, ChatGPT **always** prompts the user before executing the call (no “always allow” shortcut). If omitted, defaults treat **GET** as generally non-consequential and other methods as consequential—confirm in the latest Production documentation.  
- Use for irreversible, financial, or safety-sensitive operations.

### 12.5 Custom headers limitation

- OpenAI **Production** documentation states that **custom headers are not supported** for Actions in the sense integrators often need (arbitrary extra headers beyond the configured auth mechanism).  
- Do not rely on **`X-User-Id`** (or similar) from the GPT Actions client unless the product adds a **proxy** or **API change**. **Project-specific mapping:** [gpt-actions-bot-api-auth-mapping.md](./gpt-actions-bot-api-auth-mapping.md).

---

## 13. Authentication modes (Actions UI ↔ HTTP)

| GPT UI mode | What is sent on the wire | Typical pairing |
|-------------|---------------------------|-----------------|
| **None** | No stored secret. | Dev or public endpoints that intentionally do not require auth. |
| **API Key** → **Bearer** | `Authorization: Bearer <secret>` | Shared secret channel for protected Issue routes; aligns with server-side `GPT_ACTIONS_BEARER_SECRET` policy. Not a user JWT. |
| **API Key** → Basic / Custom header | `Authorization: Basic …` or a named header | Only if your API is written for that shape. |
| **OAuth** | After user login: `Authorization: Bearer <access_token>` (**user** token) | Per-user backends that explicitly validate user JWTs on API side. |

OpenAI documents **encryption at rest** for stored API keys / client secrets; **OAuth Client ID** may be visible to end users—design branding and security accordingly.

---

## 14. Network, rate limiting, and observability

- **Egress IP ranges:** OpenAI publishes CIDR lists (e.g. `chatgpt-connectors.json`) for allowlisting—see the official Production page linked from [openai-custom-gpt-actions-official-guide.md](./openai-custom-gpt-actions-official-guide.md).  
- Return meaningful **HTTP status codes**; **429** and repeated **5xx** influence client backoff.  
- **ActionsGPT** (linked in the official guide) can help draft or refine schemas; still validate against your real API.

---

## 15. Instructions vs OpenAPI (how this architecture uses both)

- **OpenAPI** defines **what can be called** and **with which parameters**; **Instructions** (Root, Base, **API Orchestrator**) define **when** to delegate to the tool layer, **how** to map dialogue to payloads, and **how** to explain outcomes—without inventing backend state.  
- **Anti-patterns** (OpenAI Production): triggering actions without user-relevant intent; over-long conversational API responses when structured JSON suffices.  
- **Convention for this repo:** In workflow examples, **`Bearer <token>`** means the **Actions API Key (Bearer)** value—the same string as **`GPT_ACTIONS_BEARER_SECRET`** on the server—**unless** the text explicitly says OAuth / Logto user access token. See also §7.2.1 in `GPT UI/instructions/api-orchestrator.md`.

---

## 16. Wire-up: Custom GPT Actions ↔ DOGEstonia Issue API (practice)

When moving from “instructions only” to **live calls** against DOGEstonia Issue endpoints:

1. **Public HTTPS** base URL for the FastAPI app (staging, tunnel, or reverse proxy). Hosted Actions cannot target `http://127.0.0.1` on a laptop without a tunnel.  
2. **OpenAPI artifact:** import **`GET /openapi.json`** from the running app, then reconcile and lock it with [`custom-gpt-story-intake-actions.openapi.yaml`](./custom-gpt-story-intake-actions.openapi.yaml) and [`../instructions/story-api-methods-reference.md`](../instructions/story-api-methods-reference.md).  
3. **Authentication:** **API Key → Bearer** = **`GPT_ACTIONS_BEARER_SECRET`** for protected Issue routes. Follow [gpt-actions-bot-api-auth-mapping.md](./gpt-actions-bot-api-auth-mapping.md); do not reuse unrelated backend secrets.  
4. **Identity headers:** do not rely on arbitrary custom headers from Actions client; design endpoint contracts to work with supported auth/parameters.  
5. **Operator discipline:** keep YAML ↔ SSOT markdown ↔ orchestrator in lockstep before production import.

---

## 17. Checklist for Replication in Another Project

- [ ] **Root:** Identity, hierarchy, no verbal backend/gate claims, reference to Bootstrap and module index.
- [ ] **Base:** Modes (INGEST / SEARCH / HELP / POLICY or your set), mode detection, dialogue vs non-dialogue routing, one-entity-per-input, strict protocol (artifacts, stop-the-line, batch requests, audit trail).
- [ ] **Bootstrap:** Communication parameters and commands; run before mode detection.
- [ ] **Ingest flow:** Deep Parsing (non-dialogue) → Validation → Safety (at defined points) → [Gate] → Normalizer → API Orchestrator.
- [ ] **Artifacts:** Define one artifact per stage; version and id; use them as the only handoff mechanism.
- [ ] **Single source of truth:** Data model doc, API reference doc, optional policy doc; instructions only reference them.
- [ ] **Safety & Compliance:** Activation points and report format; allow/redact/block semantics.
- [ ] **Policy Gate (optional):** Input package, policy doc, output result with reasons.
- [ ] **Normalizer:** Input = validated (and gate result); output = canonical JSON; no parsing, no API.
- [ ] **API Orchestrator:** Only module calling API; reports backend response verbatim; uses API reference only.
- [ ] **Module index:** List of modules and their files so Root/Base stay maintainable.
- [ ] **Custom GPT Actions (if using live HTTP):** Public HTTPS; OpenAPI 3.x with **`servers`**, **`operationId`**, concise **descriptions** within platform limits; **`security`** on every path that the server actually protects; **Authentication** mode aligned with backend (e.g. shared Bearer secret); **Privacy policy URL** for published GPTs; understand **no arbitrary custom headers**—plan identity accordingly.

---

## 18. Summary: What Stays the Same vs What You Replace

| Aspect | Same in any project | Replace per project |
|--------|---------------------|----------------------|
| Hierarchy | Root → Base → Safety → Modules → Output | — |
| Modes | INGEST / SEARCH / HELP / POLICY (or equivalent) | Mode names/triggers if needed |
| Bootstrap | 4–5 communication params, commands | Language set, preset names |
| Strict protocol | Artifacts, stop-the-line, batch requests, no verbal backend claims | Artifact field names, readiness levels |
| Pipeline order | Deep Parsing → Validation → Safety → [Gate] → Normalizer → API | — |
| Safety | Check points, allow/redact/block, report shape | Actual safety rules |
| Gate | Input package, output result, policy doc | Policy content and doc |
| Data model | One canonical schema doc | Your entity schema |
| API | One reference doc; one Orchestrator | Your endpoints and methods |
| File layout | root, base, bootstrap, index, modules, data model, API reference | File names (e.g. entity-normalizer) |
| Actions / OpenAPI | Optional layer: OpenAPI 3.x + auth mode + platform limits; Instructions reference `operationId` | Your server URL, security scheme, and whether you use None vs API Key vs OAuth |

Using this, you can reproduce the same **technical architecture** (dialog → validate → normalize → API) in another Custom GPT with a different data model and business context.

---

## Document map (quick links)

| Topic | Where |
|--------|--------|
| Official OpenAI compilation (URLs, limits, auth, production) | [openai-custom-gpt-actions-official-guide.md](./openai-custom-gpt-actions-official-guide.md) |
| DOGEstonia: GPT UI auth ↔ backend zones | [gpt-actions-bot-api-auth-mapping.md](./gpt-actions-bot-api-auth-mapping.md) |
| Issue API SSOT (instructions) | [story-api-methods-reference.md](../instructions/story-api-methods-reference.md) |
| Issues OpenAPI YAML (build/import artifact) | [custom-gpt-story-intake-actions.openapi.yaml](./custom-gpt-story-intake-actions.openapi.yaml) |

**Version:** 1.2 · **Updated:** 2026-04-21 — aligned artifacts and Actions practice to DOGEstonia Issue chain (`normalized_issue_payload`, `policy_gate_result`, Issue SSOT/OpenAPI lockstep).
