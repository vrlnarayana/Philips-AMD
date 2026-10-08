# Case Study 2 — AI-Powered Content Creation

## Original Use Case

An agent system that turns source inputs into engineering content — release notes, technical docs, training material — then refines it, with brand-style and factual-accuracy checks before anything is published.

### Agents Built

Brief Agent → Drafting Agent → Style/Brand Agent → Fact-Check & Review Agent

### MCP Tools

Knowledge-base reader, style-guide library, asset fetcher, publish workflow caller

### Guardrails

Pre-hook: source and prompt classification check  
Post-hook: factual + brand-compliance scan before publish  
Gate: human sign-off before any content goes live

### Why Philips

Content production is high-volume and repetitive. Immediate productivity gain, with guardrails preventing off-brand or inaccurate output.


## Participant Development Approach

This is an iterative agentic-AI build exercise. Do **not** attempt to implement the complete system in one step.

### Required Technology

- Python
- One agentic framework: **LangGraph** (recommended) or **CrewAI**
- MCP server exposing domain tools
- MCP client
- Streamlit as the user-facing application and MCP client
- Local seed data/documents for development
- Pydantic for structured data where appropriate
- SQLite or local JSON/Markdown/YAML for simple persistence
- `.env` for configuration and secrets

### Target Architecture

```text
Streamlit App
   │
   │ MCP Client
   ▼
MCP Server ──► Domain Tools / Resources
   │
   ▼
LangGraph / CrewAI
   │
   ├── Agents
   ├── Shared State
   └── Guardrails
   │
   ▼
Seed Data ──► Outputs ──► Evaluation / Audit
```

### Required Iteration Pattern

1. Specification
2. Seed data
3. Minimal workflow
4. First agent
5. MCP tool
6. Additional agents
7. Guardrails
8. Human approval
9. Evaluation
10. Failure injection
11. Agent/tool improvement
12. Regression test

Every iteration should leave the application runnable.

### General Coding Prompt

```text
Before changing code, inspect the current repository.

Implement only the requested iteration. Do not rewrite working functionality.

Use Python and the existing LangGraph or CrewAI architecture.
Use MCP for domain access instead of allowing agents to directly access files,
databases or operating-system capabilities.

Keep tools narrow, typed, validated and auditable.

Add or update tests for every meaningful change.

At the end:
1. list files changed
2. explain the workflow
3. explain how to run it
4. show tests added
5. identify known limitations
6. do not invent enterprise APIs or credentials
```

### Suggested Repository

```text
app/
├── streamlit_app.py
├── agents/
│   ├── graph.py
│   ├── state.py
│   └── prompts.py
├── mcp_server/
│   ├── server.py
│   └── tools/
├── guardrails/
│   ├── pre_hooks.py
│   └── post_hooks.py
├── seed_data/
├── outputs/
├── evaluations/
├── tests/
├── requirements.txt
└── README.md
```


## Seed Data

```text
seed_data/
├── knowledge_base/
│   ├── product-overview.md
│   ├── feature-release.md
│   └── engineering-facts.yaml
├── style_guide/
│   ├── brand-style.md
│   ├── terminology.yaml
│   └── prohibited-claims.md
├── assets/
│   ├── product-diagram.md
│   └── training-outline.md
├── content_requests/
│   ├── release-note-request.md
│   ├── technical-doc-request.md
│   └── training-content-request.md
└── expected/
    └── evaluation-criteria.yaml
```

## Prompt 1 — Create Seed Data

```text
Create a fictional engineering-product knowledge base.

Generate:
1. product overview
2. release description
3. verified engineering facts
4. brand/style guide
5. approved terminology
6. prohibited or unsupported claims
7. product assets as text/Markdown
8. release-note, technical-doc and training-content requests

Intentionally include:
- a style-guide violation
- an unsupported marketing claim
- two competing terms where only one is approved
- one fact that must be verified from the knowledge base

Clearly mark all data as fictional.
```

## Prompt 2 — Create the Technical Specification

```text
Create the implementation specification for an agentic content-creation app.

Use Python, LangGraph or CrewAI, MCP server/client and Streamlit.

Specify:
- user journey
- content types
- agents
- state schema
- MCP tools and contracts
- evidence model
- guardrails
- human approval
- Streamlit screens
- evaluation dataset
- failure scenarios
- phased implementation plan

Do not write the full application.
```

## Prompt 3 — Build the First Workflow

```text
Implement:

Brief Agent → Drafting Agent

The Brief Agent converts a request into a structured content brief.

The Drafting Agent generates content only from the brief and retrieved
source information.

Do not implement publishing.
Do not permit unsupported product facts.
```

## Prompt 4 — Build the MCP Server

```text
Implement an MCP server exposing:

1. search_knowledge_base(query)
2. read_knowledge_document(document_id)
3. get_style_guide()
4. get_approved_terms()
5. get_prohibited_claims()
6. fetch_asset(asset_id)

All responses must be structured.

The server may access only seed_data/.
Add validation, logging and tests.
```

## Prompt 5 — Add Style/Brand Agent

```text
Extend:

Brief → Drafting → Style/Brand

The Style/Brand Agent must:
- check terminology
- check tone
- identify prohibited phrases
- identify unsupported marketing language
- suggest corrections
- return revised content plus a change report

It must not invent facts.
```

## Prompt 6 — Add Fact-Check & Review

```text
Extend:

Brief → Drafting → Style/Brand → Fact-Check & Review

For every product claim:
- locate supporting source evidence
- classify VERIFIED, UNSUPPORTED or CONTRADICTED
- produce a review report
- block publication for critical unsupported claims

Never turn an unsupported claim into a verified fact.
```

## Prompt 7 — Add Guardrails and Human Gate

```text
Implement:

Pre-hook: source and prompt classification check
Post-hook: factual + brand-compliance scan before publish
Gate: human sign-off before any content goes live

The Streamlit UI must display:
- request
- retrieved evidence
- draft
- style findings
- factual findings
- final content
- approve/reject controls

Publishing must be impossible without explicit human approval.
```

## Prompt 8 — Improve Iteratively

```text
Run all three content requests.

Find the most important failure.

Possible failures:
- hallucinated capability
- incorrect terminology
- unsupported claim
- style violation
- missing evidence

Fix only the smallest relevant component.

Add a regression test and compare before/after behavior.

Explain whether the fix belongs in:
- agent prompt
- MCP tool
- workflow/state
- deterministic guardrail
- human gate
```

## Participant Challenge

Add a customer-facing FAQ content type without breaking existing content types.

## Definition of Done

Demonstrate request intake, evidence retrieval, multi-agent drafting, style checking, factual verification, deterministic guardrails, human approval, audit trail, failed evaluation and iterative improvement.
