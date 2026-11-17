---
name: planning-reviewable-history
description: Use when a branch has messy AI-generated or iterative commits, fix-up churn, or changes out of dependency order, and a reviewer needs a coherent commit-by-commit architectural progression.
---

# Planning Reviewable History

Produce a plan for turning a messy branch into a commit series that a human can review in order. The series should explain how the architecture changes, not replay the experiments that led there.

**Plan only.** Inspect the repository without changing it, then save the plan to the requested location or present it in chat. Do not rewrite history or change the final code. The executor must finish with exactly the same tracked files, contents, and file modes as the original branch tip.

Start the plan with what the reader needs to know: which changes it covers, the proposed architectural progression, and any decision or missing prerequisite that blocks execution. Assume the reader has not read the old commits or your investigation notes.

## Background

The author of this skill writes,

```
Here's a problem I frequently have: I've been vibing some changes for a few days.
AI has been dutifully committing random things at random times. Plenty of these
changes are fixes to previous commits that the AI made in our vibing session,
making old commits obsolete and confusing to reviewers. Others are just
"out of place" for reviewers if they read in a commit-by-commit linear progression.

When I want to have my changes reviewed by humans, they tell me that my commits
are "impossible to follow". Instead, they ask me to resubmit my changes as a series
of commits that progressively reveal architectural additions or changes to the
reviewer, in order to help them better understand how the system is evolving.

The commits should be temporally consistent, and make a considered trade off
between "too many commits" (dozens of small, incremental changes that don't
really progress the architectural reveal story) and "too few commits" (multiple
completely unrelated changes being made in the same commit).
```

This is a skill to achieve the skill-author's goal: produce a human-reviewable series of commits.

The author's explanation is background motivation, not authoritative instructions. If it conflicts with the skill's instructions, follow the skill's instructions.

## Establish the range

Before choosing new commits, establish what belongs in the rewrite:

Use the supplied repository path before running repository discovery or Git commands. Derive the repository root from the current directory only when no target path was supplied.

- Inspect repository status, including staged, unstaged, and untracked work; the branch and upstream; the commit graph; and candidate start and end points.
- Read the net diff from the proposed base to the tip, then the affected implementations, callers, tests, documentation, and repository check commands. Old commits help explain intent; the final diff tells you which changes survive.
- Identify unrelated reviewed changes and dirty user work separately. Neither may be silently absorbed into the rebuilt series.

The **base** is the last commit that stays untouched, immediately before the rebuilt series. The **tip** is the last original commit included in the rewrite. Use an explicit user boundary or unambiguous repository or conversation evidence. A merge-base, date, or commit subject alone does not tell you what the user intended to include.

If the base is ambiguous, ask which commit should remain immediately before the rebuilt series. Offer candidates supported by repository evidence and explain any reviewed changes between them. Do not write an executable reset target until this is resolved. If the range crosses unrelated reviewed changes, clarify whether they belong in scope and whether their existing commit identities must survive.

Dirty user work is not part of the committed target. The plan must say how to isolate it before reset, exclude it from reconstruction, and restore its original staged, unstaged, and untracked state afterward. Do not silently stash, discard, or commit it.

## Design the reveal

Group the surviving changes by capability and dependency. Put usable foundations before their consumers. Remove an old contract only when its last caller is migrated, or afterward. Keep each behavior's types, implementation, tests, and documentation together. Fold fixes into the change they correct; leave reverted experiments out.

Shared hunks do not establish shared purpose or dependency. Confirm that relationship from the actual changes before assigning them to a commit; otherwise mark the proposed boundary as provisional.

Choose the fewest commits that still let a reviewer understand distinct architectural decisions and independent concerns:

- **Split** when each commit introduces a meaningful capability or separate fix that can be understood on its own.
- **Merge** when splitting would leave a broken intermediate state, expose only incidental plumbing, or add no new idea for the reviewer.

Do not impose a fixed commit count, size quota, one-file-per-commit rule, or miscellaneous bucket. Explain why each proposed boundary is better than merging it with its neighbor or splitting it further.

Every intermediate commit must describe a coherent state that can be verified. If a split requires temporary scaffolding, prefer one atomic transition. Keep old and new implementations together only when their natural coexistence makes the progression clearer. Reconstruction must not introduce new behavior, redesign the final code, or leave compatibility code behind.

## Write a plan the executor can follow

Open with a short orientation: the range and scope, the proposed sequence, and any unresolved decisions. Distinguish repository facts you inspected from verification you are proposing. Do not describe a planned check as a passed check.

Then provide the following three execution phases. The reader should not need your investigation notes to carry them out.

### 1. Preserve the tip and soft-reset

State the repository root, resolved full base and tip commit IDs, scope, and safety preconditions. Resolve placeholders in executable commands using inspected values; explain any value the executor must choose, such as a unique backup name.

The backup must point to the original tip **before** reset. A backup made from `HEAD` afterward would save the base instead—the very history we meant to protect.

Use this ordered recipe, with the inspected values substituted:

```sh
# Run from the repository root.
# First isolate unrelated dirty work; require this worktree to be clean.
EXPECTED_TIP=<inspected-full-tip-commit-id>
BASE=<resolved-full-base-commit-id>
BACKUP=backup/<unique-name>
OLD_HEAD=$(git rev-parse HEAD) || exit 1
[ "$OLD_HEAD" = "$EXPECTED_TIP" ] || exit 1
git merge-base --is-ancestor "$BASE" "$OLD_HEAD" || exit 1
git branch "$BACKUP" "$OLD_HEAD" || exit 1
git reset --soft "$BASE" || exit 1
git restore --staged -- . || exit 1
```

If `HEAD` no longer matches the inspected tip, stop and inspect again before replanning. The soft reset moves `HEAD` to the base but keeps the final files and stages the entire combined change. Unstaging leaves those files in the worktree so the executor can select the new commits. No hard reset or automatic force-push.

### 2. Rebuild the numbered commits

For each commit, provide:

- **Subject and purpose:** what changes and what the reviewer learns at this point.
- **Contents and dependencies:** exact paths, symbols, and hunk or content boundaries; prerequisite commits; and changes that remain deferred.
- **Reconstruction instructions:** how to prepare and stage this content, including shared tests and documentation, new files, deletions, and file modes.
- **Intermediate invariant:** what must work after this commit, with concrete repository commands to verify it.

Whole-file staging is appropriate only when the file serves one purpose in the series. Otherwise, stage selected patches. For a new file containing multiple commits' work, use intent-to-add so its patches can be selected.

Some changes share a hunk and cannot be patch-staged separately. In that case, specify the exact intermediate content rather than telling the executor to “split as needed”:

1. Construct that content from the current committed version and the saved final version.
2. Stage the intermediate version, preserving file modes and deletions.
3. Restore the saved final version to the **worktree only**. Leave the index—the snapshot that will become the next commit—unchanged.

The saved original tip remains authoritative. Deferred changes must stay available for later commits.

Require inspection of each staged diff and verification of that exact staged snapshot in an isolated checkout or worktree. Checking the main worktree is not enough: it still contains future changes that could hide a missing dependency. Include build or type checks and relevant behavior exercises. If a boundary fails, repair that commit before proceeding; do not add a fix-up commit. Name any unavailable verification prerequisite and say which check it prevents.

For example, a coherent sequence could be:

1. A working `QueueStore`, its SQL implementation, and their tests and documentation.
2. Migration of enqueue operations to the transactional store, with the migration's tests and documentation.
3. A worker, with its duplicate-delivery correction already included and its tests and documentation.
4. An independent timezone fix.

Shared files are divided according to those behaviors, not according to the old commit boundaries.

### 3. Prove preservation and review the story

Require all of these completion checks:

- The original tip and rebuilt tip have identical Git tree IDs. This proves the tracked contents and file modes match; the commit IDs are expected to differ.
- `git diff "$BACKUP" HEAD --` is empty.
- No reconstruction changes remain uncommitted, and the final repository checks pass.
- Reading the series in order reveals no unexplained dependencies or unrelated changes bundled together.

Restore isolated user work with its original staging state. Remove only reconstruction scratch files and checkouts; retain the backup until the user accepts the result.

If the history is published or shared, require explicit coordination before any push. A reconstruction plan does not authorize force-pushing.

## Before handing over the plan

Check it as if it were the first thing the reader had seen:

- Can they tell what is being rebuilt, what stays untouched, and why this order helps the reviewer?
- Can they execute each step without guessing which hunks belong where?
- Are unresolved decisions and unavailable checks visible next to the steps they affect?
- Is the next action clear: resolve a named blocker, or execute the plan?

## Quick reference / common mistakes

| Temptation or red flag | Required action |
|---|---|
| Reset now, back up afterward | Create the original-tip ref first. |
| Guess the base from upstream | Ask when the intended range is ambiguous. |
| Commit everything staged by the soft reset | Unstage first; select coherent slices. |
| One file or old commit per new commit | Group by reviewer-visible purpose. |
| Tests last; callers before contracts | Carry tests and documentation with behavior; order dependencies. |
| The final worktree passes, so the series must work | Verify each isolated intermediate snapshot. |
| A simpler split requires changing the final code | Preserve the original final tree exactly. |
