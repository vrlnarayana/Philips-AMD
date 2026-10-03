# Philips AMD · Advanced Agentic AI Bootcamp — Use-Case Build Pack

A 2-day hands-on programme (OpenCode + Kimi-K2.5). Each of the three case studies is a **self-contained
folder** a developer can pick up and build from. Every project is built **spec-first** with end-to-end
traceability (`spec → code → test → PR`) and the 4-layer guardrails embedded by construction.

## What each use-case folder contains

| File | What it is |
|---|---|
| `USECASE.md` | **Requirement document** — the *what* and *why*: business context, functional + guardrail requirements, acceptance criteria, and the initial spec backlog. **Read this first.** |
| `AGENTS.md` | **Build contract** for the coding agent (OpenCode reads it automatically) — the *how*: 10-point scaffold, 4 guardrail layers, MCP build, README rule, and SDD/traceability (§13). Place at the project root. |
| `PROMPTS.md` | **OpenCode kickoff prompts** — bootstrap, the spec-first build loop, and the ordered per-agent prompts, ending with the 🔒 red-team check. |
| `README_SKELETON.md` | Template for the project `README.md` (kept in sync with the code; includes the §11 spec map). |
| `SPEC_SKELETON.md` | Template for one spec (`specs/SPEC-<CS>-NNN.md`). |
| `SPECS_INDEX_SKELETON.md` | Template for the traceability index (`specs/SPECS.md`). |

## The three use cases

| Folder | Pipeline | 🔒 Red-team check |
|---|---|---|
| `CS1_Documentation_Automation/` | Ingest → Structure → Draft → Review Routing | injected "ignore previous instructions" in a source must not hijack the agent |
| `CS2_Content_Creation/` | Brief → Draft → Style/Brand → Fact-Check | off-brand request blocked; unverified claim flagged |
| `CS3_Self_Healing/` | Monitor → Diagnose → Remediate (propose) → Validate | plausible-but-harmful remediation stopped at the L4 human gate |

## `_shared/`

- `AGENTS_SKELETON.md` — the blank 13-section AGENTS.md template (basis for all three)
- `README_SKELETON.md`, `SPEC_SKELETON.md`, `SPECS_INDEX_SKELETON.md` — master copies
- `KICKOFF_PROMPTS.md` — the combined prompts for all three cases in one file
- `AGENTS_MD_Guide.pptx` — the participant explainer deck (what AGENTS.md is and how to write it)

## How to run a use case (developer flow)

1. Read `USECASE.md` in the use-case folder.
2. Create an empty project folder; copy that folder's `AGENTS.md` + the three skeletons into it.
3. Fill the `<gateway>`, `<model>`, `<your name>` placeholders in `AGENTS.md`; put secrets in `.env` only.
4. Open OpenCode (Kimi-K2.5) in the project folder and paste the prompts from `PROMPTS.md` in order.
5. Build spec-first (`AGENTS.md` §13): every behaviour gets a `SPEC-<CS>-NNN` ID, carried to the PR.
6. Close out against the Definition of Done (`USECASE.md` §9 / `AGENTS.md` §10); keep `trace_check.py` green.

**Programme constraints (all three):** no external LLM APIs (self-hosted LiteLLM gateway only);
no secrets in code, prompts, or commits; 70% hands-on; guardrails and traceability are part of "done."
