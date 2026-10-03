# USECASE.md — CS3 · Self-Healing AI System (Requirement Document)

> **Purpose of this document.** This is the **requirement spec** for what you build — the *what* and
> *why*. The `AGENTS.md` in this folder is the **build contract** for the coding agent — the *how*.
> Read this first, then build spec-first against `AGENTS.md` (§13) using the prompts in `PROMPTS.md`.
>
> ⚠ **Highest-autonomy case.** The Layer-4 human gate is the load-bearing wall — build it **first**.
>
> **Programme:** Philips AMD · Advanced Agentic AI Bootcamp (2-day hands-on) · OpenCode + Kimi-K2.5
> **Use case ID:** CS3 · **Spec ID prefix:** `SPEC-CS3-NNN`

---

## 1. Business context & problem

Operational incidents cost downtime and on-call load. A self-healing system can detect anomalies,
diagnose root cause, and fix them faster than a human — but it is also the **highest-trust** automation in
the programme: a wrong action here is a *production consequence*, not a draft. CS3 builds an **agent system
that monitors a running service, diagnoses issues, and proposes or applies a validated fix — under a human
gate for any production change.**

This is where **responsible autonomy** is proven: the system is allowed to reason and *propose* freely, but
a hard stop sits before any consequence. Capability is high; the gate is non-negotiable.

## 2. Objective & scope

**Objective:** On a simulated incident, detect → diagnose → **propose** a remediation with rationale, and
apply it **only after human approval**, then validate — with a complete audit chain.

**In scope**
- Monitoring logs/metrics and detecting anomalies
- Diagnosing root cause
- Proposing a remediation (never auto-applying in production)
- Validating a fix after approval
- Safe-action allowlist, the four guardrail layers, a FastAPI MCP server

**Out of scope**
- Any agent applying a production change autonomously
- Running any executable action outside the safe-action allowlist
- Any external LLM API (runtime model is the self-hosted LiteLLM gateway only)

## 3. Actors & roles

| Actor | Role |
|---|---|
| **SRE / On-call engineer** | The **approver** — the mandatory human gate before any production change (L4) |
| **Monitored service** | The system under watch (simulated for the lab) |
| **Agent system** | Monitor → Diagnose → Remediate (propose) → Validate |
| **MCP server** | Exposes the tools the agents may call |

## 4. Functional requirements

| ID | Requirement |
|---|---|
| **FR-1** | The system SHALL watch logs/metrics via `log_metrics_reader` and detect anomalies into a typed `Incident`. |
| **FR-2** | The system SHALL diagnose root cause from the signals. |
| **FR-3** | The Remediation agent SHALL **propose** a fix with rationale and a confidence score; it SHALL NOT apply a production change on its own. |
| **FR-4** | The system SHALL block any apply until a human approves (L4); approval SHALL be **non-skippable**. |
| **FR-5** | The Validation agent SHALL confirm the fix worked **only after** human approval. |
| **FR-6** | Executable tools (`runbook_executor`, `ticket_rollback`) SHALL run only via a **safe-action allowlist** + auth + human confirmation. |
| **FR-7** | The system SHALL record the full chain: **signal → diagnosis → proposed fix → approver**. |

## 5. Agent pipeline

`Monitor → Diagnose → Remediate(propose) → [HUMAN GATE] → apply → Validate` — typed hand-offs (an
`Incident` object flows through). Bound the loop (max steps, max tool calls, timeout) and enforce a
safe-action allowlist for anything executable.

1. **Monitor Agent** — watches logs/metrics; detects anomalies.
2. **Diagnosis Agent** — reasons about root cause.
3. **Remediation Agent** — **proposes** a fix with rationale; cannot apply it alone.
4. **Validation Agent** — confirms the fix worked, **after** human approval.

## 6. MCP tools required

| Tool | Purpose | Access | Gate |
|---|---|---|---|
| `log_metrics_reader` | read logs/metrics | read-only | — |
| `alerting_hook` | read/ack alerts | read-only / low-impact | — |
| `runbook_executor` | run a remediation step | **destructive/consequential** | safe-action allowlist + auth + human confirmation (L4) |
| `ticket_rollback` | roll back a change | **destructive/consequential** | auth + human confirmation + rollback path (L4) |

Every tool: typed strict request/response · auth · rate limit · audit log.
**No executable tool fires without passing L4.**

## 7. Guardrail & security requirements (4 layers)

| Layer | Requirement for CS3 |
|---|---|
| **L1 · Input (pre-hook)** | Validate + classify incoming signals; reject malformed/over-broad alerts; treat log content as untrusted. |
| **L2 · Tool-call** | Schema-check + call budget + **safe-action allowlist** — no tool outside the allowlist runs. |
| **L3 · Output (post-hook)** | Every proposed fix validated + logged; no secrets in the proposal; confidence scored. |
| **L4 · Governance gate** | **Human approval mandatory and non-skippable before ANY production change**; full audit chain. |

**The live risk — Human-Agent trust exploitation:** a polished-but-wrong remediation must be caught at L4.
The gate defends against a confident, plausible, harmful proposal.

**Programme security constraints (non-negotiable):**
- **No external LLM APIs** — runtime model via the LiteLLM gateway only.
- **No secrets** in code, prompts, or commits; secrets from `.env` only.
- The **🔒 red-team check**: inject a plausible-but-harmful remediation → L4 stops the apply; the audit chain shows why.

## 8. Non-functional requirements (the 10-point scaffold)

Type hints on every boundary · LLM/tool output → Pydantic v2 strict · async I/O only · typed resilient LLM
client (timeout + retry + jitter) · provider behind a port · tool-call args validated before execution ·
**destructive tools: business-rule + auth guards (the crux of CS3)** · FastAPI request **and**
`response_model` · contract tests as a CI gate · **pure core**. See `AGENTS.md` §5.

## 9. Acceptance criteria / Definition of Done

- [ ] `Monitor → Diagnose → Remediate(propose) → Validate` runs end-to-end on a simulated incident
- [ ] **No production change applies without passing the L4 human gate** (proven in a test)
- [ ] Safe-action allowlist enforced on `runbook_executor` / `ticket_rollback`
- [ ] MCP server live (4 tools) with auth + rate limit + audit
- [ ] 🔒 A plausible-but-harmful remediation is stopped at L4; the audit chain shows why
- [ ] Ten-point scaffold passes; CI gate green; committed
- [ ] Every behaviour has a `SPEC-CS3-NNN` ID traced **spec → code → test → PR**; `trace_check.py` green
- [ ] `README.md` written and current (overview, setup, run, tools, guardrails incl. the human-gate rule, build log, spec map)

## 10. Initial spec backlog (seed SDD — write the gate FIRST)

Build spec-first (`AGENTS.md` §13). **Implement `SPEC-CS3-007` before the executable tools exist**, so the
gate is in place by construction. Suggested opening IDs — refine/split as needed:

| Spec ID | Behaviour | Layer / check |
|---|---|---|
| `SPEC-CS3-007` | **No production change applies without passing the L4 human gate** (build first) | **L4** / check 7 |
| `SPEC-CS3-001` | Monitor detects an anomaly into a strict `Incident` | check 2 |
| `SPEC-CS3-002` | Diagnosis produces a typed root-cause hypothesis | check 2 |
| `SPEC-CS3-003` | Remediation only PROPOSES (fix + rationale + confidence); cannot apply | **L3** |
| `SPEC-CS3-004` | Safe-action allowlist enforced on executable tools | **L2** / check 7 |
| `SPEC-CS3-005` | Full audit chain recorded: signal → diagnosis → proposed fix → approver | L4 |

## 11. Deliverables & traceability

- Running agent pipeline + FastAPI MCP server (4 tools)
- `specs/` with one `SPEC-CS3-NNN.md` per behaviour + `specs/SPECS.md` index
- `README.md` kept current, including the §11 spec map and the human-gate rule made unmissable
- Green CI: scaffold self-test + guardrail self-test + `trace_check.py`
- The non-skippable-approval test is the test for `SPEC-CS3-007`
- One ID → spec + code + test + PR: `grep -rn "SPEC-CS3-007" .`
