# Subagent-Driven Development Workflow

## Usage

For features requiring >3 files or >30 min work:

```bash
# 1. Trigger Brainstorming
"I want to add [feature]"

# 2. Senox asks clarifying questions
# 3. After design approval, Planning starts
# 4. Senox orchestrates Sub-Agents:
#    - Codex: Implementation
#    - DrNeuron: Code Review
#    - Repeat per task
# 5. Final verification + merge
```

## Skills Used

- brainstorming
- writing-plans
- test-driven-development
- testing-anti-patterns
- verification-before-completion

## Quality Gates

1. Self-Reflection (implementer must review own work)
2. DrNeuron Review (mandatory, no skips)
3. Verification Output (show, don't tell)

## Example Timeline

Feature: Add caching (3 tasks)
- Brainstorming: 5 min
- Planning: 10 min
- Task 1 (implement + review): 8 min
- Task 2 (implement + review): 6 min
- Task 3 (implement + review): 7 min
- Final verification: 2 min
**Total:** ~40 min

vs. Ad-hoc coding: 25 min (aber 3 Bugs, 2 Rewrites) → Real Total: 60+ min

## When to Use

✅ Use when:
- Feature touches >3 files
- Complex logic
- Multiple approaches possible
- High correctness requirement

❌ Skip when:
- Typo fix
- Comment update
- 1-liner bug fix

## Deliverables Checklist

- [ ] Design doc created
- [ ] Implementation plan with verification steps
- [ ] All tasks implemented
- [ ] Code reviews passed
- [ ] Tests pass
- [ ] Commits clean (Conventional Commits)
