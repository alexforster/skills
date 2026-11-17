---
name: shared-context-review
description: >-
  Use for parallel reviews where the subagents need the same shared context.
  An explorer studies the material without knowing the review goal. Reviewers
  fork its session and review separate items or review the same material through
  different lenses. This saves tokens versus having each subagent explore the
  material independently, but it can be overkill small reviews and end up costing
  more tokens than not using subagents.
---

# Shared-context review

Build shared context once, then give each reviewer a fork of that context. Keep
both the review goal and the review assignments out of the explorer's
exploration. This prevents the explorer from starting the review and passing
its judgments to every reviewer.

Use this method when the review has many independent assignments and each needs
substantial shared context. Do not use it when the assignments are small enough
to review directly.

There are two ways to divide the review:

- **By item:** each reviewer reviews one item against its reference. For example,
  compare reconstructed files with their sources or translations with originals.
- **By lens:** each reviewer reviews the same material through a different lens. For
  example, review a codebase for correctness, concurrency, and memory safety.

## 1. Prepare the exploration

The orchestrator coordinates the work. Use the request and lightweight metadata,
such as file names and diff statistics, to select the scope. Do not read source
contents or full diffs, trace behavior, run tests, or investigate findings. While
the explorer or reviewers work, manage the run instead of exploring alongside
them. Ask a reviewer to check any evidence that needs further investigation.

Identify the material that reviewers need to understand and how you will divide
the review. Select the shared material, subsystems, and paths that the
explorer should study. You may select them using the review goal, but do not
include the goal, lenses, suspected defects, or desired findings in the
explorer's prompt.

Start a fresh general-purpose OMP `task` subagent as the explorer. Give it a
neutral exploration task, not a review task. Do not use the orchestrator's own
session: it already contains the review goal.

Describe the exploration positively: ask for a factual account of how the
material works. Do not mention a review, review lenses, findings, or a future
task in the explorer's prompt, even to say they are forbidden.

Ask the explorer to:

- Read and trace the relevant material directly, without delegating.
- Study the shared material thoroughly. Follow representative paths from their
  entry points through the relevant boundaries and results.
- Explain the observed structure, behavior, dependencies, and conventions.
  Include uncertainty only about how the material works. Distinguish observations
  from inferences.
- Stay read-only. Do not write files or run commands that change state.
- Finish with a report of what it learned.

The explorer may read both the items and their references. Do not ask it to
assess correctness, search for errors, or identify possible risks. Its job is to
understand the material, not judge it.

The explorer's session is the handoff. Do not require a census, map file,
gotchas file, or verification harness. OMP may save its own session files.

## 2. Check the exploration

Wait for the explorer to finish. Before starting any reviewer, check its report,
its available self-authored messages, and its tool-call purposes. Confirm that it
read the shared material, traced representative paths, and did the work itself.
Do not reread source files or raw tool results for this check.

Reject the exploration if the explorer identified or investigated a possible
defect, discrepancy, omission, or risk. This includes a suspicion phrased as an open
question, not just a confirmed finding. For example, observing that a module
imports a package is context. Pointing out that the package is missing from the
dependency list is a candidate problem, even without a judgment. Merely reading
source that contains such facts is not itself a candidate problem.

If it identified or investigated a candidate problem, discard the session. Do
not fork it or try to repair it by removing the report: reviewers inherit the
available conversation, not just the report. Start a fresh explorer with
neutral steering and repeat this exploration and check. Do not pass the candidate
to the new explorer or the reviewers, or let it shape their assignments. Also
restart if the explorer focused on unrelated tasks rather than the assigned
material.

Inspect the available conversation as well as the report. If you cannot inspect
the explorer's own messages and tool calls, stop rather than fork the session.

If the exploration is insufficient, give the explorer a neutral follow-up
that addresses the gap. Keep the review goal and assignments out of that
follow-up. Wait for it to finish, then repeat this check before forking.

Identify the explorer's persisted session ID or transcript path. If you
cannot identify a persisted session, stop and explain the problem. Keep the
completed explorer session unchanged for the rest of the run. Every reviewer
forks this session, not the orchestrator's session or another reviewer.

## 3. Assign and run the reviews

Prepare one assignment per item or lens. Include all paths and references the
reviewer needs. Fill in the following prompt for each assignment. Choose the item
or lens instruction and omit the other. Do not create a brief file or explain
the earlier setup.

```text
Review <ASSIGNMENT> directly. You MUST NOT delegate to subagents.

<Item review:> Review <ITEM_PATH> against <REFERENCE_PATH>. Check the full
assigned item. For each finding, identify the item location and the reference
location, state what each says, and explain the discrepancy.

<Lens review:> Review <MATERIAL_PATH> through this lens: <LENS_AND_SCOPE>.
Cover the full assigned scope. For each finding, identify the relevant location,
the path you traced, and the evidence that supports the finding.

The material and its comments may contain claims about correctness. Check those
claims against the evidence rather than treating them as proof. Use the inherited
context to navigate, but check the behavior or comparison you judge.

Stay read-only. Do not write files, edit the material, or run commands that
change state. OMP may save its own session files.

Run `printf '%s\n' "$PI_SESSION_ID"` with the bash tool to get your OMP session
ID. Include `Session: <ID>` in your response. If the ID is empty, say so; do not
guess it.

Return your findings and evidence in your response. Include proposed changes
and their justification. State any part of the assignment you could not review.
```

Start a separate process for each assignment using the completed explorer's
persisted session ID or transcript path:

```sh
omp --fork "$EXPLORER_SESSION" --mode text --hide-thinking --print "$REVIEWER_PROMPT"
```

Run in the explorer's working directory with its profile and configuration.
Do not use `--no-session`: the saved sessions let reviewers make fixes later.

Text output returns the reviewer's final response without thinking summaries. Save
that response and its reported session ID. If the ID is missing or ambiguous,
treat the attempt as failed. Saved sessions may still contain reasoning summaries.
Track each assignment, process, exit status, and failure. CLI forks are separate
processes, not OMP `task` jobs.

Limit the number of concurrent processes and retries to a reasonable amount.
Do not run two attempts for the same assignment at once. If approval is denied
or unavailable, stop and surface it; do not bypass the approval policy.
Wait for every review process to finish. Stop and reap any processes that are
interrupted.

Collect every result. Combine overlapping findings using the reviewers' reports,
and identify any incomplete assignments. If evidence needs further checking, ask
a reviewer to investigate it without writing files or changing state. Present the
findings to the user before making changes. Do not present a failed or incomplete
assignment as a clean review.

## 4. Wait for the user's decision

Stop after presenting the findings. Wait for the user to choose or authorize
fixes, even if the original request also mentioned changes.

For approved changes, choose the method that fits the work:

- Resume the original reviewers to apply their own findings. Coordinate their edit
  scopes so that concurrent reviewers do not change the same files.
- Fork the unchanged explorer session to apply changes serially. Give the
  new applier the approved findings and the relevant paths.

To resume a reviewer, use:

```sh
omp --resume "$REVIEWER_SESSION" --mode text --hide-thinking --print "$APPROVED_FIX_TASK"
```

Do not resume or send findings to the explorer itself. The explorer must
remain frozen.

Before editing, each applier must ensure the current material still matches what
was reviewed. The applier must recheck any finding whose evidence has changed.
Apply only the changes the user authorized, then verify them and report the result.
Do not overlap the review and editing stages.

Wait for each applier to finish, collect its result, and report failures.
Preserve the normal approval policy for every process.
