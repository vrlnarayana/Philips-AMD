# Case Study 1 — Documentation Automation

## Original Use Case

An agent system that reads source artefacts (code, specs, meeting notes) via MCP tools and produces structured documentation — technical specs, SOP documents, process maps.

### Agents Built

Ingestion Agent → Structure Agent → Drafting Agent → Review Routing Agent

### MCP Tools

File reader, template library, version control access, approval workflow caller

### Guardrails

Pre-hook: source data classification check  
Post-hook: output reviewed before publish  
Gate: human sign-off before any doc goes live

### Why Philips

Documentation workflows are a known productivity bottleneck. Immediate applicability. Safe domain for first guardrail implementations.


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

Create this fictional engineering repository:

```text
seed_data/
├── source_code/
│   ├── payment_service.py
│   ├── notification_service.py
│   └── user_service.py
├── specifications/
│   ├── payment_requirements.md
│   └── notification_requirements.md
├── meeting_notes/
│   └── payment-service-review.md
├── existing_docs/
│   └── documentation-style-example.md
├── templates/
│   ├── technical-spec-template.md
│   ├── sop-template.md
│   └── process-map-template.md
└── metadata/
    └── source-classification.yaml
```

## Prompt 1 — Create Seed Data

```text
Create a fictional but internally consistent software engineering repository
for an agentic Documentation Automation system.

Generate:
1. three small related Python services
2. engineering requirements/specifications
3. realistic meeting notes with decisions, ambiguities and action items
4. one existing documentation example
5. technical-spec, SOP and process-map templates
6. source classification metadata

Intentionally include:
- one ambiguous requirement
- one outdated statement
- one missing requirement
- one contradiction between meeting notes and specification

Clearly label everything as fictional training data.

Return each file with its filename and complete contents.
```

## Prompt 2 — Create the Technical Specification

```text
Using the seed repository, create an implementation specification for this
Documentation Automation application.

Cover:
- business problem
- user journey
- actors
- agents
- LangGraph/CrewAI workflow
- state schema
- MCP client/server interaction
- MCP tool contracts
- seed-data contract
- Streamlit screens
- guardrails
- human approval
- failure cases
- evaluation cases
- project structure

Do not implement the application yet.
```

## Prompt 3 — Build the Minimal Workflow

```text
Implement only:

Ingestion Agent → Structure Agent → Drafting Agent

The Ingestion Agent retrieves and classifies source material through MCP.

The Structure Agent creates an outline and identifies conflicts.

The Drafting Agent creates a draft but cannot publish.

Use structured state and typed outputs.

Do not implement Review Routing yet.
Do not allow the LLM direct filesystem access.
```

## Prompt 4 — Build the MCP Server

```text
Implement an MCP server exposing:

1. list_source_files()
2. read_source_file(path)
3. get_template(template_name)
4. get_source_metadata(path)

Requirements:
- validate all paths
- prevent path traversal
- only expose seed_data/
- return structured errors
- log every tool call
- reject unsupported file types
- include unit tests
```

## Prompt 5 — Build the Streamlit MCP Client

```text
Create a Streamlit application that acts as the MCP client.

Allow the user to:
1. select source files
2. inspect metadata
3. select a template
4. start the workflow
5. see agent progress
6. inspect the outline
7. inspect the draft
8. inspect MCP tool calls
9. inspect guardrail results
10. approve or reject publication

Never publish automatically.
```

## Prompt 6 — Add Review Routing

```text
Extend:

Ingestion → Structure → Drafting → Review Routing

Review Routing must identify:
- missing information
- contradictions
- unsupported claims
- required human review items

Return:
READY_FOR_REVIEW or NEEDS_REVISION

The agent must never approve its own document for publication.
```

## Prompt 7 — Add Guardrails

```text
Implement:

Pre-hook: source data classification check
Post-hook: output reviewed before publish
Gate: human sign-off before any document goes live

Prefer deterministic Python checks for safety-critical rules.

Return structured guardrail results with:
passed, findings, severity and requires_human_review.

A failed guardrail must stop or route the workflow appropriately.
```

## Prompt 8 — Improve the Agent Iteratively

```text
Run the system against all seed documents.

Find one important failure:
- hallucinated fact
- ignored contradiction
- missing requirement
- poor structure
- incorrect template usage

Do not rewrite the whole application.

Fix the smallest relevant component:
agent prompt, state, MCP tool or deterministic guardrail.

Add a regression test reproducing the failure.
Run the test and explain the improvement.
```

## Participant Challenge

Add a new SOP generation capability.

The participant must update:
- template
- MCP tool access
- Structure Agent
- Drafting Agent
- Streamlit UI
- evaluation tests

Existing technical-spec generation must continue to work.

## Definition of Done

Demonstrate source ingestion, MCP retrieval, multi-agent workflow, structured state, documentation generation, classification guardrail, review guardrail, human approval, audit trail, one intentional failure and one agent improvement.
