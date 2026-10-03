# PROMPTS.md — CS2 · AI-Powered Content Creation (OpenCode kickoff prompts)

> **Setup.** Create an empty project folder, copy this folder's `AGENTS.md`, `README_SKELETON.md`,
> `SPEC_SKELETON.md`, `SPECS_INDEX_SKELETON.md` into it, open OpenCode (Kimi-K2.5) there, then paste the
> prompts **in order**. Read `USECASE.md` first for the requirements.
>
> **Rule (AGENTS.md §13):** *no code without a spec ID.* Flow is always **spec → confirm → build+test+tag
> → attack → commit**, carrying `SPEC-CS2-NNN` through code comment, test name, and PR title.
> Fill the `<gateway>`, `<model>`, `<your name>` placeholders in `AGENTS.md` first. Secrets in `.env` only.

---

## Prompt 0 — Bootstrap (skeleton only, no behaviour yet)

```
Read AGENTS.md end to end and treat it as the contract. Confirm back, in a few lines, that you've
understood: the 10-point scaffold (§5), the 4 guardrail layers (§6), the MCP tool list (§7), the
hexagonal layout (§8), the README rule (§12), and spec-first SDD with SPEC-CS2-NNN traceability (§13).

Then scaffold the PROJECT SKELETON ONLY — no agent logic yet:
- the folder tree from §8 (app/core, app/adapters, app/api, app/tests) with empty stubs
- pyproject.toml via uv; run `uv sync --frozen` once deps are listed
- .env.example (OPENAI_BASE_URL, key, model) — placeholders only
- specs/ with an empty SPECS.md from SPECS_INDEX_SKELETON.md
- scripts/trace_check.py per §13 (a spec ID must appear in >=1 code file AND >=1 test, else non-zero exit)
- .github/workflows/security.yml running: uv sync, guardrail self-test, pytest, trace_check
- README.md from README_SKELETON.md, §1–§2 filled for CS2; rest TODO

Print the tree. Stop after the skeleton — implement nothing.
```

## The build loop (reuse for every behaviour)

**A — spec first (stop after the spec):**
```
Following §13, write the next spec SPEC-CS2-NNN for: <one small, testable behaviour>.
Create specs/SPEC-CS2-NNN.md from SPEC_SKELETON.md (intent, pass/fail criteria, scaffold check + guardrail
layer). Add a draft row to specs/SPECS.md. STOP — I'll approve before you write code.
```
**B — build + test + tag (after approval):**
```
SPEC-CS2-NNN is approved. Implement it against the 10-point scaffold (§5): strict Pydantic models in the
pure core, the mapped guardrail layer at the right seam, a test test_spec_cs2_<nnn>_<behaviour>, and a
# [SPEC-CS2-NNN] comment on the implementing line(s). Update specs/SPECS.md (implemented) and the matching
README section in the SAME change. Then run pytest, the guardrail self-test, and scripts/trace_check.py.
```
**C — attack + commit:**
```
Try to break SPEC-CS2-NNN: <adversarial input>. Show the guardrail catching it and the audit entry. If it
holds and trace_check is green, commit: <type>(<scope>): <summary> [SPEC-CS2-NNN]. Next spec.
```

## Ordered build (each through A→B→C)

1. **SPEC-CS2-005 first — the brand-safety gate (the §13 worked example):**
   ```
   Build loop, Step A: SPEC-CS2-005 — "The L3 brand-safety gate must block off-brand output before
   publish, and unverified factual claims must be flagged, never shipped silently." Stop after the spec.
   ```
2. **Brief Agent** — interpret the request into a structured `ContentBrief`; validate before drafting.
3. **Drafting Agent** — generate content against the brief as a typed `ContentPiece`.
4. **Style/Brand Agent** — enforce tone/voice via `style_guide_library` (carries SPEC-CS2-005).
5. **Fact-Check & Review Agent** — verify claims; flag anything unverified.
6. **MCP server** — expose the 4 tools over FastAPI; `publish_workflow` destructive → auth + human
   confirmation; every tool typed + auth + rate limit + audit (a spec per tool).
7. **🔒 Attack & close out:**
   ```
   Request content that violates the brand guidelines and slip in an unverifiable claim. Prove the brand
   gate holds and the claim is flagged; show the audit log. Then walk the Definition of Done in USECASE.md
   §9 and AGENTS.md §10; confirm trace_check is green and the README spec map is current.
   ```

**One-ID check:** `grep -rn "SPEC-CS2-005" .` → spec + code + test + PR.
