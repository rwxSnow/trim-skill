---
name: trim-skill
description: Trim and compress skill documents to reduce token count while preserving capability. Use when trimming, compressing, or optimizing an existing skill.
---

# Trim Skill

<HARD-GATE>
Do not delete or rewrite any content that affects the skill's triggerability, decision boundaries, output contract, or safety constraints.
If you are not confident that a passage is movable, keep the original text. "Shorter" must never come at the cost of "might be wrong."
</HARD-GATE>

## Goal

Delete tokens, not behavior.

Minimize token count without harming the skill's capabilities: triggerability, decision boundaries, output contract, and safety constraints.

## Core Judgments

Apply **Movable Judgment** and then **Immovable Judgment**. **Immovable Judgment** is a hard exclusion: once something is judged immovable, arbitration cannot override that decision. **Conflict Priority** is used only to arbitrate tradeoffs among movable items.

### Movable Judgment

- Content that adds no new constraints, such as stylistic phrasing, background, or noise, may be treated as a candidate for reduction.
- If a passage only explains why and does not add behavioral constraints, it may be listed as a candidate, but it must still pass final review under **Immovable Judgment**.

### Immovable Judgment

**Generative test**: If deleting, externalizing, or rewriting a passage would create at least one reasonable input scenario in which the agent's behavior diverges from the original in an undesired, observable way, that passage is immovable.

Common high-risk categories include the following, but the list is not exhaustive. Absence from this list does not imply movability:

- Trigger words, discoverability, and trigger boundaries
- Decision boundaries, exceptions, red flags, conflict priorities, and tradeoff rules
- Output contracts, including fields, structure, format, and domain-critical term definitions
- Safety constraints, refusal boundaries, failure handling, fallback paths, and the qualifiers, conditionals, and semantic anchors that prevent semantic drift

**Intentional redundancy rule**: Redundancy used to disambiguate, emphasize priority, or stabilize formatting counts as a constraint and is immovable.

### Conflict Priority

Correct > Triggerable > Semantically Clear > Discoverable > Shorter

## Strategies

- **Externalize**: Externalize only substantial supporting material that does not affect the main path, such as long examples, large few-shot blocks, variant details, large parameter tables, template boilerplate, or extended explanations. If the target skill does not support external files, skip externalization.
- **Delete**: Remove synonymous repetition, repeated statements of the same control rule, non-instructional narrative, generic model advice, modifiers with no decision value, and full parameter tables that can be replaced by `--help` or reference files.
- **Merge/Compress**: Convert prose into headings plus lists, if-then rules, decision rules, or imperative rules. Merge multiple synonymous rules, similar example groups, or explanatory passages into a stronger, shorter expression that still preserves constraints.

## Process

In this document, `Change List` is also referred to as `CL`.

### 1. Read and Understand

- Read the entire target skill and understand its core function, structure, and constraints.

### 2. Scan and Adjudicate

- Scan candidate items through the lens of `Externalize / Delete / Merge/Compress`
- Filter with **Immovable Judgment** and arbitrate using **Conflict Priority**
- Mark each item with `Strategy / Granularity / Risk / Rationale / Dependencies`
- Produce the `CL`, with all initial states set to `Pending`

### 3. Benefit Guard

- Compute the **line ratio**, total proposed modified lines divided by total original lines, and the **granularity distribution**
- If the `CL` is empty, or the line ratio is `< 5%` with no paragraph-level items, judge the expected benefit insufficient, recommend stopping, and continue only if explicitly requested

### 4. Present and Approve

- Show the user the `CL` and the files to be modified
- Do not perform any operation before user approval
- Each item may only be set to `Approved / Rejected / Revision Requested`
- Iterate modifications until the user is satisfied
- If all items are rejected, stop

### 5. Implement

- Back up all files to be modified first to `original-project-name_backup/original-file-name_timestamp(to-the-second)`
- If backup fails, exit immediately
- Implement only `Approved` items

### 6. Verify

**Per-item check: semantic quality**. For each `Implemented` item:

- Format is valid and the skill structure is intact
- Trigger conditions are clear and core constraints remain complete
- No ambiguity, unintended negative connotation shift, or semantic drift
- Tone has not been weakened

**Global check: deep structural integrity**. Evaluate the skill as a whole:

- **Self-sufficiency**: After compression, does the skill still drive behavior via its own rules rather than relying on the model's default tendencies to fill in deleted content?
- **Robustness**: Are the constraints still consistently enforced rather than only occasionally remembered? Are boundary cases still controlled?
- **Rule support / Explainability**: Is output quality primarily supported by rules rather than examples? Can you clearly explain which rules make the skill work reliably? If it mainly depends on examples or model priors, the rule layer is too thin.

Rollback failed items from backup. If their dependencies are invalidated, cascade rollback accordingly.

### 7. Delivery Report

- Output according to the delivery report contract
- After delivery, the process ends. Do not apply the same batch of `CL` a second time

## Change List Contract

- The `CL` records only items adjudicated as movable. Immovable items must not enter the `CL`.
- Required columns: `ID / Status / File:Line Range / Strategy / Granularity / Risk / Description / Dependencies / Rationale`
- `Strategy` values: `Delete / Externalize / Merge / Compress`. During scanning, `Merge/Compress` may be treated as one strategy family.
- `Granularity` values: `Paragraph (>=3 lines) / Sentence (1-2 sentences) / Term (word, modifier, or punctuation)`
- `Risk` values: `High / Medium / Low`. High means near the immovable boundary. Medium means it may produce differences under certain inputs. Low means explicit redundancy or noise.
- Status enum: `Pending / Approved / Rejected / Revision Requested / Implemented / Rolled Back`
- Status transitions: `init -> Pending`; `Pending -> Approved / Rejected / Revision Requested`; `Revision Requested -> Approved / Rejected`; `Approved -> Implemented`; `Implemented -> Rolled Back`
- Terminal states are only: `Rejected / Implemented / Rolled Back`

## Delivery Report Contract

- Output in sections aligned with the `CL`
- Required sections: `Overview / Change List (Final Status) / Verification Results / Follow-up Suggestions`
- `Overview` must include: `Original File / Backup Path / Token Count Before / Token Count After / Net Savings`
- `Change List (Final Status)` must contain the full `CL`, and all item statuses must be terminal
- `Verification Results` must list at least: `ID / Surface Quality / Deep Integrity / Conclusion`
- `Follow-up Suggestions` should list content that could still be trimmed but was not included in this round. If none, write `None`

## Key Principles

- Delete tokens, not behavior. When unsure, keep it.
- Do not skip the process just because the target skill looks simple. Prefer direct deletion. Rewrite only when net benefit is clear and constraints are not weakened.
- Implement strictly by `CL`. Do nothing before user approval. Stop if all items are rejected. Exit if backup fails.
- On conflict: `Correct > Triggerable > Semantically Clear > Discoverable > Shorter`
