---
name: documenter
description: Use when documenting project work or chat handoffs, resuming substantial work that depends on past conversations, or getting stuck on a problem that may have been solved before.
---

# Documenter

Give the next AI enough evidence to continue without rediscovering decisions or repeating failed attempts. Preserve detailed relevant history in retrievable records; load only the parts needed for the current task. This skill uses accessible history and saved notes, not guaranteed access to every conversation or automatic background recording.

## Recover context

1. Identify the project, current user objective, and applicable instructions. Read the existing documentation index and relevant notes. When context is missing, a request refers to past work, or an error recurs, use [history-recovery.md](references/history-recovery.md).
2. Build a working account of user requirements and corrections, decisions and reasons, implementations, failed attempts, verified results, unfinished work, and next steps. Attach source pointers and dates; distinguish user instructions, assistant proposals, observations, and inference.
3. Resolve conflicts using instruction priority and the latest applicable user direction. Retain superseded decisions as history. Verify changeable facts before acting: branch, files, versions, processes, remote state, and service health. An old approval is evidence about its original scope; assess whether it authorizes today's action.

Recovery is complete when the current objective, controlling constraints, relevant prior attempts, verification limits, and next action are known, or missing information is explicitly identified. Continue useful authorized work when history is incomplete.

## Save a durable handoff

When the user requests documentation or has authorized ongoing journaling, use [record-templates.md](references/record-templates.md). Prefer their existing private documentation location. Otherwise use a separate private directory under `${CODEX_HOME:-$HOME/.codex}/documenter/<project-key>/`, keyed by a readable project name plus a stable identifier for its canonical path or repository.

Keep the index concise and put detailed session records and reusable solutions behind links. Record material decisions when settled, recovery discoveries after verification, and unfinished state before a handoff or context compaction when possible. Append new evidence to the appropriate record; mark corrections explicitly and retain earlier outcomes. Avoid duplicate records and empty template sections.

Automatic selection helps recover context; it does not authorize unrelated writes. Honor environment rules for memory updates. Keep operational records outside the reusable skill and public repositories unless the user chose that destination. Redact secrets and sensitive payloads, preserving safe commands and credential variable names. If the private destination is unavailable, deliver the handoff in chat and identify the unsaved state.

## When stuck

Use [getting-unstuck.md](references/getting-unstuck.md) before repeating a failed approach. Find matching history, compare the old environment with today's, run the smallest relevant authorized diagnostic, and record the actual outcome. Treat an old fix as a candidate until reproduced and verified here.

## Completion

Check that links resolve, claims have evidence or uncertainty labels, the current index agrees with the latest records, and the next action is executable. Report what was recovered, what was saved and where, and remaining gaps. Distinguish documentation validation from validation of the project itself.
