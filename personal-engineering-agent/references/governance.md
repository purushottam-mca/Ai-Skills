# Controlled Learning and Change Governance

## Correction handling

Treat every correction as a scoped observation first. Apply it to the current task immediately, but do not infer a global preference from one correction. At task completion, classify it as one of the following:

| Classification | Action |
|---|---|
| One-off task instruction | Keep it in the task notes and let it expire. |
| Project fact or decision | Add it to the active project's context or decisions file. |
| Reusable procedure | Propose a skill change or a new reference section. |
| Stable collaboration preference | Propose a memory update. |
| Ambiguous or conflicting evidence | Ask for clarification or retain as a proposal with low confidence. |

## Proposal record

Create a proposal under `memory/proposals/` using this format:

```markdown
---
id: 2026-08-19-prefer-small-diffs
kind: memory-update | skill-update | project-update | deprecation
status: proposed
scope: global | domain | project
confidence: medium
source: user-correction | repeated-pattern | explicit-request
created: 2026-08-19
---

# Proposed change

## Current behavior

State what the system currently does.

## Proposed text or patch

Show the exact memory statement, skill section, or project-context entry.

## Evidence

Summarize at least one concrete example and distinguish observation from inference.

## Benefit

Explain what future tasks improve.

## Risks and exceptions

Explain overgeneralization, conflicts, maintenance cost, and rollback conditions.

## Approval question

Ask the user to approve, reject, or revise the proposal.
```

## Promotion thresholds

Promote a memory only when the user explicitly approves it, requests it directly, or repeats the same preference across multiple materially different tasks. Promote a skill change when the procedure is reusable, has a clear trigger, and has either been used successfully more than once or was explicitly requested as a standard. Promote project context when the fact is verified in repository files, tool output, or an explicit project decision.

Do not promote secrets, credentials, personal sensitive data, temporary states, speculative diagnoses, or instructions that exist solely to bypass safety or review. Do not create a new skill for one unusual task when an existing skill plus project context is sufficient.

## Review and expiry

Assign every proposal a status of `proposed`, `approved`, `rejected`, `deprecated`, or `superseded`. Set `review_after` for memories that may drift. Mark project facts stale when the repository or toolchain changes. Prefer deleting obsolete project facts to preserving an ever-growing history in active context.

Review proposed changes at the end of a task or during a deliberate maintenance session. Never alter active memory or skill files midway through an unrelated task without approval.

## Conflict and rollback

When a proposed update conflicts with an approved rule, keep both records visible and ask which scope should win. To roll back, restore the prior text, mark the new record `superseded`, and add a changelog entry explaining why. Never erase evidence needed to understand a bad update, but keep historical records out of the default retrieval path.

## Approval response protocol

Use a compact confirmation request:

> Proposed update: **[scope] [title]**. Evidence: **[short summary]**. Benefit: **[short summary]**. Risk: **[short summary]**. Approve, reject, or revise?

After approval, make the smallest change possible, validate the syntax and links, update `last_confirmed`, and record the change in `CHANGELOG.md`.
