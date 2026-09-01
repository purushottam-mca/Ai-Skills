# Portable Schemas and Templates

Keep schemas deliberately small. YAML frontmatter provides discovery metadata; Markdown body provides instructions that remain readable in any agent.

## Skill template

```markdown
---
name: skill-name
description: What this skill does and when to use it.
---

# Skill title

## Use when

State the trigger conditions and the boundary of the skill.

## Inputs and outputs

Describe the artifacts, assumptions, and expected result.

## Workflow

1. Inspect the relevant context.
2. Select the appropriate variant.
3. Execute the procedure.
4. Verify the result.

## Decision rules

State preferred defaults, trade-offs, and stop conditions.

## Dependencies

- `requires`: optional skill IDs that are genuinely needed.
- `compatible_with`: skills that compose without prescribing the same decision.
- `conflicts_with`: skills whose instructions must be reconciled before use.

## References

- [variant.md](references/variant.md): Read only for that variant.

## Verification checklist

State how to determine whether the work is complete.
```

Keep the body under 500 lines. Put framework-specific detail, long examples, API tables, and generated boilerplate in one-level-deep `references/` files. Use `scripts/` only for deterministic operations that would otherwise be rewritten repeatedly; test every bundled script.

## Memory template

```markdown
---
id: memory-id
kind: preference | principle | constraint
scope: global | domain | project
status: approved | proposed | deprecated
confidence: high | medium | low
source: user-stated | repeated-correction | approved-inference
created: YYYY-MM-DD
last_confirmed: YYYY-MM-DD
review_after: YYYY-MM-DD
---

# Short statement

The durable rule, written as a concise, testable statement.

## Evidence

Summarize the supporting user statement or repeated examples without copying sensitive conversation data.

## Apply when

Describe the tags, tasks, and conditions where this memory is relevant.

## Exceptions

State when explicit current instructions or project context should supersede it.
```

A memory should express one durable rule. If it needs many exceptions, it is probably a skill or project rule instead.

## Project-context template

```markdown
---
project: project-name
scope: project
status: active | archived
last_reviewed: YYYY-MM-DD
---

# Project context

## Purpose and current objective

Describe what the project does and what is currently being changed.

## Stack and versions

List languages, frameworks, operating systems, EDA tools, databases, and pinned versions.

## Repository map

Record important directories, entry points, generated files, and ownership boundaries.

## Commands

Provide canonical setup, test, lint, build, package, and verification commands.

## Local conventions

Document naming, architecture, error handling, GUI, threading, commit, and review conventions.

## Constraints and known hazards

Record compatibility limits, offline requirements, licensing, security boundaries, flaky tests, and known defects.

## Decisions

Link to accepted decisions and state which generic skill behavior they override.

## Evidence freshness

Mark facts as verified, inferred, or stale. Remove obsolete facts instead of accumulating history.
```

## Catalog template

```yaml
skills:
  - id: debugging
    kind: skill
    path: skills/debugging/SKILL.md
    tags: [debugging, troubleshooting, testing]
    scope: global
    summary: Reproduce, isolate, instrument, fix, and verify failures.
    requires: [engineering-core]
    status: active
    last_reviewed: 2026-08-19
memories:
  - id: prefer-simple-maintainable
    kind: memory
    path: memory/engineering-principles.md
    tags: [architecture, design, review, maintenance]
    scope: global
    summary: Prefer the simplest maintainable solution and challenge unnecessary complexity.
    status: approved
    last_reviewed: 2026-08-19
```

The catalog is an index, not a second copy of the content. Keep summaries short enough for routing and keep the authoritative statement in the referenced file.
