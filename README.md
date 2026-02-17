# OpenClaw Superpowers Workflow

A complete Subagent-Driven Development workflow system for OpenClaw, adapted from [obra/superpowers](https://github.com/obra/superpowers).

## 🎯 What This Is

Structured workflows for AI-assisted software development using OpenClaw:
- **Brainstorming** → Clarify requirements before coding
- **Planning** → Break features into 2-5 minute verifiable tasks  
- **Implementation** → Codex sub-agents with self-reflection
- **Review** → DrNeuron code reviews with severity levels
- **Verification** → Prove it works, don't just say it

## 🚀 Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/hardy/openclaw-superpowers-workflow.git

# 2. Copy skills to your OpenClaw workspace
cp -r skills/* ~/.openclaw/workspace/skills/

# 3. Copy DrNeuron integration
cp drneuron-integration/code-review-skills.md ~/.openclaw/agents/drneuron/agent/

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
│   └── writing-plans/            # OpenClaw-specific
│       └── SKILL.md
├── docs/
│   └── subagent-driven-development.md  # Workflow guide
├── drneuron-integration/
│   └── code-review-skills.md     # DrNeuron code review config
└── examples/
    └── e2e-test/                 # Health Check Endpoint example
        ├── design.md
        └── plan.md
```

## 🔄 The Workflow

```
User: "I want to add [feature]"
  ↓
Senox (Main Agent)
  ↓
Brainstorming Skill → Design Doc
  ↓
Writing Plans Skill → Implementation Plan
  ↓
For Each Task:
  - Dispatch Codex (Implementation + Self-Review)
  - Dispatch DrNeuron (Code Review)
  - Fix Loop if REJECT
  - Commit if APPROVE
  ↓
Final Verification
```

## ✅ Quality Gates

**No skipping allowed:**
1. **Self-Reflection** → Implementer reviews own work
2. **DrNeuron Review** → Mandatory with checklist
3. **Verification Output** → Show, don't tell

## 📊 Results

From our E2E test (Health Check Endpoint):
- **Before:** 3-5 bugs per feature, 60+ min total (with rewrites)
- **After:** 0-1 bugs per feature, 40 min total (first try)

## 🛠️ Requirements

- OpenClaw (v2026.2.13+)
- Multiple agent configs (main, drneuron)
- Git for commit history

## 📖 Learn More

- [Original Superpowers](https://github.com/obra/superpowers) by Jesse Vincent
- [Workflow Documentation](docs/subagent-driven-development.md)
- [Example: Health Check Endpoint](examples/e2e-test/)

## 📝 License

MIT License - see [LICENSE](LICENSE)

## 🙏 Credits

- **Jesse Vincent** for creating [Superpowers](https://github.com/obra/superpowers)
- **Peter Steinberger** for OpenClaw
- **Hardy** for OpenClaw-specific adaptations

---

**Status:** Production-ready ✅  
**Last Updated:** 2026-02-17
