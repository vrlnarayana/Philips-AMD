# AGENTS.md — Case Study 3: Self-Healing AI System

> Read automatically by **OpenCode** (running **Kimi-K2.5**) in this project. Place as `AGENTS.md`
> at the **project root**. Base template: `AGENTS_SKELETON.md`. Filled version for CS3.
> ⚠ This is the **highest-autonomy** case — the Layer-4 human gate is the load-bearing wall.

---

## 1. Project

- **Goal (one line):** An agent system that monitors a running service, detects anomalies, diagnoses
  root cause, and **applies or proposes a validated fix — under a human gate for any production change.**
- **Use case:** CS3 · Self-Healing AI System  ·  **Owner:** <your name>
- **Why this one:** self-healing cuts downtime and on-call load, but it is the **highest-trust**
  automation — responsible autonomy means a hard stop before any consequence.

## 2. Tech stack & runtime

- Python 3.11+ · `uv` (`uv sync --frozen`) · FastAPI (async) · <LangGraph> for the monitor→fix loop.
- **Runtime model:** LiteLLM gateway `http://<gateway>:4000/v1`, model `<model>`. **No external APIs.**
- **Coding agent:** OpenCode + Kimi-K2.5. **Secrets from `.env` only.**

## 3. Golden rules for the coding agent

1. **Spec-first (§13)** — SDD; **no code without a `SPEC-CS3-NNN` ID**, carried spec → code → test → PR.
2. Build first, small reviewable steps. 3. Never blind-accept. 4. Six-step cycle every lab step.
5. Ten-point scaffold (§5) non-negotiable. 6. Guardrails (§6) are part of the build.
7. **The Remediation agent PROPOSES; it never applies a production change on its own.**
8. **Keep `README.md` current (§12)** — document what you build as you build it; update the README in
the **same** step as the code. A stale README is a defect.

## 4. Architecture to build

Orchestrator → four domain agents, **typed hand-offs** (an `Incident` object flows through):

1. **Monitor Agent** — watches logs/metrics; detects anomalies.
2. **Diagnosis Agent** — reasons about root cause from the signals.
3. **Remediation Agent** — **proposes** a fix with rationale; **cannot apply it alone**.
4. **Validation Agent** — confirms the fix worked, **after** human approval.

- Flow: `Monitor → Diagnose → Remediate(propose) → [HUMAN GATE] → apply → Validate`.
- Bound the loop: max steps, max tool calls, timeout; **safe-action allowlist** for anything executable.

## 5. The ten-point scaffold — MUST pass by construction

1. Type hints on every boundary. 2. LLM/tool output → Pydantic v2 **strict**. 3. Async I/O only.
4. Typed resilient LLM client (timeout + retry + **jitter**). 5. Provider behind a port.
6. Tool-call args validated **before** execution. 7. **Destructive tools: business-rule + auth guards
(this is the crux of CS3).** 8. FastAPI request **and** `response_model`. 9. Contract tests as a CI gate.
10. **Pure core.**

## 6. Guardrails — the four layers

| Layer | Where | What it checks for CS3 |
|---|---|---|
| **L1 · Input (pre-hook)** | before Monitor/Diagnose reasons | validate + classify incoming signals; reject malformed/over-broad alerts; treat log content as untrusted |
| **L2 · Tool-call** | every MCP tool call | schema check + call budget + **safe-action allowlist** (no tool outside the allowlist runs) |
| **L3 · Output (post-hook)** | before a remediation is surfaced | every proposed fix validated + logged; no secrets in the proposal; confidence scored |
| **L4 · Governance gate** | **before ANY production change** | **human approval is mandatory and non-skippable**; full audit chain: signal → diagnosis → proposed fix → approver |

- **Human-Agent trust exploitation is the live risk:** a polished-but-wrong remediation must be caught
  at L4. The gate defends against a confident, plausible, harmful proposal.

## 7. MCP server build requirements

- FastAPI HTTP transport; container-ready.
- **Tools to expose (CS3):**
  - `log_metrics_reader` — read logs/metrics (read-only)
  - `alerting_hook` — read/ack alerts (read-only / low-impact)
  - `runbook_executor` — **destructive/consequential** → safe-action allowlist + auth + **human confirmation** (check 7)
  - `ticket_rollback` — **destructive/consequential** → auth + human confirmation + rollback path (check 7)
- Every tool: typed strict request/response, **auth**, **rate limit**, **audit log**.
- **No executable tool (`runbook_executor`, `ticket_rollback`) fires without passing L4.**

## 8. Project layout

```
app/
  core/
    models.py      # Signal, Incident, Diagnosis, ProposedFix (strict models)
    ports.py       # ModelPort / Executor protocols
    agents.py      # Monitor, Diagnose, Remediate, Validate + orchestrator
    guardrails.py  # L1–L4; L4 gate + safe-action allowlist live here
  adapters/
    llm_client.py
    mcp_tools.py   # log_metrics_reader, alerting_hook, runbook_executor, ticket_rollback
  api/server.py
  tests/           # incl. a test that a proposed fix CANNOT apply without approval; tests carry spec IDs (§13)
specs/             # one spec per behaviour, SPEC-CS3-NNN (§13); SPECS.md = the index
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

- [ ] Monitor→Diagnose→Remediate(propose)→Validate runs end-to-end on a simulated incident
- [ ] **No production change applies without passing the L4 human gate** (prove it in a test)
- [ ] Safe-action allowlist enforced on `runbook_executor` / `ticket_rollback`
- [ ] MCP server live (4 tools) with auth + rate limit + audit
- [ ] 🔒 Inject a plausible-but-harmful remediation → L4 stops it; audit chain shows why
- [ ] Ten-point scaffold passes; CI gate green; committed
- [ ] **Every behaviour has a `SPEC-CS3-NNN` ID, traced spec → code → test → PR** (§13); trace check green
- [ ] **`README.md` written and current** — overview, setup, run steps, the 4 tools, guardrails,
  the human-gate rule, and the build log

## 11. Do / Don't

**Do:** **spec-first — write/confirm the spec, then code (§13)** · carry the `SPEC-CS3-NNN` ID into code, test, PR ·
keep Remediation at "propose + rationale" · gate every executable tool behind L4 + allowlist ·
log the full signal→fix→approver chain · test that approval is non-skippable · **update `README.md` in the same step as the code**.
**Don't:** **write code without a spec ID** · let any agent apply a production change autonomously · run a tool
outside the allowlist · trust a confident proposal without the gate · return raw dicts · import the SDK in `core/` · **leave the README stale**.

## 12. Documentation — `README.md` (create and keep updated)

**Create a `README.md` at the project root and update it at the end of every build step.** OpenCode must
add or append the matching README section **whenever it creates or changes a component**, so the README
always matches the current code. The `README.md` must contain, in this order:

1. **What this is** — one paragraph: a self-healing agent system (CS3) and what it does.
2. **Architecture** — Monitor → Diagnose → Remediate (propose) → [human gate] → apply → Validate,
   the orchestrator, and the MCP server.
3. **Prerequisites** — Python 3.11+, `uv`, access to the LiteLLM gateway.
4. **Setup** — `uv sync --frozen`, copy `.env.example` → `.env`, fill `OPENAI_BASE_URL`, key, model.
5. **How to run** — commands to start the MCP server and run the loop on a simulated incident.
6. **MCP tools** — table of `log_metrics_reader`, `alerting_hook`, `runbook_executor`, `ticket_rollback`
   — what each does, read-only vs destructive, the safe-action allowlist, and that the executable tools
   need auth + **human confirmation (L4)**.
7. **Guardrails** — the four layers, with emphasis that **no production change applies without passing
   the L4 human gate**; document the audit chain (signal → diagnosis → proposed fix → approver).
8. **Environment variables** — every var, what it's for, where to get it. **Never commit real values.**
9. **Testing** — how to run the contract tests, the guardrail self-test, and the test proving approval
   is non-skippable.
10. **Build log / status** — a short, dated list of what has been built so far; append each session.

> **Rule:** add an agent, tool, guardrail, or endpoint → update the matching README section in the same
> change. The README must make the human-gate rule unmissable. Start from the provided `README_SKELETON.md`.

## 13. Spec-Driven Development (SDD) & Traceability — the method for ALL work

Build CS3 **spec-first**. Nothing is coded until it has a spec with an ID, and that ID is carried all
the way to the PR. Result: **end-to-end traceability** `spec → code → test → PR`, greppable by one ID.
For this high-autonomy system, traceability is also your audit story — the human gate must trace back
to the spec that mandated it.

**Workflow — every behaviour:** (1) write the spec in `specs/` with a `SPEC-CS3-NNN` ID → (2) implement
and tag the code → (3) write the test referencing the ID → (4) PR title carries the ID → (5) `grep -rn`
the ID returns spec + code + test.

**Spec ID scheme:** `SPEC-CS3-NNN` (zero-padded, never reused).

**The four anchors — concrete CS3 example:**

| Anchor | Example |
|---|---|
| **Spec** | `specs/SPEC-CS3-007.md` — "No production change applies without passing the L4 human gate" (+ row in `specs/SPECS.md`) |
| **Code** | `# [SPEC-CS3-007] L4 governance gate — block apply without human approval` |
| **Test** | `def test_spec_cs3_007_apply_blocked_without_approval(): ...` |
| **PR/commit** | `feat(governance): non-skippable human gate [SPEC-CS3-007]` |

- A spec is small and testable; link each to its scaffold check(s) and guardrail layer(s).
- The **non-skippable-approval** test (§10) is the test for `SPEC-CS3-007` — name it with the ID.
- **Traceability check in CI (§9) — `scripts/trace_check.py`:** every `SPEC-CS3-NNN` in `specs/` must
  appear in ≥1 code file **and** ≥1 test; an orphan spec, or an ID with no spec, is a **FAIL**.

> **Rule for the coding agent:** no code without a spec ID. If it doesn't exist, write the spec first
> (`SPEC_SKELETON.md` provided), confirm it, then implement — carrying the ID into code, test, and PR.
> Keep `specs/SPECS.md` updated as the index.
