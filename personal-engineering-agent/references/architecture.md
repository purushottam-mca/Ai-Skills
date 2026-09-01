# Recommended Architecture

## Design judgment

A Skills + Memory system is useful, but treating it as a flat pile of Markdown files is flawed. The durable unit should be a **small, typed knowledge record with scope, provenance, confidence, and review status**, presented through Markdown so it remains portable. Skills describe how to perform a class of work; memories describe how to collaborate with this user; project context describes facts and constraints that are true only for one repository or environment.

The architecture should optimize for **retrieval quality and controlled change**, not for exhaustive documentation. A small catalog and explicit routing metadata are more valuable than dozens of loosely named files.

## Version 1 directory structure

```text
agent-system/
├── core/
│   ├── policy.md                 # stable operating rules and precedence
│   ├── routing.md                # tags, scoring, and loading procedure
│   └── catalog.yaml              # compact index of skills and memories
├── skills/
│   ├── python/
│   │   ├── SKILL.md
│   │   └── references/           # optional variant details
│   ├── qt-gui/
│   │   └── SKILL.md
│   ├── debugging/
│   │   └── SKILL.md
│   ├── cpp-review/
│   │   └── SKILL.md
│   ├── web-pwa/
│   │   └── SKILL.md
│   ├── eda-skill/
│   │   └── SKILL.md
│   ├── verification-systemverilog/
│   │   └── SKILL.md
│   └── ai-agent-design/
│       └── SKILL.md
├── memory/
│   ├── user-preferences.md       # a small curated set, not a diary
│   ├── engineering-principles.md # stable collaboration and design preferences
│   └── proposals/                # pending, approved, rejected, or expired
├── projects/
│   └── project-name/
│       ├── context.md            # repository-local facts and constraints
│       ├── decisions.md           # accepted architecture decisions
│       └── verification.md       # commands and latest evidence
└── CHANGELOG.md                  # human-readable approved durable changes
```

Use one file per coherent skill or memory category rather than one file per sentence. Split a file only when retrieval would otherwise load unrelated content. Keep the root catalog small enough to scan in one pass.

## Boundary rules

| Layer | Store here | Do not store here |
|---|---|---|
| Core | Precedence, routing, safety, update approval, naming conventions | Domain-specific implementation advice or personal trivia |
| Skills | Reusable procedures, heuristics, checklists, tool usage, domain patterns | Current repository paths, secrets, one-off fixes, personality preferences |
| Memory | Stable user preferences, recurring constraints, collaboration style, approved principles | Transient conversation details, credentials, guesses, project facts |
| Project context | Versions, build commands, architecture, local conventions, known failures, decisions | Global preferences or generic tutorials |
| Proposals | Candidate updates with evidence and review state | Unreviewed changes presented as facts |

Never store secrets, tokens, private keys, raw credentials, or sensitive data in this system. Do not store every conversation, every completed task, or speculative psychological conclusions. Keep temporary notes in the task workspace and let them expire.

## Initial skills

Start with a small set that composes around the user's recurring work:

| Skill | Why it belongs in Version 1 |
|---|---|
| `engineering-core` | Turns the user's preference for simple, maintainable solutions into a repeatable analysis and verification loop. |
| `debugging` | Cross-cutting procedure for reproducing, isolating, instrumenting, and verifying failures. |
| `python` | Frequent language with broad reuse across automation, GUI, AI, and data tools. |
| `cpp-review` | Covers correctness, ownership, API design, performance, and review communication. |
| `qt-gui` | Captures PySide2/PyQt5/Qt event-loop, threading, ownership, and UI testing patterns. |
| `eda-development` | Encodes domain constraints, tool interoperability, Tcl/Cadence SKILL, and reproducibility. |
| `web-pwa` | Covers offline-first/local-first web architecture, service workers, sync, and installability. |
| `verification-systemverilog` | Provides stimulus, assertions, reference models, coverage, and reproducibility guidance. |
| `ai-agent-design` | Handles tools, MCP, memory, evaluation, guardrails, and agent workflows. |

Defer Docker, homelab, security, DevOps, documentation, and LeetCode skills until repeated tasks reveal stable procedures. They can initially be supporting references under `engineering-core` or project context.

## Retrieval model

Represent each skill and memory in the catalog with `id`, `kind`, `tags`, `scope`, `summary`, `path`, `requires`, `status`, and `last_reviewed`. Route using a two-stage process: first select candidates from tags and artifact type; then read the candidates and keep only the minimal relevant sections.

A simple Version 1 score is sufficient:

```text
score = 4 * explicit_match
      + 3 * artifact_match
      + 2 * technology_match
      + 2 * project_match
      + 1 * task_type_match
      - 3 * scope_mismatch
```

Load the highest-scoring primary skill and supporting skills with a positive score. Cap the initial load at one primary skill, three supporting skills, the active project context, and five relevant memories. Increase the cap only when the task explicitly spans more domains.

## Composition and override model

Skills should be **loosely coupled**. A skill can state prerequisites such as `requires: [engineering-core, debugging]`, but the router should load a dependency only when its procedure is actually used. Prefer shared concepts and checklists over skills importing large amounts of one another.

Project context does not rewrite generic skills. It adds scoped constraints and can explicitly disable or replace a step, for example: “For this legacy Qt 5.12 build, use Python 3.8 and do not introduce Qt 6 APIs.” The agent must cite the project rule when applying the override and must not promote it to global memory without approval.

## What the agent should persist initially

Persist only a few high-value memories: the user prefers simple and maintainable solutions; wants assumptions and trade-offs challenged; values low context usage and composability; works across the listed engineering domains; prefers reproducible tooling and clear technical documentation; and wants durable changes to require approval. These are collaboration preferences and system-governance rules, not project facts.

Use confidence and review dates. A memory with no recent evidence should become a review candidate rather than silently gaining authority.
