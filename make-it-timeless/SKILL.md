---
name: make-it-timeless
description: Remove unexplained conversational and drafting residue from documents and responses while preserving the context, history, dates, limitations, and uncertainty their readers need. Use when writing or revising material that should stand on its own beyond the conversation that produced it.
hide: true
---

Uses RFC 2119: BCP 14: Key words for use in RFCs to Indicate Requirement Levels. The capitalized terms MUST, MUST NOT, SHOULD, and SHOULD NOT have the meanings defined in BCP 14 (RFC 2119, as updated by RFC 8174).

# Make It Timeless

Write for the intended reader and purpose, not for the conversation or process that produced the text. Make the result understandable on its own, without unexplained references to earlier exchanges, abandoned work, or assumptions the reader cannot see. This applies to new writing as well as revisions.

## Establish what belongs

Identify the document's purpose, intended audience, and the facts, evidence, constraints, and actions needed for that purpose. Lead with the subject or answer, not with the drafting process. Assume the reader has the normal background of the intended audience, but not access to the originating conversation. The result MUST include the context needed to understand material claims, references, limitations, and instructions; use an identified external source where appropriate rather than reproducing everything.

Inspect each piece of conversational or procedural history for its function in the finished work:

- You SHOULD remove unexplained exclusions, prior plans, failed attempts, changes of approach, inspection notices, generic warnings, and references such as “as discussed” when they do not help the reader understand, decide, or act.
- Keep an exclusion or prior event only when it serves the reader's purpose; state what it concerns and why it matters in the document's own context. A true statement about an unrelated topic is still residue, even if rewritten as a clean scope statement. A backup inventory can say it is not a full system image when that affects recovery; it need not mention unrelated installation plans.
- Replace vague temporal or conversational references with the actual state, condition, or relationship when known. For example, replace “enabled for now” with the specific condition that ends the need for it. You MUST NOT invent that condition if it is not known.
- Remove descriptions of the author's investigation or drafting process unless they establish evidence, provenance, limitations, or another fact the reader needs. A bare statement that something was inspected, without a meaningful scope or finding, is not a substitute for useful evidence; omit it unless the inspection itself matters to the reader or record.
- You SHOULD remove unmotivated reminders to verify or recheck information, even when they name a particular value or date. Do not recast such a reminder as evidence that the information is unreliable. You MUST keep verification requirements and status when supported by a specific risk, obligation, documented limitation, or the reader's task; explain the reason when available.

You MUST NOT turn useful history into an orphaned conclusion. An incident report, decision record, audit, or research memo may require a timeline, rejected alternatives and their reasons, a failed test, or a dated observation. Keep the history that explains the outcome or serves the document's purpose, including the relevant causal or decision-making connections.

## Preserve meaning and evidence

You MUST honor the user's explicit requirements for content, wording, quotations, and format. You MUST keep required operational and safety steps even if the source does not explain their rationale. When a quotation must be retained verbatim, you MUST NOT rewrite it to remove conversational phrasing.

Timeless does not mean undated, absolute, or permanent. You MUST retain dates, times, locations, measurements, version information, attribution, and provenance when they establish what was observed, when it applied, or what can be relied on. Keep real deadlines and conditions, important exclusions and their consequences, unresolved questions, and the difference between a recommendation, a plan, and completed work.

You MUST preserve uncertainty and the strength of the evidence. You MUST NOT convert correlation into causation, an unverified claim into a fact, an observation into a lasting rule, an untested backup into a verified restore, or an intended action into a completed one. When material sources disagree, retain the disagreement and attribution rather than silently choosing an account or inventing a timeline. Keep dates, sources, and verification claims attached to the specific events they describe; proximity alone does not establish that an inspection date applies to a separate check.

You MUST NOT infer missing reasons, decisions, outcomes, dates, or commitments. In particular, do not assume that an embedded conversation or an undated source takes place on the current system date. Assert a deadline's status or convert a relative date only when the relevant reference time is established by the task or evidence. If essential context is unavailable or contradictory, preserve the uncertainty or state what is missing; ask for clarification when necessary rather than inventing an explanation.

Use the appropriate level of detail for the reader and task. Do not retain every conversational detail, but you MUST NOT silently discard material facts merely because they describe past events, negative scope, uncertainty, or a temporary condition. You SHOULD NOT add generic caveats to make the prose sound prudent; state specific limitations where they matter.

## Deliver

Silently check the result: can the intended reader understand each material reference, and have the relevant evidence, limitations, and actionable facts survived? Revise accordingly. Return the requested document or answer in the requested format and voice; do not add an explanation of the cleanup unless the user asks for one.
