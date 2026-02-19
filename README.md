# OpenClaw Superpowers Workflow

A complete Subagent-Driven Development workflow system for OpenClaw, adapted from [obra/superpowers](https://github.com/obra/superpowers).

## 🎯 What This Is

Structured workflows for AI-assisted software development using OpenClaw:
- **Brainstorming** → Clarify requirements before coding
- **Planning** → Break features into 2-5 minute verifiable tasks
- **Implementation** → Coder sub-agents with self-reflection
- **Review** → Reviewer code reviews with severity levels
- **Verification** → Prove it works, don't just say it
- **Governed Agents** → Accountable sub-agent orchestration with automatic verification gates and reputation tracking

## 🚀 Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/hardy/openclaw-superpowers-workflow.git

# 2. Copy skills to your OpenClaw workspace
cp -r skills/* ~/.openclaw/workspace/skills/

# 3. Copy code review integration
cp reviewer-integration/code-review-skills.md ~/.openclaw/agents/[your-reviewer-agent]/agent/

# 4. Read the workflow guide
cat docs/subagent-driven-development.md
```

## 📁 Structure

```
.
├── skills/
│   ├── superpowers-adapted/     # From obra/superpowers
│   │   ├── test-driven-development.md
│   │   ├── testing-anti-patterns.md
│   │   └── verification-before-completion.md
│   ├── brainstorming/            # OpenClaw-specific
│   │   └── SKILL.md
│   ├── writing-plans/            # OpenClaw-specific
│   │   └── SKILL.md
│   └── governed-agents/          # Accountable sub-agent orchestration
│       ├── SKILL.md
│       ├── install.sh            # One-step installer
│       ├── README.md
│       └── governed_agents/      # Python package (zero deps)
├── docs/
│   └── subagent-driven-development.md  # Workflow guide
├── reviewer-integration/
│   └── code-review-skills.md     # Code review agent config
└── examples/
    └── e2e-test/                 # Health Check Endpoint example
        ├── design.md
        └── plan.md
```

## 🔄 The Workflow

```
User: "I want to add [feature]"
  ↓
Main Agent
  ↓
Brainstorming Skill → Design Doc
  ↓
Writing Plans Skill → Implementation Plan
  ↓
For Each Task:
  - Dispatch Coder (Implementation + Self-Review)
  - Dispatch Reviewer (Code Review)
  - Fix Loop if REJECT
  - Commit if APPROVE
  ↓
Final Verification
```

## ✅ Quality Gates

**No skipping allowed:**
1. **Self-Reflection** → Implementer reviews own work
2. **Reviewer Review** → Mandatory with checklist
3. **Verification Output** → Show, don't tell

## 📊 Results

From our E2E test (Health Check Endpoint):
- **Before:** 3-5 bugs per feature, 60+ min total (with rewrites)
- **After:** 0-1 bugs per feature, 40 min total (first try)

## 🛠️ Requirements

- OpenClaw (v2026.2.13+)
- Multiple agent configs (main, coder, reviewer)
- Git for commit history

## 📖 Learn More

- [Original Superpowers](https://github.com/obra/superpowers) by Jesse Vincent
- [Workflow Documentation](docs/subagent-driven-development.md)

## 🛡️ Governed Agents

The problem no one had solved: AI sub-agents can claim "success" without delivering.
Governed Agents makes hallucinated success detectable and penalized automatically.

```bash
# Install
bash skills/governed-agents/install.sh

# Use
from governed_agents.orchestrator import GovernedOrchestrator

g = GovernedOrchestrator.for_task(
    objective="Add rate limiter",
    model="openai/gpt-5.2-codex",
    criteria=["api/rate_limiter.py exists", "tests pass"],
    required_files=["api/rate_limiter.py"],
    run_tests="pytest tests/test_rate_limiter.py",
)
# After sub-agent completes:
result = g.record_success()  # Verifies independently. Hallucinated success → score -1.0
```

See [skills/governed-agents/README.md](skills/governed-agents/README.md) for full documentation.
- [Example: Health Check Endpoint](examples/e2e-test/)

## 📝 License

MIT License - see [LICENSE](LICENSE)

## 🙏 Credits

- **Jesse Vincent** for creating [Superpowers](https://github.com/obra/superpowers)
- **Peter Steinberger** (@steipete) for creating OpenClaw
- Community contribution for OpenClaw-specific adaptations

---

**Status:** Production-ready ✅  
**Last Updated:** 2026-02-17
