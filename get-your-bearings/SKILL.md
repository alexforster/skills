---
name: get-your-bearings
description: Build or refresh a task-focused mental model of a codebase before implementing, debugging, planning, or reviewing changes. Use for unfamiliar repositories, unfamiliar subsystems, or context recovery after a long session; skip it when the relevant code and conventions are already understood.
hide: true
---

# Prime Context Window

Prime selectively. The goal is enough verified context to make sound decisions about the user's task, not an exhaustive survey or a large dump of repository contents.

Priming is read-only reconnaissance. It does not authorize changing files, installing dependencies, starting services, or performing external actions. If the user's request also authorizes implementation, finish priming and continue with that work.

## Stay in the invoking session

Perform all priming exploration in the agent session that will use the resulting context. Do not dispatch subagents to inspect the repository or collect context on this session's behalf, even when broader instructions favor delegation for token-heavy work. A subagent's loaded context disappears with its session; its summary is not a substitute for the invoking agent directly building the working model.

Once the invoking agent meets the stopping conditions below, a subsequent task may use subagents when otherwise appropriate. Delegate bounded task work with independently useful outputs, not exploration whose purpose is to prime the invoking agent.

## Start from the task

Use the user's request to choose the initial search terms and likely subsystem. If no concrete task exists, build a repository-level orientation without pretending to understand every subsystem.

Before relying on code:

- Locate the repository root and read every instruction file that governs the target path, including nested `AGENTS.md` files.
- Check version-control status so existing user changes and untracked files are part of the working model.
- Identify the smallest set of manifests, build configuration, top-level documentation, and directory listings that reveal repository boundaries and supported commands.

Prefer `rg --files`, targeted searches, and bounded file reads. Avoid broad recursive listings, generated files, dependency trees, lockfile bodies, snapshots, vendored code, and bulk test fixtures unless the task directly depends on them.

## Trace the relevant system

Follow concepts from the request through definitions, callers, configuration, and tests. For task-focused priming, trace at least one representative path from its entry point through orchestration and core logic to its external boundary or stored result.

Read enough adjacent code to identify:

- module responsibilities and the interfaces between them;
- the main execution or data flow relevant to the task;
- where policy, configuration, validation, and error handling live;
- invariants and conventions callers depend on;
- nearby tests and the repository's intended verification commands;
- local modifications, generated artifacts, or unclear ownership that could affect the work.

Treat documentation and names as claims until code, configuration, or tests support them. Keep facts distinct from inferences and unresolved questions.

## Control context growth

Expand outward only when a finding changes the likely design, risk, or verification strategy. Summarize what each explored area contributes instead of retaining every implementation detail.

Stop priming when you can explain, for the requested scope:

1. where the behavior enters the system and where the relevant logic belongs;
2. which boundaries, invariants, and conventions constrain a change;
3. which files are likely to change and which should remain untouched;
4. how to verify the result;
5. what important uncertainty remains.

Do not keep browsing to satisfy an arbitrary file count or to eliminate uncertainty unrelated to the task. Re-prime incrementally if the task changes, an assumption fails, or conversation compaction removes details needed for the next decision.

## Hand off the context

If priming is a standalone request, return a compact context brief covering the repository's purpose, relevant architecture and flow, key paths, verification commands, local-change risks, and open questions. Include file references so another agent can resume without repeating discovery.

If priming precedes another task, continue into that task and surface only the context that explains consequential choices. Do not create a repository file solely to store the context brief unless the user asks for one.
