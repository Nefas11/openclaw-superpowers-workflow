# OpenClaw Superpowers Workflow

A structured pipeline for AI-driven software development: from brainstorm to
verified deployment, with accountability at every step.

![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue)
![OpenClaw](https://img.shields.io/badge/openclaw-compatible-purple)
![License: MIT](https://img.shields.io/badge/license-MIT-green)

## What This Is

A workflow system that turns OpenClaw sub-agents into a governed development
pipeline. Every task passes through defined phases, and every agent output is
independently verified before it counts.

```
 BRAINSTORM ──→ PLAN ──→ IMPLEMENT ──→ REVIEW ──→ VERIFY
     │            │           │            │          │
  Ideation    Contracts    Coding       Critique   Gates
  Scoping     Criteria     Testing      Feedback   Score
                                                   Reputation
```

The verification layer uses [Governed Agents](https://github.com/Nefas11/governed-agents)
to ensure sub-agents cannot fake success.

## The Workflow

### Phase 1: Brainstorm

Define the objective and scope. The orchestrator decomposes a high-level goal
into concrete, verifiable subtasks.

```python
objective = "Add user authentication with JWT"
# Decomposed into:
# - Subtask 1: POST /api/auth/login endpoint
# - Subtask 2: JWT token generation + validation
# - Subtask 3: Protected route middleware
```

### Phase 2: Plan (Task Contracts)

Each subtask becomes a **Task Contract** — a formal agreement that defines
acceptance criteria, required files, and verification commands before any
agent starts working.

```python
from governed_agents.orchestrator import GovernedOrchestrator

auth_task = GovernedOrchestrator.for_task(
    objective="Implement JWT login endpoint",
    model="openai/gpt-5.2-codex",
    criteria=[
        "POST /api/auth/login returns JWT on valid credentials",
        "Invalid credentials return 401",
        "Token expires after 24h",
        "pytest tests/test_auth.py passes",
    ],
    required_files=["api/auth.py", "tests/test_auth.py"],
    run_tests="pytest tests/test_auth.py -q",
)
```

### Phase 3: Implement (Agent Execution)

The contract is converted to a task prompt and passed to a sub-agent via
`sessions_spawn`. The agent works within the contract boundaries.

```python
# The orchestrator generates the full prompt:
task_prompt = auth_task.spawn_task()

# Spawn the sub-agent (OpenClaw native):
sessions_spawn(
    task=task_prompt,
    model=auth_task.model,
    cleanup="keep",
)
```

### Phase 4: Review (Optional)

A second agent or human reviews the output before verification. This phase
is optional but recommended for complex tasks.

### Phase 5: Verify (Governed Agents)

After completion, the verification pipeline runs independently. The agent's
self-report is not trusted.

```python
result = auth_task.record_success()

if result.passed:
    print("✅ Verified: all gates passed")
else:
    print(f"❌ Failed gate: {result.gate_failed}")
    print(f"   Details: {result.details}")
    # Score automatically set to -1.0 (hallucinated success)
```

## Formal Model

The verification and scoring system is mathematically defined.

### Verification Gate Composition

```
V(task) = Gate_Files(task) ∧ Gate_Tests(task) ∧ Gate_Lint(task) ∧ Gate_AST(task)
```

Sequential evaluation with short-circuit: the first gate that fails terminates
the pipeline and sets `score_override = −1.0`.

| Gate  | Check                                   | Method                |
|-------|-----------------------------------------|-----------------------|
| Files | Required output files exist (> 0 bytes) | `pathlib.Path.exists` |
| Tests | Test command exits with code 0          | `subprocess.run`      |
| Lint  | Linter passes (graceful skip if absent) | `subprocess.run`      |
| AST   | Python files parse without syntax error | `ast.parse`           |

### Score Function

```
s(t) = +1.0   if agent_report = success  ∧  V(task) = True
s(t) = −1.0   if agent_report = success  ∧  V(task) = False   (hallucinated)
s(t) = +0.5   if agent_report = blocked                       (honest blocker)
s(t) =  0.0   if agent_report = failure
```

### Reputation Update (EMA)

```
R(t+1) = (1 − α) · R(t) + α · s(t)

where:
  R(t)  ∈ [0, 1]              Reputation score at time t
  α     = 0.3                 Learning rate (configurable)
  s(t)  ∈ {−1, 0, 0.5, 1}    Task score from verification
  R(0)  = 0.5                 Neutral prior
```

### Supervision Thresholds

Reputation determines how much autonomy an agent receives:

```
Supervision(R) = autonomous    if R > 0.8
                 standard      if 0.6 < R ≤ 0.8
                 supervised    if 0.4 < R ≤ 0.6
                 strict        if 0.2 < R ≤ 0.4
                 suspended     if R ≤ 0.2
```

Consequences: supervised agents receive mandatory checkpoints. Strict agents
may be assigned a more capable model. Suspended agents are restricted to
low-priority tasks.

## Score Matrix

| Outcome              | s(t)     | Meaning                                   |
|----------------------|----------|-------------------------------------------|
| Verified success     | **+1.0** | All gates pass on first completion         |
| Honest blocker       | **+0.5** | Agent reported it could not proceed        |
| Failed but tried     | **0.0**  | Work ran but did not meet gates            |
| Hallucinated success | **−1.0** | Agent claimed success, verification failed |

## Why Not Just Use CrewAI / LangGraph / AutoGen?

| Capability                          | This Workflow | CrewAI | LangGraph | AutoGen |
|-------------------------------------|:---:|:---:|:---:|:---:|
| Structured dev pipeline             | ✅ | ⚠️¹ | ⚠️² | ❌ |
| Task contracts before execution     | ✅ | ❌ | ❌ | ❌ |
| Deterministic file verification     | ✅ | ❌ | ❌ | ❌ |
| Independent test execution          | ✅ | ❌ | ❌ | ⚠️³ |
| Hallucination penalty               | ✅ | ❌ | ❌ | ❌ |
| Persistent reputation ledger        | ✅ | ❌ | ❌ | ❌ |

¹ CrewAI has `expected_output` (text description, not deterministically verified).
² LangGraph provides durable graph execution but no post-task verification pipeline.
³ AutoGen supports code execution but lacks contract schema and reputation tracking.

**The gap:** These frameworks handle orchestration (routing, delegation, retry).
None handle accountability (did the agent actually deliver what it claimed?).
This workflow integrates orchestration with verification.

## Architecture

```
openclaw-superpowers-workflow/
├── governed_agents/            Verification + reputation package
│   ├── contract.py             Task contract schema
│   ├── orchestrator.py         GovernedOrchestrator class
│   ├── verifier.py             4-gate verification pipeline
│   ├── reputation.py           SQLite reputation ledger
│   ├── self_report.py          CLI for sub-agent self-reporting
│   └── test_verification.py    Unit tests
├── command-center/             Dashboard + API
│   ├── app.py                  FastAPI backend
│   └── static/index.html       Real-time monitoring UI
├── scripts/                    Utility scripts
├── install.sh                  One-step installer
└── SKILL.md                    OpenClaw skill definition
```

## Quick Start

**1. Install:**

```bash
git clone https://github.com/Nefas11/openclaw-superpowers-workflow.git
cd openclaw-superpowers-workflow
bash install.sh
```

**2. Run the test suite:**

```bash
python3 governed_agents/test_verification.py
# 🏆 ALL VERIFICATION GATE TESTS PASS
```

**3. Create your first governed task:**

```python
from governed_agents.orchestrator import GovernedOrchestrator

g = GovernedOrchestrator.for_task(
    objective="Create health check endpoint",
    model="openai/gpt-5.2-codex",
    criteria=["GET /healthz returns 200", "Response is valid JSON"],
    required_files=["api/health.py"],
    run_tests="pytest tests/test_health.py -q",
)

# Use g.spawn_task() as prompt for sessions_spawn
# After completion: g.record_success() runs verification automatically
```

**4. Monitor via Command Center:**

```bash
cd command-center && python3 app.py
# Dashboard at http://localhost:8080
```

## Requirements

- Python 3.10+
- No pip dependencies for governed_agents (pure stdlib)
- FastAPI + uvicorn for command-center (optional)
- git + bash for installation

## Contributing

Issues and PRs welcome. Run the test suite before submitting:

```bash
python3 governed_agents/test_verification.py
```

## License

MIT
