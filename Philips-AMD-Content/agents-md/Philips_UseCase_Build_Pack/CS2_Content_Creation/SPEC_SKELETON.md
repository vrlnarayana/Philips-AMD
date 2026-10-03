# SPEC-<CS>-NNN · <short title>

> One spec = one small, testable behaviour. Copy this file to `specs/SPEC-<CS>-NNN.md`, fill it in,
> get it confirmed, then implement. Carry the ID into the code comment, the test, and the PR (AGENTS.md §13).
> Also add a row to `specs/SPECS.md`.

- **ID:** `SPEC-<CS>-NNN`   (CS1 / CS2 / CS3 · zero-padded · never reused)
- **Title:** <short behaviour name>
- **Status:** draft → approved → implemented → verified
- **Owner:** <name>   ·   **Date:** YYYY-MM-DD

## Intent
<One or two sentences: what behaviour this spec defines and why it matters.>

## Behaviour (acceptance criteria — must be pass/fail testable)
- Given <context>, when <action>, then <expected result>.
- <criterion 2 …>
- <criterion 3 …>

If you cannot write a pass/fail test for every bullet, the spec is too big — split it.

## Scaffold & guardrail link
- **Scaffold check(s):** <e.g. check 2 (strict model), check 6 (tool-call validation), check 8 (response_model)>
- **Guardrail layer(s):** <L1 / L2 / L3 / L4 / none>  — what this spec makes the guardrail do.

## Traceability (the four anchors share this ID)
- **Spec:** `specs/SPEC-<CS>-NNN.md`  (this file)  ·  indexed in `specs/SPECS.md`
- **Code:** `# [SPEC-<CS>-NNN] …`  in  `<file(s)>`
- **Test:** `test_spec_<cs>_<nnn>_<behaviour>()`  in  `<test file>`
- **PR / commit:** title includes `[SPEC-<CS>-NNN]`  ·  PR #: <filled when merged>

## Notes
<edge cases, open questions, links to related specs>
