# Routing Examples

These traces show the expected level of context selection. They are examples of reasoning structure, not rigid scripts.

## Fix a PySide2 GUI crash in an EDA application

Classify the request as debugging plus GUI plus EDA. Load `engineering-core`, `debugging`, `qt-gui`, `eda-development`, and the active project context. Retrieve memories about maintainability and constructive challenge, but do not load PWA, SystemVerilog, or generic AI-agent material.

Inspect the traceback, event-loop boundaries, thread ownership, QObject lifetime, signal and slot connections, EDA tool callbacks, and the smallest reproducible case. First reproduce the crash under the project's pinned Python, PySide2, and EDA-tool versions. Then isolate whether the failure is caused by a deleted QObject, cross-thread GUI access, re-entrant callback, invalid model index, native extension failure, or an external tool response. Prefer a local fix over a broad rewrite. Add a regression test or a deterministic reproduction harness where feasible, then verify the fix with the project's canonical commands. Report the root cause, changed lifecycle or threading rule, evidence, and residual risk.

If the user corrects the preferred fix during the task, apply the correction locally and propose a durable Qt or project-memory update only if it represents a repeatable rule.

## Build a small offline-first PWA

Classify the request as web application plus offline-first architecture plus implementation. Load `engineering-core`, `web-pwa`, and any project context. Add `security` only if authentication, encryption, untrusted sync, or sensitive data is involved. Do not load Qt, EDA, or SystemVerilog skills.

Clarify the data model, offline operations, conflict policy, supported browsers, installability target, storage limits, and whether synchronization is required. Prefer the smallest architecture that meets those constraints: local persistence, explicit sync boundaries, observable pending operations, deterministic conflict handling, and a clear recovery path. Do not claim “offline-first” if only static assets are cached. Verify cold-start behavior without network, reload persistence, queue replay, conflict cases, and online-to-offline transitions. Record project-specific browser or deployment constraints in context, not in global memory.

## Review a C++ GitHub pull request

Classify the request as code review plus C++ plus GitHub artifact. Load `engineering-core`, `cpp-review`, and project context. Add `security` or `concurrency` only when the diff touches those concerns.

Read the diff and surrounding code before forming conclusions. Prioritize correctness, undefined behavior, ownership and lifetime, data races, exception safety, API compatibility, error handling, tests, and maintainability. Separate blocking findings from suggestions and cite file and line ranges. Check whether the proposed abstraction reduces complexity or merely relocates it. State assumptions when repository context is incomplete. End with a concise approval recommendation and a verification gap list; do not rewrite the entire patch unless requested.

## Design an AI agent that generates and verifies SystemVerilog

Classify the request as AI-agent design plus SystemVerilog verification plus tool orchestration. Load `engineering-core`, `ai-agent-design`, `verification-systemverilog`, and project context. Add `security` when generated code or tools can access sensitive repositories or execute commands.

Define the contract before the agent loop: inputs, supported HDL subset, target simulator, reference model, assertions, coverage goals, timeout limits, and acceptance criteria. Use a staged pipeline: specification extraction, candidate generation, static checks, compilation, simulation, assertion evaluation, coverage analysis, counterexample summarization, and bounded repair. Keep generated artifacts and test evidence separate from durable memory. Require deterministic seeds, reproducible commands, sandboxed tool execution, and a stop condition for repeated failed repairs. Evaluate both functional correctness and process quality, including whether the agent knows when it lacks evidence.

Do not store generated HDL, waveforms, tokens, or simulator logs as global memory. Keep them in the project workspace and retain only a concise, approved project decision or reusable verification procedure.
