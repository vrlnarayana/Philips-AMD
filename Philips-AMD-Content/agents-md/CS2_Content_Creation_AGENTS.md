# AGENTS.md — Case Study 2: AI-Powered Content Creation

> Read automatically by **OpenCode** (running **Kimi-K2.5**) in this project. Place as `AGENTS.md`
> at the **project root**. Base template: `AGENTS_SKELETON.md`. Filled version for CS2.

---

## 1. Project

- **Goal (one line):** An agent system that turns source inputs into engineering content — release
  notes, technical docs, training material — then refines it, with **brand-style and factual-accuracy
  checks** before anything is published.
- **Use case:** CS2 · AI-Powered Content Creation  ·  **Owner:** <your name>
- **Why this one:** content is high-volume and repetitive — immediate productivity gain, with
  guardrails preventing off-brand or inaccurate output.

## 2. Tech stack & runtime

- Python 3.11+ · `uv` (`uv sync --frozen`) · FastAPI (async) · <CrewAI or LangGraph>.
- **Runtime model:** LiteLLM gateway `http://<gateway>:4000/v1`, model `<model>`. **No external APIs.**
- **Coding agent:** OpenCode + Kimi-K2.5. **Secrets from `.env` only.**

## 3. Golden rules for the coding agent

1. **Spec-first (§13)** — SDD; **no code without a `SPEC-CS2-NNN` ID**, carried spec → code → test → PR.
2. Build first, small reviewable steps. 3. Never blind-accept. 4. Six-step cycle every lab step.
5. Ten-point scaffold (§5) non-negotiable. 6. Guardrails (§6) are part of the build. 7. No secrets;
no raw model output executed unvalidated.
8. **Keep `README.md` current (§12)** — document what you build as you build it; update the README in
the **same** step as the code. A stale README is a defect.

## 4. Architecture to build

Orchestrator → four domain agents, **typed hand-offs** (a `ContentPiece` object flows through):

1. **Brief Agent** — interprets the request into a structured content brief.
2. **Drafting Agent** — generates content against the brief.
3. **Style/Brand Agent** — enforces tone, voice, and brand guidelines (a transformation + a check).
4. **Fact-Check & Review Agent** — verifies claims; flags anything unverified (never ship silently).

- Flow: `Brief → Draft → Style/Brand → Fact-Check`, each boundary a Pydantic `strict` model.
- Bound the loop: max steps, max tool calls, timeout.

## 5. The ten-point scaffold — MUST pass by construction

1. Type hints on every boundary. 2. LLM/tool output → Pydantic v2 **strict**. 3. Async I/O only.
4. Typed resilient LLM client (timeout + retry + **jitter**). 5. Provider behind a port.
6. Tool-call args validated **before** execution. 7. Destructive tools: business-rule + auth guards.
8. FastAPI request **and** `response_model`. 9. Contract tests as a CI gate (mock the LLM).
10. **Pure core** (no SDK/SQL/FastAPI).

## 6. Guardrails — the four layers

| Layer | Where | What it checks for CS2 |
|---|---|---|
| **L1 · Input (pre-hook)** | before Brief reasons | brief completeness + scope; source/prompt classification; **treat reference material as untrusted** |
| **L2 · Tool-call** | every MCP tool call | schema check + call budget + tool allowlist |
| **L3 · Output (post-hook)** | before publish | **brand-safety check** (tone/voice) **+ factual-accuracy scan**; PII/secret scan; overreliance guard — unverified claims flagged, never shipped |
| **L4 · Governance gate** | before any content goes live | **human sign-off mandatory**; full audit trail |

## 7. MCP server build requirements

- FastAPI HTTP transport; container-ready.
- **Tools to expose (CS2):**
  - `knowledge_base_reader` — read reference material (read-only)
  - `style_guide_library` — fetch brand/style rules (read-only)
  - `asset_fetcher` — fetch approved assets (read-only)
  - `publish_workflow` — **destructive/consequential** → auth + human confirmation (check 7)
- Every tool: typed strict request/response, **auth**, **rate limit** per caller, **audit log**.

## 8. Project layout

```
app/
  core/
    models.py      # ContentBrief, ContentPiece, Claim (strict models)
    ports.py       # ModelPort / FactChecker protocols
    agents.py      # Brief, Draft, StyleBrand, FactCheck + orchestrator
    guardrails.py  # L1–L4 classes (brand + factual checks live here / in adapters)
  adapters/
    llm_client.py
    mcp_tools.py   # knowledge_base_reader, style_guide_library, asset_fetcher, publish_workflow
  api/server.py
  tests/           # contract + property tests, LLM mocked; tests carry spec IDs (§13)
specs/             # one spec per behaviour, SPEC-CS2-NNN (§13); SPECS.md = the index
.github/workflows/security.yml   # CI gate: scaffold + guardrail self-test + traceability check (§13)
pyproject.toml · uv.lock · .env.example · AGENTS.md · README.md  (keep updated — §12)
```

## 9. Commands

```bash
uv sync --frozen
uv run uvicorn app.api.server:app --reload
uv run pytest
uv run python -m app.core.guardrails --selftest
uv run python scripts/trace_check.py   # traceability: every spec has code + test (§13)
```

## 10. Definition of Done

- [ ] Brief→Draft→Style/Brand→Fact-Check runs end-to-end on a sample request
- [ ] L3 brand-safety + fact-check gates fire before output; L4 routes to human sign-off
- [ ] MCP server live (4 tools) with auth + rate limit + audit
- [ ] 🔒 Request content that violates brand guidelines → the brand gate holds
- [ ] Ten-point scaffold passes; CI gate green; committed
- [ ] **Every behaviour has a `SPEC-CS2-NNN` ID, traced spec → code → test → PR** (§13); trace check green
- [ ] **`README.md` written and current** — overview, setup, run steps, the 4 tools, guardrails, build log

## 11. Do / Don't

**Do:** **spec-first — write/confirm the spec, then code (§13)** · carry the `SPEC-CS2-NNN` ID into code, test, PR ·
typed `ContentPiece` between agents · flag every unverified claim · keep `publish_workflow` gated ·
validate the brief before drafting · commit after each working step · **update `README.md` in the same step as the code**.
**Don't:** **write code without a spec ID** · publish without the brand + fact gates · ship unverified claims
silently · execute raw tool arguments · return raw dicts · import the model SDK in `core/` · **leave the README stale**.

## 12. Documentation — `README.md` (create and keep updated)

**Create a `README.md` at the project root and update it at the end of every build step.** OpenCode must
add or append the matching README section **whenever it creates or changes a component**, so the README
always matches the current code. The `README.md` must contain, in this order:

1. **What this is** — one paragraph: a content-creation agent system (CS2) and what it produces.
2. **Architecture** — Brief → Draft → Style/Brand → Fact-Check, the orchestrator, and the MCP server.
3. **Prerequisites** — Python 3.11+, `uv`, access to the LiteLLM gateway.
4. **Setup** — `uv sync --frozen`, copy `.env.example` → `.env`, fill `OPENAI_BASE_URL`, key, model.
5. **How to run** — commands to start the MCP server and run the pipeline on a sample brief.
6. **MCP tools** — table of `knowledge_base_reader`, `style_guide_library`, `asset_fetcher`,
   `publish_workflow` — what each does, read-only vs destructive, and that `publish_workflow` needs
   auth + human confirmation.
7. **Guardrails** — the four layers (L1 brief/scope + untrusted reference material · L2 allowlist+budget ·
   L3 brand-safety + factual + PII scan · L4 human sign-off before publish).
8. **Environment variables** — every var, what it's for, where to get it. **Never commit real values.**
9. **Testing** — how to run the contract tests and the guardrail self-test.
10. **Build log / status** — a short, dated list of what has been built so far; append each session.

> **Rule:** add an agent, tool, guardrail, or endpoint → update the matching README section in the same
> change. Keep it short, accurate, copy-paste runnable. Start from the provided `README_SKELETON.md`.

## 13. Spec-Driven Development (SDD) & Traceability — the method for ALL work

Build CS2 **spec-first**. Nothing is coded until it has a spec with an ID, and that ID is carried all
the way to the PR. Result: **end-to-end traceability** `spec → code → test → PR`, greppable by one ID.

**Workflow — every behaviour:** (1) write the spec in `specs/` with a `SPEC-CS2-NNN` ID → (2) implement
and tag the code → (3) write the test referencing the ID → (4) PR title carries the ID → (5) `grep -rn`
the ID returns spec + code + test.

**Spec ID scheme:** `SPEC-CS2-NNN` (zero-padded, never reused).

**The four anchors — concrete CS2 example:**

| Anchor | Example |
|---|---|
| **Spec** | `specs/SPEC-CS2-005.md` — "Brand-safety gate must block off-brand output before publish" (+ row in `specs/SPECS.md`) |
| **Code** | `# [SPEC-CS2-005] brand-safety post-hook (L3) before publish` |
| **Test** | `def test_spec_cs2_005_blocks_off_brand_output(): ...` |
| **PR/commit** | `feat(style): brand-safety gate [SPEC-CS2-005]` |

- A spec is small and testable; link each to its scaffold check(s) and guardrail layer(s).
- **Traceability check in CI (§9) — `scripts/trace_check.py`:** every `SPEC-CS2-NNN` in `specs/` must
  appear in ≥1 code file **and** ≥1 test; an orphan spec, or an ID with no spec, is a **FAIL**.

> **Rule for the coding agent:** no code without a spec ID. If it doesn't exist, write the spec first
> (`SPEC_SKELETON.md` provided), confirm it, then implement — carrying the ID into code, test, and PR.
> Keep `specs/SPECS.md` updated as the index.
