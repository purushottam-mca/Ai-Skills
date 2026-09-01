# Governance in Practice

This walkthrough shows the difference between **acting on a correction now** and **learning from it durably**. The agent should never silently convert a local correction into a global rule.

## The five-state loop

```text
User correction
      ↓
Apply locally to the current task
      ↓
Classify: one-off, project fact, reusable procedure, or stable preference
      ↓
Create a proposal with evidence and risks
      ↓
User approves, rejects, or revises
      ↓
Make the smallest approved change and record it
```

The important boundary is between the second and fourth steps. The agent can follow a correction immediately, but durable state remains unchanged until the user approves a sufficiently justified proposal.

## Scenario 1: One-off implementation preference

**User correction:** “For this legacy EDA plugin, do not introduce a new dependency. Patch the existing callback instead.”

**Immediate behavior:** The agent stops proposing a new dependency and continues the current fix using the existing callback path. It records the constraint in task notes or the active project's context if it is verified as a project rule.

**Durable decision:** No global memory proposal is created. The preference may be specific to this repository and this legacy plugin. If the project consistently prohibits new dependencies, the agent can propose a project-context entry:

```markdown
---
id: 2026-08-19-eda-no-new-dependency
kind: project-update
status: proposed
scope: project
confidence: high
source: explicit-request
created: 2026-08-19
---

# Proposed change

For the EDA plugin, prefer fixes using existing dependencies. Introduce a new dependency only after an explicit project decision.

## Evidence

The user gave this instruction while fixing the legacy callback path.

## Risk

This rule may be too broad if the project later adopts a dependency-management policy.

## Approval question

Approve as a project rule, reject it, or narrow it to the current plugin?
```

## Scenario 2: Repeated collaboration preference becomes memory

**Observed corrections:** Across several unrelated Python, C++, and PWA tasks, the user asks the agent to avoid speculative abstractions, show trade-offs, and prefer the smallest maintainable design.

**Immediate behavior:** The agent follows that preference in each current task.

**Proposal:** After repeated evidence, it proposes a global memory rather than editing every domain skill:

```markdown
---
id: prefer-simple-maintainable
kind: principle
scope: global
status: proposed
confidence: high
source: repeated-correction
created: 2026-08-19
last_confirmed: 2026-08-19
review_after: 2027-02-19
---

# Prefer simple, maintainable solutions

When multiple approaches meet the requirements, prefer the one with fewer moving parts, clearer failure modes, and lower maintenance cost. Explicitly challenge unnecessary abstraction and state the trade-offs.

## Exceptions

A more complex design is acceptable when the simpler option fails a stated requirement, creates a material security or reliability risk, or cannot meet verified scale and performance constraints.

## Approval question

Approve this as a global engineering principle?
```

**Approval:** The user says “Approve.” The agent then changes `memory/engineering-principles.md`, changes the proposal status to `approved`, updates `last_confirmed`, and adds one entry to `CHANGELOG.md`. It does not rewrite Python, Qt, C++, or PWA skills individually.

## Scenario 3: Correction becomes a reusable skill improvement

**User correction:** “Do not call Qt widget methods from a worker thread. Marshal the update to the GUI thread and add a regression test.”

**Immediate behavior:** The agent applies the rule to the current PySide2 crash fix, verifies thread affinity, and adds the requested test if the project supports it.

**Promotion decision:** This is a reusable Qt procedure, not merely a user preference. If it is already a stable rule, the agent proposes a skill update to `qt-gui/SKILL.md`:

```markdown
---
id: 2026-08-19-qt-gui-thread-affinity
kind: skill-update
status: proposed
scope: domain
confidence: high
source: explicit-request
created: 2026-08-19
---

# Proposed change

Add a non-negotiable verification step: identify the owning thread for every QObject touched by asynchronous code; marshal GUI mutations to the GUI thread; add a regression test for the failure mode when feasible.

## Evidence

The user corrected a worker-thread GUI update during a PySide2 crash investigation.

## Benefit

Makes a recurring failure mode explicit and improves future Qt debugging and review tasks.

## Risk and exception

Some non-visual QObject operations may be safe outside the GUI thread. The wording must distinguish widget mutations from thread-safe worker objects.

## Approval question

Approve this as a general Qt skill rule, revise its scope, or keep it project-local?
```

The agent must not add the rule globally merely because it sounds technically sensible. Approval confirms that the wording and scope match the user's intended engineering practice.

## Scenario 4: Rejecting a bad learning proposal

**Proposal:** “Always use Docker Compose for local development.”

**User response:** “Reject. Some small utilities should run directly with the system Python.”

**Action:** Mark the proposal `rejected`, retain the reason in the proposal record, and do not add the rule to memory or `engineering-core`. The agent may continue to recommend Docker Compose when project context or reproducibility requirements justify it.

This prevents a useful local habit from becoming an over-broad global instruction.

## Scenario 5: Revising a proposal

**Initial proposal:** “Always ask before adding dependencies.”

**User response:** “Revise: ask before adding runtime dependencies, but development-only tooling is fine when it is documented.”

**Action:** Keep the original proposal as `superseded`, create the revised text with the narrower scope, ask for approval again, and only then update the relevant project or global rule. The agent should not silently edit the original proposal into a different claim.

## What the agent says at task completion

A good completion report includes a short governance section:

> **Learning review:** Applied the user's no-new-dependency correction to this repository only. No global memory change proposed because the evidence is project-specific. Proposed one project-context rule for review: runtime dependencies require explicit approval; development-only tools remain allowed when documented.

If there is enough evidence for a durable update, use:

> **Learning review:** The same preference appeared in three unrelated tasks. Proposed global memory `prefer-simple-maintainable`. Current task behavior is complete; no durable files will change until approval.

## Guardrails

Never store credentials, tokens, private keys, raw logs, generated artifacts, sensitive personal data, or speculative conclusions as durable memory. Never let a learned preference override a current explicit request, verified repository behavior, safety constraints, or a narrower project rule. When evidence is ambiguous, keep the proposal low-confidence or ask a clarifying question instead of guessing.
