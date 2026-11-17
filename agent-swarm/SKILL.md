---
name: agent-swarm
description: Use when coordinating multiple OMP agents that must agree on shared interfaces before parallel implementation in Git worktrees.
---

# Agent swarm coordination

Use the swarm to make shared design decisions explicit before parallel code changes. A collection of agents working independently is not coordination.

## Before dispatch

1. Pick one integration owner. Split work by independently owned responsibilities and files; identify shared interfaces and dependencies first.
2. Resolve dependencies and shared prerequisites before creating task branches. Give each implementation task one worktree, one branch, an exact base commit, and an absolute worktree path.
3. Put exact model and effort settings on every delegated task. Never assume that an agent inherits the controller's model or effort.
4. Write one concise interface contract: data ownership, lifecycle, coordinate units, message ordering, error behavior, and which files each task owns.

## Join and agree

- The coordinator joins the named swarm with `peer_join`, then verifies membership with `peer_list`.
- Each participant must join the same swarm and confirm the same contract before editing. Use `peer_send` with broker-issued participant UUIDs; agent names and job IDs are not peer IDs.
- Require an explicit `AGREED` from every implementation participant. A shared task prompt is useful context, but is not evidence that peers exchanged or accepted the contract.
- Keep messages concrete: state the interface and invariants, not a general request to “coordinate.”
- `peer_send` success means the broker accepted the message, not that its recipient read it. Check replies or the mailbox before claiming delivery.

## Membership failures

- Treat `not_joined` as a real membership failure. Do not claim that a task agent joined, and do not retry a non-retryable error indefinitely.
- A headless OMP subprocess may require `OMP_SWARM_ALLOW_JOIN=1` at launch. Set it only when the human has explicitly authorized additional headless participants; the flag is an authorization grant, not a workaround for an unapproved join.
- If task runtimes cannot join, tell the controller and wait for an explicit, controller-mediated contract confirmation. That relay can unblock the work, but it is not direct peer communication. If actual IRC delivery is required, use an authorized OMP subprocess relay, verify broker acceptance, and report whether delivery was confirmed.
- Record the exact failure and fallback in the task ledger. Never conceal the difference between an agent's direct agreement and a relayed message.

## Worktree discipline

- Give each agent only its own worktree and files. The task tool may start agents in the controller's directory; use absolute paths and an explicit shell `cwd`.
- Avoid shared manifests, lockfiles, generated files, and barrel modules during a parallel wave. Assign one owner or resolve those changes before dispatch.
- Preserve unrelated staged, unstaged, and untracked user work. Do not stage or overwrite it to make a worktree appear clean.
- Record each task's base, branch, worktree path, agent identity, and review state before context can be lost.

## Integration

- While agents work, do useful local coordination: prepare the next brief, review completed work, or update the ledger. Do not poll idle interfaces or wait while actionable work remains.
- Accept a result only from the task's named worktree and scope. Review the committed diff and its callers, then merge one task at a time through the integration owner.
- Run the task's covering checks after its implementation is complete and the combined suite once after a parallel wave merges. Keep test claims tied to observed output.
- After a contract change, update the shared brief and notify every downstream owner before they code against it.
- Close out by recording unresolved findings, preserving user changes, and removing only worktrees and branches that contain no unmerged work.
