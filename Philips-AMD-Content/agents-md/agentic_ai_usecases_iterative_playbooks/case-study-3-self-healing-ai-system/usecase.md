# Case Study 3 — Self-Healing AI System

## Original Use Case

An agent system that monitors a running service, detects anomalies, diagnoses root cause, and applies or proposes a validated fix — under a human gate for production changes.

### Agents Built

Monitor Agent → Diagnosis Agent → Remediation Agent → Validation Agent

### MCP Tools

Log/metrics reader, alerting hook, runbook executor, ticket + rollback caller

### Guardrails

Pre-hook: authorised services + safe-action allowlist  
Post-hook: every fix validated + logged before close  
Gate: human approval before any production change

### Why Philips

Self-healing cuts downtime and on-call load. The highest-trust automation — responsible autonomy with mandatory human gates.


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


## Safety Requirement

This case study must use a **simulated environment**.

Do not connect the exercise to a real production environment.

All remediation must initially operate on local simulated state.

## Seed Data

```text
seed_data/
├── services/
│   └── payment-api.yaml
├── metrics/
│   ├── normal.json
│   └── degraded.json
├── logs/
│   ├── normal.log
│   └── incident.log
├── runbooks/
│   ├── high-latency.md
│   ├── connection-pool.md
│   └── dependency-timeout.md
├── allowed_actions.yaml
├── simulated_environment/
│   └── service_state.json
└── incidents/
    └── expected-diagnosis.yaml
```

## Prompt 1 — Create the Simulated Environment

```text
Create a fictional simulated production-like environment for a self-healing
AI demonstration.

Create:
1. one payment-api service
2. normal and degraded metrics
3. application logs
4. three runbooks
5. an allowed-action policy
6. simulated service state
7. expected diagnosis and remediation outcomes

Include incidents for:
- high latency caused by connection-pool exhaustion
- dependency timeout
- elevated error rate caused by a simulated configuration issue

For each incident provide:
- symptoms
- evidence
- likely root cause
- approved remediation
- validation criteria
- rollback condition

All remediation must remain simulated.
```

## Prompt 2 — Create the Technical Specification

```text
Create the implementation specification.

Use Python, LangGraph or CrewAI, MCP server/client and Streamlit.

Specify:
- monitoring flow
- incident state
- agent responsibilities
- MCP tools
- action allowlist
- human approval gate
- validation
- rollback
- audit logging
- failure scenarios
- evaluation dataset
- Streamlit incident-console screens

Do not implement yet.
```

## Prompt 3 — Build Monitor Agent

```text
Implement only the Monitor Agent.

It reads simulated metrics and logs through MCP and produces:

incident_id
service
severity
symptoms
evidence
status

It must not diagnose or remediate.
```

## Prompt 4 — Build MCP Server

```text
Create an MCP server exposing:

1. read_metrics(service)
2. read_logs(service)
3. get_runbook(name)
4. get_allowed_actions(service)
5. create_incident_ticket(incident)
6. execute_simulated_action(action)
7. validate_service(service)
8. rollback_simulated_action(action)

execute_simulated_action must operate only on local simulated state.

Every tool call must be logged.

No tool may execute arbitrary shell commands.
```

## Prompt 5 — Add Diagnosis

```text
Extend:

Monitor → Diagnosis

The Diagnosis Agent must:
- use logs and metrics as evidence
- retrieve relevant runbooks
- produce ranked root-cause hypotheses
- cite supporting evidence
- identify missing evidence
- recommend remediation only when evidence is sufficient

It must not execute remediation.
```

## Prompt 6 — Add Remediation

```text
Extend:

Monitor → Diagnosis → Remediation

The Remediation Agent must:
- check the safe-action allowlist
- prepare a remediation plan
- identify expected outcome
- identify rollback action
- wait for human approval

The LLM must never directly perform a production change.

Only the approved simulated action may be sent through MCP after the
Streamlit human approval gate.
```

## Prompt 7 — Add Validation

```text
Extend:

Monitor → Diagnosis → Remediation → Validation

Validation must:
- compare metrics before and after
- check error rate
- check latency
- verify intended state
- return PASS or FAIL
- recommend rollback when validation fails

Every remediation must have an explicit validation result.
```

## Prompt 8 — Implement Guardrails

```text
Implement:

Pre-hook: authorised services + safe-action allowlist
Post-hook: every fix validated + logged before close
Gate: human approval before any production change

Use deterministic Python checks for safety-critical rules.

The LLM must never bypass:
- service authorization
- action allowlist
- human approval
- validation
- rollback policy
```

## Prompt 9 — Build Streamlit Incident Console

```text
Create a Streamlit incident console displaying:
1. service health
2. anomaly
3. logs
4. metrics
5. diagnosis
6. evidence
7. proposed remediation
8. allowlist result
9. expected result
10. rollback plan

Provide:
Approve Remediation
Reject Remediation
Request More Evidence

After approval:
- execute only the approved simulated action through MCP
- validate it
- show result
- show audit trail

Never automatically approve remediation.
```

## Prompt 10 — Improve Iteratively

```text
Run all seeded incidents.

Find one failure:
- wrong root cause
- insufficient evidence
- unsafe action
- incomplete validation
- missing rollback

Improve the smallest relevant component.

Determine whether the fix belongs in:
1. agent prompt
2. MCP tool contract
3. workflow/state
4. deterministic guardrail
5. evaluation data
6. human gate

Add a regression test and demonstrate it passes.
```

## Participant Challenge

Introduce a new unseen simulated incident.

Demonstrate:
- detection
- diagnosis
- runbook retrieval
- safe remediation proposal
- human approval
- simulated execution
- validation
- rollback when validation fails

## Definition of Done

Demonstrate monitoring, MCP retrieval, anomaly detection, evidence-based diagnosis, safe remediation planning, action allowlisting, mandatory human approval, simulated remediation, validation, rollback, audit logging, regression testing and iterative agent improvement.
