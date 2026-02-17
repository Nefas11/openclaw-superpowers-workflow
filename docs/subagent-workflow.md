# Subagent-Driven Development Workflow

## Overview

Main Agent (Senox/Sonnet) orchestriert Sub-Agents (Codex) für Task-Execution + Review-Loops.

## Steps

### 1. Load Plan
Read plan from `docs/plans/[date]-[feature].md`

### 2. For Each Task

#### 2a. Dispatch Implementation Sub-Agent (Codex)
Prompt Template:
```
You are implementing Task N from the plan.

BEFORE starting:
1. Read these skills:
   - ~/.openclaw/workspace/skills/superpowers-adapted/test-driven-development.md
   - ~/.openclaw/workspace/skills/superpowers-adapted/verification-before-completion.md

2. Read Task N from docs/plans/[plan-file].md

IMPLEMENTATION:
- Follow RED-GREEN-REFACTOR (write test first)
- Implement minimal code to pass test
- Run verification command from plan

BEFORE reporting Done:
- Self-Reflection:
  * Does this solve the task as specified?
  * Edge cases covered?
  * Tests pass?
- If issues found during reflection: FIX THEM NOW
- Then report:
  * What you implemented
  * Test results (copy/paste output)
  * Self-reflection findings
  * Files changed

Do NOT say "Done" without verification output.
```

#### 2b. Review Implementation (DrNeuron)
Prompt Template:
```
Code Review for Task N.

BEFORE reviewing:
1. Read:
   - docs/plans/[plan-file].md (Task N section)
   - ~/.openclaw/workspace/skills/superpowers-adapted/testing-anti-patterns.md
   - [files changed] (from implementer report)

REVIEW CHECKLIST:
[ ] Task matches plan spec?
[ ] Tests exist and pass?
[ ] No testing anti-patterns?
[ ] Edge cases handled?
[ ] Code quality acceptable?

REPORT:
- Issues found (CRITICAL / HIGH / MED / LOW)
- Recommendation: APPROVE / REJECT / REVISE

If CRITICAL/HIGH issues: REJECT, list fixes needed.
```

#### 2c. Fix Loop (if REJECT)
If DrNeuron rejects:
1. Dispatch Fixer Sub-Agent (Codex) with DrNeuron's feedback
2. Re-run DrNeuron Review
3. Repeat until APPROVE

### 3. Commit
After APPROVE:
```bash
git add [files]
git commit -m "feat: [task description]"
```

### 4. Next Task
Repeat Step 2 for next task.

### 5. Final Verification
After all tasks:
```bash
npm test  # or equivalent
# All tests must pass
```

## Quality Gates

**No skipping:**
- Self-Reflection (implementer)
- DrNeuron Review
- Verification command output

**If gate fails:**
- Fix immediately
- Don't proceed to next task

## Example Session

```
[Senox] Loading plan: docs/plans/2026-02-17-add-caching.md
[Senox] Dispatching Task 1 implementation (Codex)...
[Codex] Read skills: TDD, Verification
[Codex] Implementing Task 1...
[Codex] Tests pass. Self-reflection: Looks good. Report: [...]
[Senox] Dispatching Code Review (DrNeuron)...
[DrNeuron] CRITICAL: No edge case test for null key. REJECT.
[Senox] Dispatching Fixer (Codex) with feedback...
[Codex] Added null-key test. Tests pass. Report: [...]
[Senox] Re-review (DrNeuron)...
[DrNeuron] APPROVE.
[Senox] Committing Task 1...
[Senox] Next: Task 2...
```
