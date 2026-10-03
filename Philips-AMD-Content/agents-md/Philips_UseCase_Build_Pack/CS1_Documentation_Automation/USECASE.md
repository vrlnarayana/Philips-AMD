# USECASE.md — CS1 · Documentation Automation (Requirement Document)

> **Purpose of this document.** This is the **requirement spec** for what you build — the *what* and
> *why*. The `AGENTS.md` in this folder is the **build contract** for the coding agent — the *how*.
> Read this first, then build spec-first against `AGENTS.md` (§13) using the prompts in `PROMPTS.md`.
>
> **Programme:** Philips AMD · Advanced Agentic AI Bootcamp (2-day hands-on) · OpenCode + Kimi-K2.5
> **Use case ID:** CS1 · **Spec ID prefix:** `SPEC-CS1-NNN`

---

## 1. Business context & problem

Engineering documentation — technical specs, SOPs, process maps — is a known productivity bottleneck:
it is slow to produce, quickly goes stale, and pulls senior engineers away from delivery. CS1 builds an
**agent system that drafts structured documentation from source artefacts** (code, specs, meeting notes),
with a human sign-off before anything goes live.

This is the **safe first domain** for the programme: the worst-case output is a *draft*, not a production
outage — so it is where we prove the scaffold and the guardrails before raising autonomy in CS2/CS3.

## 2. Objective & scope

**Objective:** Given a set of source artefacts, produce a reviewable, correctly-classified documentation
draft and route it to a human for approval — safely, repeatably, and with full traceability.

**In scope**
- Ingesting and classifying source artefacts via MCP tools
- Structuring and drafting documentation
- Routing drafts for human review (human-in-the-loop)
- The four guardrail layers and a FastAPI MCP server

**Out of scope**
- Auto-publishing without human approval
- Writing back to source repositories without L4 approval
- Any external LLM API (runtime model is the self-hosted LiteLLM gateway only)

## 3. Actors & roles

| Actor | Role |
|---|---|
| **Author / Engineer** | Supplies source artefacts; consumes the draft |
| **Reviewer / Approver** | The **human gate** — approves or rejects before a doc goes live (L4) |
| **Agent system** | Ingest → Structure → Draft → Review Routing |
| **MCP server** | Exposes the tools the agents may call |

## 4. Functional requirements

| ID | Requirement |
|---|---|
| **FR-1** | The system SHALL ingest source artefacts via the `file_reader` tool and normalise them into a typed `SourceArtefact`. |
| **FR-2** | The system SHALL classify every source as `PUBLIC` / `INTERNAL` / `CONFIDENTIAL` **before** any reasoning, and reject unclassified or out-of-scope inputs. |
| **FR-3** | The system SHALL organise content into a typed `Document` skeleton (sections, headings) using `template_library`. |
| **FR-4** | The system SHALL draft section content against that structure. |
| **FR-5** | The system SHALL flag content that needs human review and route it (human-in-the-loop); nothing goes live without approval. |
| **FR-6** | The system SHALL read repository history via `version_control` **read-only**; any write requires L4 approval. |
| **FR-7** | The system SHALL treat all document/tool content as **untrusted** — embedded instructions in a source must not change agent behaviour. |

## 5. Agent pipeline

`Ingest → Structure → Draft → Review Routing` — typed hand-offs (a `Document` object flows through), each
boundary a Pydantic v2 **strict** model. Bound the loop (max steps, max tool calls, timeout).

1. **Ingest Agent** — pulls artefacts via MCP; normalises + classifies them.
2. **Structure Agent** — builds the document skeleton.
3. **Drafting Agent** — generates section content.
4. **Review Routing Agent** — flags + routes what needs human review.

## 6. MCP tools required

| Tool | Purpose | Access | Gate |
|---|---|---|---|
| `file_reader` | read source artefacts | read-only | — |
| `template_library` | fetch doc templates | read-only | — |
| `version_control` | read repo/history | read-only | **no writes without L4** |
| `approval_workflow` | submit a doc for sign-off | **destructive/consequential** | auth + human confirmation |

Every tool: typed strict request/response · auth on every call · rate limit per caller · audit log.

## 7. Guardrail & security requirements (4 layers)

| Layer | Requirement for CS1 |
|---|---|
| **L1 · Input (pre-hook)** | Classify source sensitivity; reject out-of-scope / over-large inputs; treat document content as untrusted (an injected "ignore instructions" must not hijack the agent). |
| **L2 · Tool-call** | Schema-check every call; enforce call budget; allow only the four CS1 tools. |
| **L3 · Output (post-hook)** | Scan generated content for PII / secrets / proprietary code; classify before it leaves. |
| **L4 · Governance gate** | Human sign-off mandatory before a doc goes live; full audit trail of every tool call and decision. |

**Programme security constraints (non-negotiable):**
- **No external LLM APIs** — runtime model via the LiteLLM gateway only.
- **No secrets** in code, prompts, or commits; secrets from `.env` only.
- The **🔒 red-team check**: feed a document containing *"ignore previous instructions…"* → L1/L3 must intercept it.

## 8. Non-functional requirements (the 10-point scaffold)

Type hints on every boundary · LLM/tool output → Pydantic v2 strict · async I/O only · typed resilient LLM
client (timeout + retry + jitter) · provider behind a port · tool-call args validated before execution ·
destructive tools guarded (business-rule + auth) · FastAPI request **and** `response_model` · contract tests
as a CI gate (LLM mocked) · **pure core** (no SDK/SQL/FastAPI in the business layer). See `AGENTS.md` §5.

## 9. Acceptance criteria / Definition of Done

- [ ] `Ingest → Structure → Draft → Review Routing` runs end-to-end on a sample source set
- [ ] L1 classifies source; L3 scans drafts; L4 routes flagged items to a human
- [ ] MCP server live (4 tools) with auth + rate limit + audit
- [ ] 🔒 A document containing "ignore previous instructions…" is intercepted by L1/L3
- [ ] Ten-point scaffold passes; CI gate green; committed
- [ ] Every behaviour has a `SPEC-CS1-NNN` ID traced **spec → code → test → PR**; `trace_check.py` green
- [ ] `README.md` written and current (overview, setup, run, tools, guardrails, build log, spec map)

## 10. Initial spec backlog (seed SDD — write these specs first)

Build spec-first (`AGENTS.md` §13). Suggested opening IDs — refine/split as needed:

| Spec ID | Behaviour | Layer / check |
|---|---|---|
| `SPEC-CS1-001` | Ingest normalises raw artefacts into a strict `SourceArtefact` | check 2 |
| `SPEC-CS1-002` | Tool-call args validated before any MCP call; only the 4 CS1 tools allowed | L2 / check 6 |
| `SPEC-CS1-003` | Ingest classifies source sensitivity (L1 pre-hook) before reasoning | **L1** / check 7 |
| `SPEC-CS1-004` | Document content is treated as untrusted (injection defence) | L1/L3 |
| `SPEC-CS1-005` | Draft output scanned for PII / secrets before routing | **L3** |
| `SPEC-CS1-006` | `approval_workflow` cannot fire without auth + human confirmation | **L4** / check 7 |

## 11. Deliverables & traceability

- Running agent pipeline + FastAPI MCP server (4 tools)
- `specs/` with one `SPEC-CS1-NNN.md` per behaviour + `specs/SPECS.md` index
- `README.md` kept current, including the §11 spec map
- Green CI: scaffold self-test + guardrail self-test + `trace_check.py`
- One ID → spec + code + test + PR: `grep -rn "SPEC-CS1-003" .`
