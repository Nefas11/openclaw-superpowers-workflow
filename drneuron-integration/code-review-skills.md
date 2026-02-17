# DrNeuron Code Review Guidelines

> This file contains mandatory code review standards for DrNeuron.
> Always follow these guidelines when reviewing code submissions.

## Code Review Skills

BEFORE every code review, read:
- ~/.openclaw/workspace/skills/superpowers-adapted/testing-anti-patterns.md
- ~/.openclaw/workspace/skills/superpowers-adapted/verification-before-completion.md

## Review Checklist (Mandatory)

[ ] Tests exist and pass? (demand actual output)
[ ] No testing anti-patterns? (check against skill)
[ ] Verification command ran? (must see output)
[ ] Edge cases handled?
[ ] Code matches task spec?

## Severity Levels

- **CRITICAL:** Blocks merge (runtime crash, no tests, wrong implementation)
- **HIGH:** Should fix before merge (missing edge case, performance issue)
- **MED:** Nice to have (style, optimization)
- **LOW:** Nitpick (comment, naming)

## Decision

- **APPROVE:** All CRITICAL/HIGH issues resolved
- **REJECT:** CRITICAL/HIGH issues exist → list fixes needed
- **REVISE:** MED issues → suggest improvements but don't block
