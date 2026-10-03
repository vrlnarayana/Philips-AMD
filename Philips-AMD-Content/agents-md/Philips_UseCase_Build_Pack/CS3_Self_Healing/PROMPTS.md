# PROMPTS.md — CS3 · Self-Healing AI System (OpenCode kickoff prompts)

> **Setup.** Create an empty project folder, copy this folder's `AGENTS.md`, `README_SKELETON.md`,
> `SPEC_SKELETON.md`, `SPECS_INDEX_SKELETON.md` into it, open OpenCode (Kimi-K2.5) there, then paste the
> prompts **in order**. Read `USECASE.md` first for the requirements.
>
> ⚠ **Highest-autonomy case — build the L4 human gate (SPEC-CS3-007) FIRST, before the executable tools.**
>
> **Rule (AGENTS.md §13):** *no code without a spec ID.* Flow is always **spec → confirm → build+test+tag
> → attack → commit**, carrying `SPEC-CS3-NNN` through code comment, test name, and PR title.
> Fill the `<gateway>`, `<model>`, `<your name>` placeholders in `AGENTS.md` first. Secrets in `.env` only.

---

## Prompt 0 — Bootstrap (skeleton only, no behaviour yet)

```
Read AGENTS.md end to end and treat it as the contract. Confirm back, in a few lines, that you've
understood: the 10-point scaffold (§5), the 4 guardrail layers (§6) with L4 as the load-bearing gate, the
MCP tool list (§7), the hexagonal layout (§8), the README rule (§12), and spec-first SDD with
SPEC-CS3-NNN traceability (§13).

Then scaffold the PROJECT SKELETON ONLY — no agent logic yet:
- the folder tree from §8 (app/core, app/adapters, app/api, app/tests) with empty stubs
- pyproject.toml via uv; run `uv sync --frozen` once deps are listed
- .env.example (OPENAI_BASE_URL, key, model) — placeholders only
- specs/ with an empty SPECS.md from SPECS_INDEX_SKELETON.md
- scripts/trace_check.py per §13 (a spec ID must appear in >=1 code file AND >=1 test, else non-zero exit)
- .github/workflows/security.yml running: uv sync, guardrail self-test, pytest, trace_check
- README.md from README_SKELETON.md, §1–§2 filled for CS3; rest TODO

Print the tree. Stop after the skeleton — implement nothing.
```

## The build loop (reuse for every behaviour)

**A — spec first (stop after the spec):**
```
Following §13, write the next spec SPEC-CS3-NNN for: <one small, testable behaviour>.
Create specs/SPEC-CS3-NNN.md from SPEC_SKELETON.md (intent, pass/fail criteria, scaffold check + guardrail
layer). Add a draft row to specs/SPECS.md. STOP — I'll approve before you write code.
```
**B — build + test + tag (after approval):**
```
SPEC-CS3-NNN is approved. Implement it against the 10-point scaffold (§5): strict Pydantic models in the
pure core, the mapped guardrail layer at the right seam, a test test_spec_cs3_<nnn>_<behaviour>, and a
# [SPEC-CS3-NNN] comment on the implementing line(s). Update specs/SPECS.md (implemented) and the matching
README section in the SAME change. Then run pytest, the guardrail self-test, and scripts/trace_check.py.
```
**C — attack + commit:**
```
Try to break SPEC-CS3-NNN: <adversarial input>. Show the guardrail catching it and the audit entry. If it
holds and trace_check is green, commit: <type>(<scope>): <summary> [SPEC-CS3-NNN]. Next spec.
```

## Ordered build (each through A→B→C)

1. **SPEC-CS3-007 FIRST — the non-skippable human gate (the §13 worked example):**
   ```
   Build loop, Step A: SPEC-CS3-007 — "No production change applies without passing the L4 human gate.
   The Remediation agent only PROPOSES; apply is blocked until a human approves, and the full chain
   signal → diagnosis → proposed fix → approver is audited." Stop after the spec.
   ```
   > Implement this before any executable tool exists, so the gate is in place by construction. The
   > non-skippable-approval test is the test for SPEC-CS3-007 — name it with the ID.
2. **Monitor Agent** — watch logs/metrics via `log_metrics_reader`; detect anomalies into a typed `Incident`.
3. **Diagnosis Agent** — reason about root cause from the signals.
4. **Remediation Agent** — **propose** a fix with rationale + confidence; cannot apply alone (ties to SPEC-CS3-007).
5. **Validation Agent** — confirm the fix worked, only **after** human approval.
6. **MCP server** — expose the 4 tools over FastAPI; `runbook_executor` / `ticket_rollback` destructive →
   safe-action allowlist + auth + human confirmation, and cannot fire without passing L4 (a spec per tool).
7. **🔒 Attack & close out:**
   ```
   Inject a confident-but-harmful proposed remediation. Prove L4 blocks the apply and the audit chain
   (signal → diagnosis → proposed fix → approver) explains why. Then walk the Definition of Done in
   USECASE.md §9 and AGENTS.md §10; confirm trace_check is green and the README spec map + human-gate rule
   are current.
   ```

**One-ID check:** `grep -rn "SPEC-CS3-007" .` → spec + code + test + PR.
