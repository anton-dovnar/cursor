---
description: Design an implementation approach using the architect agent and coding standards skill, then execute with high-quality engineering practices.
---

# Implement Command

Use `/implement` when you want to go from requirements to an implementation that is consistent with the repository’s standards.

## What This Command Does

1. **Clarify requirements** (inputs, outputs, constraints, edge cases)
2. **Propose an approach** (architecture, data flow, trade-offs)
3. **Draft a step-by-step plan** (phases, files to touch)
4. **Implement** following coding standards
5. **Review** for correctness, security, and maintainability

## References

- Architect agent: `~/.cursor/agents/architect.md`
- Coding standards skill: `~/cursor/skills/coding-standards/SKILL.md`

## Guidance

- Prefer small, focused changes and keep files cohesive
- Validate inputs and handle errors explicitly
- Avoid mutation where feasible; return new objects
