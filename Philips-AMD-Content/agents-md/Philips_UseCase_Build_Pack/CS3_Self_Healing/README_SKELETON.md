# <Project / Use-Case Name>

> Copy this file to `README.md` at your project root and fill it in. **Keep it updated** — if you
> change the code, update the matching section in the same step (see `AGENTS.md` §12). A stale README
> is a defect.

---

## 1. What this is

<One paragraph: what this system does, which use case (CS1 / CS2 / CS3), and what it produces.>

## 2. Architecture

- **Flow:** `<Agent1> → <Agent2> → <Agent3> → <Agent4>` (typed objects between each step)
- **Orchestrator:** <how it routes / composes the agents>
- **MCP server:** a FastAPI app exposing the tools the agents call (see §6)
- (optional) a diagram:

```
<source> --> Ingest --> Structure --> Draft --> Review Routing --> [human gate] --> output
```

## 3. Prerequisites

- Python 3.11+
- `uv` (environment + lockfile)
- Access to the LiteLLM gateway (runtime model endpoint) — no external LLM APIs

## 4. Setup

```bash
git clone <repo> && cd <project>
uv sync --frozen                # install the exact, hash-locked dependency tree
cp .env.example .env            # then fill in the values (see §8)
```

## 5. How to run

```bash
# start the MCP server (dev)
uv run uvicorn app.api.server:app --reload        # http://127.0.0.1:8000  (docs at /docs)

# run the agent pipeline / client against a sample input
uv run python run_client.py
```

## 6. MCP tools

| Tool | What it does | Read-only / Destructive | Gated by |
|---|---|---|---|
| `<tool_1>` | <…> | read-only | — |
| `<tool_2>` | <…> | read-only | — |
| `<tool_3>` | <…> | **destructive** | auth + human confirmation (L4) |
| `<tool_4>` | <…> | **destructive** | auth + allowlist + human confirmation (L4) |

Every tool: typed strict request/response · auth on every call · rate limit per caller · audit log.

## 7. Guardrails (the four layers)

| Layer | What it checks here |
|---|---|
| **L1 · Input (pre-hook)** | <validate + classify input; treat tool/doc content as untrusted> |
| **L2 · Tool-call** | schema check + call budget + tool allowlist |
| **L3 · Output (post-hook)** | <PII / secret / brand / factual scan> |
| **L4 · Governance gate** | <human approval before any consequential action> + full audit log |

## 8. Environment variables

| Variable | What it's for | Where to get it |
|---|---|---|
| `OPENAI_BASE_URL` | LiteLLM gateway endpoint | from the trainer / `http://<gateway>:4000/v1` |
| `OPENAI_API_KEY` | your per-seat gateway key | from your seat sheet — **never commit** |
| `DEFAULT_MODEL` | runtime model name | e.g. `<model>` |
| `<other>` | <…> | <…> |

> **Never commit real secrets.** `.env` is git-ignored; only `.env.example` (placeholders) is committed.

## 9. Testing

```bash
uv run pytest                                   # contract + property tests (LLM mocked)
uv run python -m app.core.guardrails --selftest # guardrail self-test
```

## 10. Build log / status

Keep a short, dated list of what has been built. Append to it at the end of each session.

- `YYYY-MM-DD` — <what you built / changed>
- `YYYY-MM-DD` — <…>

## 11. Spec map / traceability

This project is built **spec-first** (see `AGENTS.md` §13). Every behaviour has a `SPEC-<CS>-NNN`
ID carried `spec → code → test → PR`. The specs live in `specs/`, indexed in `specs/SPECS.md`.

- **Specs:** `specs/SPEC-<CS>-NNN.md` (start from `SPEC_SKELETON.md`) · index: `specs/SPECS.md`
- **In code:** each implementation carries `# [SPEC-<CS>-NNN] …` on the relevant line(s)
- **In tests:** `def test_spec_<cs>_<nnn>_<behaviour>(): …`
- **In the PR/commit:** the title includes `[SPEC-<CS>-NNN]`
- **Trace check:** `uv run python scripts/trace_check.py` — every spec must appear in ≥1 code file
  **and** ≥1 test, else the build fails.

```bash
grep -rn "SPEC-<CS>-007" .    # one ID → its spec, its code, its test, its PR
```

| Spec ID | Title | Status | Code ref | Test ref | PR |
|---|---|---|---|---|---|
| `SPEC-<CS>-001` | <behaviour> | verified | `app/core/…` | `test_spec_<cs>_001_…` | #<n> |

> Keep this table (or a pointer to `specs/SPECS.md`) current — it is the at-a-glance audit map.
