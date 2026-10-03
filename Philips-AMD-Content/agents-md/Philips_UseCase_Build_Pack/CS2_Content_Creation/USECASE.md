# USECASE.md — CS2 · AI-Powered Content Creation (Requirement Document)

> **Purpose of this document.** This is the **requirement spec** for what you build — the *what* and
> *why*. The `AGENTS.md` in this folder is the **build contract** for the coding agent — the *how*.
> Read this first, then build spec-first against `AGENTS.md` (§13) using the prompts in `PROMPTS.md`.
>
> **Programme:** Philips AMD · Advanced Agentic AI Bootcamp (2-day hands-on) · OpenCode + Kimi-K2.5
> **Use case ID:** CS2 · **Spec ID prefix:** `SPEC-CS2-NNN`

---

## 1. Business context & problem

Engineering content — release notes, technical docs, training material, internal comms — is high-volume
and repetitive, which makes it an immediate productivity win for agents. But content is also
**brand-facing and factual**: an off-brand tone or an unverified claim shipped at scale is a real risk.
CS2 builds an **agent system that turns source inputs into content, then refines it**, with
**brand-style and factual-accuracy checks** before anything is published.

This use case raises the stakes from CS1: the output is meant to be *consumed*, so the guardrails move
from "classify the draft" to "block off-brand or inaccurate output before it goes live."

## 2. Objective & scope

**Objective:** Given a content request and reference material, produce on-brand, fact-checked content and
route it to a human for sign-off before publish — with every unverified claim flagged, never shipped silently.

**In scope**
- Interpreting a request into a structured brief
- Drafting content against the brief
- Enforcing tone/voice/brand guidelines
- Fact-checking and flagging unverified claims
- The four guardrail layers and a FastAPI MCP server

**Out of scope**
- Auto-publishing without human sign-off
- Fetching unapproved assets or sources
- Any external LLM API (runtime model is the self-hosted LiteLLM gateway only)

## 3. Actors & roles

| Actor | Role |
|---|---|
| **Requester** | Submits the content request / brief |
| **Brand / Editorial owner** | The **human gate** — signs off before publish (L4) |
| **Agent system** | Brief → Draft → Style/Brand → Fact-Check |
| **MCP server** | Exposes the tools the agents may call |

## 4. Functional requirements

| ID | Requirement |
|---|---|
| **FR-1** | The system SHALL interpret a request into a structured, validated `ContentBrief` before drafting. |
| **FR-2** | The system SHALL generate content against the brief as a typed `ContentPiece`. |
| **FR-3** | The system SHALL enforce tone, voice, and brand guidelines using `style_guide_library` (a transformation + a check). |
| **FR-4** | The system SHALL verify factual claims and **flag any unverified claim** — never ship it silently. |
| **FR-5** | The system SHALL block off-brand output at L3 before it can reach publish. |
| **FR-6** | The system SHALL fetch only **approved** assets/reference material via read-only tools, treated as **untrusted**. |
| **FR-7** | The system SHALL require human sign-off (L4) before any content goes live. |

## 5. Agent pipeline

`Brief → Draft → Style/Brand → Fact-Check` — typed hand-offs (a `ContentPiece` object flows through), each
boundary a Pydantic v2 **strict** model. Bound the loop (max steps, max tool calls, timeout).

1. **Brief Agent** — interprets the request into a structured content brief.
2. **Drafting Agent** — generates content against the brief.
3. **Style/Brand Agent** — enforces tone, voice, and brand guidelines.
4. **Fact-Check & Review Agent** — verifies claims; flags anything unverified.

## 6. MCP tools required

| Tool | Purpose | Access | Gate |
|---|---|---|---|
| `knowledge_base_reader` | read reference material | read-only | — |
| `style_guide_library` | fetch brand/style rules | read-only | — |
| `asset_fetcher` | fetch approved assets | read-only | — |
| `publish_workflow` | push content live | **destructive/consequential** | auth + human confirmation |

Every tool: typed strict request/response · auth on every call · rate limit per caller · audit log.

## 7. Guardrail & security requirements (4 layers)

| Layer | Requirement for CS2 |
|---|---|
| **L1 · Input (pre-hook)** | Check brief completeness + scope; classify source/prompt; treat reference material as untrusted. |
| **L2 · Tool-call** | Schema-check every call; enforce call budget; tool allowlist. |
| **L3 · Output (post-hook)** | **Brand-safety check** (tone/voice) **+ factual-accuracy scan** + PII/secret scan; overreliance guard — unverified claims flagged, never shipped. |
| **L4 · Governance gate** | Human sign-off mandatory before publish; full audit trail. |

**Programme security constraints (non-negotiable):**
- **No external LLM APIs** — runtime model via the LiteLLM gateway only.
- **No secrets** in code, prompts, or commits; secrets from `.env` only.
- The **🔒 red-team check**: request content that violates brand guidelines → the brand gate must hold (and a planted unverifiable claim must be flagged).

## 8. Non-functional requirements (the 10-point scaffold)

Type hints on every boundary · LLM/tool output → Pydantic v2 strict · async I/O only · typed resilient LLM
client (timeout + retry + jitter) · provider behind a port · tool-call args validated before execution ·
destructive tools guarded · FastAPI request **and** `response_model` · contract tests as a CI gate (LLM
mocked) · **pure core**. See `AGENTS.md` §5.

## 9. Acceptance criteria / Definition of Done

- [ ] `Brief → Draft → Style/Brand → Fact-Check` runs end-to-end on a sample request
- [ ] L3 brand-safety + fact-check gates fire before output; L4 routes to human sign-off
- [ ] MCP server live (4 tools) with auth + rate limit + audit
- [ ] 🔒 A request for off-brand content is blocked by the brand gate; a planted unverifiable claim is flagged
- [ ] Ten-point scaffold passes; CI gate green; committed
- [ ] Every behaviour has a `SPEC-CS2-NNN` ID traced **spec → code → test → PR**; `trace_check.py` green
- [ ] `README.md` written and current (overview, setup, run, tools, guardrails, build log, spec map)

## 10. Initial spec backlog (seed SDD — write these specs first)

Build spec-first (`AGENTS.md` §13). Suggested opening IDs — refine/split as needed:

| Spec ID | Behaviour | Layer / check |
|---|---|---|
| `SPEC-CS2-001` | Brief is validated for completeness + scope before drafting | L1 / check 2 |
| `SPEC-CS2-002` | Tool-call args validated; only the 4 CS2 tools allowed | L2 / check 6 |
| `SPEC-CS2-003` | Reference material treated as untrusted | L1 |
| `SPEC-CS2-004` | Unverified claims flagged, never shipped silently | **L3** |
| `SPEC-CS2-005` | Brand-safety gate blocks off-brand output before publish | **L3** |
| `SPEC-CS2-006` | `publish_workflow` cannot fire without auth + human confirmation | **L4** / check 7 |

## 11. Deliverables & traceability

- Running agent pipeline + FastAPI MCP server (4 tools)
- `specs/` with one `SPEC-CS2-NNN.md` per behaviour + `specs/SPECS.md` index
- `README.md` kept current, including the §11 spec map
- Green CI: scaffold self-test + guardrail self-test + `trace_check.py`
- One ID → spec + code + test + PR: `grep -rn "SPEC-CS2-005" .`
