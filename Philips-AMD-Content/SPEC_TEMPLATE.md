---
# Machine-readable header. Keep this block — the CI traceability check reads it.
spec_id: SPEC-NNN                 # unique, never reused, never renumbered
version: v1                       # bump on every change that alters behaviour
status: draft                     # draft | in review | approved | superseded | retired
supersedes:                       # SPEC-NNN vN, if this replaces something
risk_band: amber                  # green | amber | red  (see the risk band rules)
owner:                            # the person accountable, not the person typing
approver:                         # who signs it off. Required for amber and red
service:                          # which service or repo this belongs to
created:                          # YYYY-MM-DD
last_reviewed:                    # YYYY-MM-DD
related_adrs: []                  # [ADR-012, ADR-014]
ai_involved: true                 # does a model produce or shape any output?
---

# SPEC-NNN · <short title in plain words>

> **How to use this template.** Fill in every section. If a section does not apply, write
> *"Not applicable — <reason>"*. Do not delete the heading, because a missing heading looks the same
> as a forgotten one. Delete these grey instruction blocks as you fill each section in.
>
> **The test of a production-grade spec:** hand it to someone who was not in any of the meetings.
> If they can build it, test it, and tell you what it must never do — the spec is ready. If they have
> to come and ask you, it is not.
>
> This matters more than it used to. When a person writes the code, a weak spec means they come and
> ask questions. When a coding agent writes the code, a weak spec means it assumes something and
> writes a thousand lines based on that assumption, very quickly, across many files.

---

## 1. Summary

> One paragraph, maximum five sentences. Written so that a business sponsor could read it aloud in a
> meeting and be understood. No technology names here.

<!-- Example:
When an agent closes an arrears case, the system checks the affordability working against the
forbearance policy and tells the agent whether the calculation is consistent with policy. It does not
talk to the customer and it does not close anything by itself. Every "inconsistent" result goes to a
senior case handler. -->

---

## 2. Problem and context

### 2.1 What happens today
> Describe the current situation as it actually is, not as the process document says it is. Include
> the workaround people use when things go wrong, because that is usually where the real problem sits.

### 2.2 Evidence
> Numbers, not opinions. Where did each number come from, and when was it measured?

| Evidence | Value | Source | Date |
|---|---|---|---|
| <what you measured> | <value> | <report, query, system> | YYYY-MM-DD |

### 2.3 Why now
> What changed, or what deadline exists. "The sponsor asked" is a reason but not a justification.

### 2.4 What we tried before
> Any earlier attempt and why it did not work. This is the cheapest information available and it is
> almost always forgotten. If nothing was tried before, say so.

---

## 3. Goals and non-goals

### 3.1 Goals
1. <a change in the world, not a feature>
2. …

### 3.2 Non-goals
> The things people will assume are included. Write them down so nobody is disappointed later.
1. <thing this will deliberately not do>
2. …

### 3.3 The number that matters
> One measure. What it is today, what it should become, and who agrees with both.

| Measure | Today | Target | Agreed with | How it is measured |
|---|---|---|---|---|
| | | | | |

---

## 4. Users and stakeholders

| Role | What they need | What they can veto | Notes |
|---|---|---|---|
| <named role, not "users"> | | | |

> Also answer: **who is worst off if this works?** Every change disadvantages somebody. Naming them
> here is how you avoid discovering them at go-live.

---

## 5. Scope

### 5.1 In scope
- <thing this release includes>

### 5.2 Out of scope
- <thing explicitly excluded, and why>

### 5.3 Deferred
| Item | Why deferred | Revisit when |
|---|---|---|
| | | |

---

## 6. Acceptance criteria

> **This is the most important section in the document.** Everything downstream — the code, the tests,
> the evals, the pull request — points back to the IDs you write here.
>
> **A criterion is testable when a stranger can decide pass or fail without asking you.**
>
> | Weak | Strong |
> |---|---|
> | The system should be fast | 95% of requests return within 2 seconds, measured at the API |
> | Answers should be accurate | Every answer cites at least one policy clause ID that exists in the corpus |
> | It should handle errors well | When retrieval returns nothing, the system refuses and states which corpus it searched |
> | It should be secure | No request from a user without the `case.review` permission reaches the scoring step |
>
> Write the criterion first, then ask: *how would I check this?* If you cannot answer, you have not
> yet understood the requirement.

| ID | Criterion | Priority | Verified by | Test / eval reference |
|---|---|---|---|---|
| AC-1 | <plain-language, testable statement> | Must | Test / Eval / Manual | `<test name or eval case>` |
| AC-2 | | Must | | |
| AC-3 | | Should | | |

**Rules for this table**

- IDs are never reused and never renumbered, even when a criterion is removed. Mark it `withdrawn in v3` instead.
- Every `Must` criterion needs at least one test or eval before this spec can be approved.
- `Verified by: Manual` is allowed but must name who does the check and how often.

---

## 7. Behaviour

### 7.1 The normal path
> Step by step, in plain words. What goes in, what happens, what comes out.

### 7.2 Edge cases
| Situation | Expected behaviour | Criterion |
|---|---|---|
| Empty input | | AC-n |
| Very large input | | AC-n |
| Input in an unexpected format or language | | AC-n |
| No result found | | AC-n |
| Two users doing this at the same time | | AC-n |

> The last row is the one most often left blank, and it is the one that produces the strangest
> production bugs — the ones that only appear when volume is high.

### 7.3 Failure behaviour
> What the system does when something it depends on is unavailable. "It retries" is not a failure
> behaviour; it is a delay before the failure.

| Dependency | If it is down | What the user sees | Criterion |
|---|---|---|---|
| | | | AC-n |

### 7.4 What this system must never do
> A short, hard list. These become tests that must fail loudly, not guidelines.
1. <never>
2. …

---

## 8. Data

| Question | Answer |
|---|---|
| What data does this read? | |
| Where does it physically live? | |
| Are we allowed to use it for this purpose? | <state the licence or agreement, not an assumption> |
| Does any of it leave its jurisdiction? | |
| What is deliberately excluded, and why? | |
| What is written, and where? | |
| How long is it kept? | |
| Does any personal or special-category data appear in logs? | |

> **Record exclusions, not only inclusions.** "We are not using the call recordings because clause
> 14.3 of the servicing agreement forbids it" is a sentence that saves someone six months of work.

---

## 9. AI-specific requirements

> Fill this in whenever a model produces or shapes any part of the output. If `ai_involved: false`
> in the header, write "Not applicable" and move on.

| Item | Specification |
|---|---|
| What the model is used for | <extraction · classification · drafting · reasoning> |
| Model and version | |
| Prompt version, and where it is stored | |
| Grounding source | <which corpus, retrieved how> |
| Is the output allowed without a citation? | |
| Refusal policy | <when the system must decline to answer> |
| Determinism expectations | <temperature, and whether repeat runs may differ> |
| Fallback model, and when it engages | |
| Classes that must never fall back | |
| Cost ceiling per request | |

### 9.1 Human decision points
| Where | Who decides | What they are shown | Criterion |
|---|---|---|---|
| | | | AC-n |

> "A human reviews it" is not a decision point. Name the step, the role, and what information that
> person has in front of them at that moment.

### 9.2 Untrusted content
> Which text entering the model was written by somebody outside your control? Retrieved documents,
> user tickets, uploaded files and emails are all untrusted content. State what stops that text from
> being followed as an instruction.

---

## 10. Non-functional requirements

| Requirement | Target | Measured how | Criterion |
|---|---|---|---|
| Latency (p95) | | | AC-n |
| Throughput at peak | | | AC-n |
| Availability | | | |
| Cost per request | | | AC-n |
| Concurrency | | | AC-n |

> Include the peak, not the average. If there is a month-end or a seasonal spike, state it here —
> systems are sized for the worst normal day, not the typical one.

---

## 11. Security notes

| Surface | What could go wrong | Control | Where the control lives |
|---|---|---|---|
| Identity and permission | | | |
| Untrusted content entering the model | | | |
| Output handling (does anything execute it?) | | | |
| Secrets and credentials | | | |
| What this system can write to | | | |
| Cost abuse | | | |

> The full threat model may live in a separate document. Link it here. This table is the minimum that
> belongs in the spec itself, because it shapes the design.

---

## 12. Governance and compliance

| Item | Answer |
|---|---|
| Risk band, and why | |
| Impact assessment | <link, or "not required because …"> |
| Who is affected if the output is wrong | |
| Is a wrong "yes" worse than a wrong "no"? | <these are rarely symmetric> |
| Regulatory obligations that apply | |
| Audit record: what is written, and where | |
| Approvals needed before go-live | |

### 12.1 ISO/IEC 42001 control objectives touched
> Map by objective theme. Verify exact sub-control references against the standard text before
> quoting them anywhere outside the team.

| Objective | Theme | How this spec addresses it |
|---|---|---|
| A.5 | Impact assessment | |
| A.6 | AI system life cycle, human oversight | |
| A.7 | Data | |
| A.8 | Information for interested parties | |
| A.9 | Use of AI systems | |
| A.10 | Third-party relationships | |

---

## 13. Evaluation plan

> **Every spec ships with its eval.** Written at the same time as the acceptance criteria, by the same
> person, before the code exists.

| Item | Specification |
|---|---|
| Eval set location | `<path in repo>` |
| Number of cases | |
| How cases were chosen | <real examples, edge cases, known past failures> |
| Who reviewed the expected answers | |
| Baseline score | |
| Gate threshold | <what score blocks a merge> |
| Compared against | <main branch, not just pass/fail> |
| Review cadence | <when the eval set is rebuilt> |

### 13.1 Criterion to eval mapping
| Criterion | Eval cases | What "pass" means for this criterion |
|---|---|---|
| AC-1 | | |

> A criterion with no test and no eval is either not done, or the spec is out of date. Both are
> problems; neither is acceptable at approval.

---

## 14. Observability

| Signal | What we watch | Alert when | Who is paged |
|---|---|---|---|
| Quality | | | |
| Refusal / fallback rate | | | |
| Retrieval miss rate | | | |
| Latency | | | |
| Cost per resolved request | | | |

### 14.1 Drift signals
| Drift type | How we detect it | First response |
|---|---|---|
| Input drift — what arrives has changed | | |
| Output drift — responses change shape | | |
| Eval-score drift — measured quality falls | | |

> If eval-score drift is the first thing you notice, detection failed two stages earlier.

---

## 15. Rollout and rollback

| Item | Plan |
|---|---|
| Rollout approach | <dark launch · percentage · pilot group · big bang> |
| Who sees it first, and for how long | |
| Success signal to continue | |
| Rollback trigger | <specific and observable> |
| Rollback mechanism | <how, and how long it takes> |
| Has rollback been tested? | |
| Communication if we roll back | |

---

## 16. Dependencies and assumptions

### 16.1 Dependencies
| Dependency | Owner | What happens if it is not ready | Confirmed? |
|---|---|---|---|
| | | | |

### 16.2 Assumptions
| # | We are assuming | How we would find out if it is wrong | Impact if wrong |
|---|---|---|---|
| A-1 | | | |

> Keep this table honest. A spec with no assumptions listed has either not been thought about
> properly, or is not telling you the truth.

---

## 17. Open questions

| # | Question | Who can answer it | Needed by | Status |
|---|---|---|---|---|
| Q-1 | | | | Open |

> Open questions do not block approval by themselves. An open question with no owner and no date
> does.

---

## 18. Decisions

| ADR | Decision | Why it mattered |
|---|---|---|
| ADR-nnn | | |

> Record decisions where a competent engineer could reasonably have chosen otherwise. If there was
> only one sensible option, it is not a decision worth recording.

---

## 19. Change log

| Version | Date | Author | What changed | Why |
|---|---|---|---|---|
| v1 | YYYY-MM-DD | | First version | |

> Bump the version whenever behaviour changes. Keep every old version in git history. If the code was
> changed and this table was not updated, that is spec drift, and it starts exactly here.

---

## 20. Spec readiness checklist

> The gate. A spec is production-grade only when every line below is true. Tick honestly — an untrue
> tick is worse than an empty one, because it stops anyone from looking.

**Clarity**
- [ ] Someone outside the team could build this without asking a question
- [ ] The summary names no technology
- [ ] Every section is filled in, or explicitly marked not applicable with a reason

**Acceptance criteria**
- [ ] Every criterion has an ID
- [ ] Every criterion is testable by a stranger, with no discussion needed
- [ ] Every `Must` criterion has a test or an eval named against it
- [ ] Criteria cover the failure cases, not only the normal path

**Behaviour**
- [ ] Edge cases listed, including two users at the same time
- [ ] Failure behaviour defined for every external dependency
- [ ] The "must never do" list exists and each item is testable

**Data and AI**
- [ ] We have stated that we are permitted to use each data source for this purpose
- [ ] Exclusions are recorded with reasons
- [ ] Model, prompt version and grounding source are named
- [ ] Human decision points name the step, the role and what that person sees
- [ ] Untrusted content entry points are identified

**Non-functional**
- [ ] Latency, cost and concurrency targets are numbers, not adjectives
- [ ] Peak load is stated, not just average

**Governance**
- [ ] Risk band set, with a reason
- [ ] Impact assessment done, or the exemption justified
- [ ] Audit record defined
- [ ] Approver named, and they have actually seen this

**Evaluation**
- [ ] Eval set exists and its location is written down
- [ ] Gate threshold agreed
- [ ] Every criterion maps to at least one eval case or test

**Operations**
- [ ] Alerts defined with a named owner
- [ ] Drift detection defined for all three stages
- [ ] Rollback trigger is observable and the mechanism has been tested

**Traceability**
- [ ] The spec lives in the repository, not in a chat message or a document tool
- [ ] Version number is in the header and in the code comment
- [ ] Every criterion ID will appear in a test name and in the pull request

---

## Appendix A · Traceability convention

One ID, carried the whole way. No tooling required — a text search is enough.

```
Spec        docs/specs/SPEC-014.md          (v2)
Criterion   AC-3: refuse when no policy clause is found
Code        src/retrieval.py                # SPEC-014 v2 AC-3
Test        test_refuses_when_no_clause     # AC-3
Eval        evals/spec-014/case-12.yaml     # AC-3
PR          PR-238  "SPEC-014 v2 — AC-1, AC-3"
```

Anyone can now search the repository for `AC-3` and see why the code exists, what it promised, and
how that promise is checked.

## Appendix B · A minimal CI drift check

Put something like this in the pipeline. It does not need to be clever to be useful.

```bash
#!/usr/bin/env bash
# Warn when an acceptance criterion has no test referencing it.
set -uo pipefail
SPEC="docs/specs/${1:?usage: check-traceability.sh SPEC-014}.md"
missing=0

# Pull AC IDs out of the spec's criteria table
grep -oE '^\| (AC-[0-9]+)' "$SPEC" | awk '{print $2}' | sort -u | while read -r ac; do
  if ! grep -rq "$ac" tests/ evals/ 2>/dev/null; then
    echo "WARNING: $ac has no test or eval referencing it"
    missing=1
  fi
done

exit 0   # warn only at first. Make it fail the build once the team is used to it.
```

Start it as a warning. Turn it into a blocking check once the backlog of untraced criteria is clear —
switching it on cold on an existing codebase only teaches people to ignore the warning.

## Appendix C · Worked criterion

A single criterion, written properly, showing what each column is for.

| ID | Criterion | Priority | Verified by | Test / eval reference |
|---|---|---|---|---|
| AC-3 | When retrieval returns no policy clause above the relevance threshold, the system must return "cannot determine", must name the corpus it searched, and must not produce an affordability opinion | Must | Test + Eval | `test_refuses_when_no_clause`, `evals/spec-014/case-12` |

Why this one works:

- A stranger can decide pass or fail. There is no judgement needed.
- It states what must happen **and** what must not happen. The second half is the part that matters.
- It is checkable two ways: a unit test for the refusal path, and an eval for the wording of the message.
- It came from a real failure, not from imagination.

---

*Template version 1.0 · Techademy · FORGE FDE Academy · companion to AI-Assisted Engineering Part 2
and to the Production-isation Playbook workbook.*
