# trim-skill

An agent skill that trims and compresses other skill documents, reducing token count without losing capability.

## How It Works

```text
Read skill -> Scan & adjudicate -> Benefit guard -> Present CL -> Implement -> Verify -> Report
                 |                      |                 |                          |
          Movable / immovable     Abort if savings   User approves            Roll back
             judgment                < 5%            each item                failed items
```

Core concepts:

| Concept | Purpose |
|---------|---------|
| **Immovable Judgment** | If removing a passage could cause undesirable behavioral divergence in *any* reasonable input scenario, the passage stays. |
| **Conflict Priority** | `Correct > Triggerable > Semantically Clear > Discoverable > Shorter` |
| **Change List (CL)** | Every proposed edit is tracked with strategy, granularity, risk, rationale, and dependencies. Nothing is applied without user approval. |
| **Benefit Guard** | If total editable lines < 5 % of the document with no paragraph-level items, the skill advises the user to stop, proceeding only on explicit request. |
| **Backup-First** | All target files are backed up before any modification; backup failure aborts the entire run. |

## Key Principles

- **Delete tokens, not behavior**: the only goal.
- **When in doubt, keep it**: a few extra tokens are always cheaper than a broken skill.
- **Correct > Triggerable > Semantically Clear > Discoverable > Shorter**: the tiebreaker chain.
- **CL is the single source of truth**: nothing outside the CL gets touched.
- **User approval is a hard gate**: no edits before sign-off; all vetoed means stop.

## Install

Simply open your Claude Code, Codex, or other AI tools, and enter this single command:

`Install this skill: https://github.com/rwxSnow/trim-skill`

## Usage

Simply ask your agent to trim a skill:

- *"Help me trim this skill to reduce tokens."*
- *"Compress my-skill/SKILL.md, it's too long."*
- *"Optimize this skill document for token efficiency."*

The agent will follow the full workflow: read, scan, present a Change List, wait for your approval on each item, back up files, implement approved changes, verify, and deliver a final report.

## Limitations of This Skill

- This skill is based on static text analysis and cannot guarantee full runtime equivalence. Reasoning about "reasonable input scenarios" is also limited by the agent's capabilities.
- For skills that depend heavily on implicit context, subtle tone, or cross-paragraph semantic dependencies, compression carries higher risk. If confidence is insufficient, keep the original and advise the user to review independently.