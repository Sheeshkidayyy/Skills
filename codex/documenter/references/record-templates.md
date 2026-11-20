# Durable records

Use existing project documentation conventions when available. Otherwise create only records with useful content:

```text
<private-project-directory>/
  INDEX.md
  sessions/<date>-<topic>-<unique-id>.md
  solutions/<problem-key>.md
```

Keep each project in a distinct directory, including when repository names coincide. Include a source chat ID when available, or another unique suffix, to avoid same-day collisions. Store in a user-chosen private writable location; public documentation requires deliberate selection and redaction. Do not place private chat excerpts inside the installed skill.

## INDEX.md: start here

Target one quick read; link out when detail grows. Maintain:

- Project identity and canonical path/repository; documentation location; last update and timezone.
- Current objective and completion criteria in the user's terms.
- Controlling user requirements, corrections, exclusions, and authorization scope, with source/date.
- Architecture and dependencies that explain the task; paths and entrypoints relevant to the next action.
- Current state: completed, in progress, blocked, and unverified; links to evidence and newest handoff.
- Decisions with reasons and superseded alternatives; detailed discussion lives in session records.
- Troubleshooting index: symptom/search terms, applicable environment, outcome, and solution-record link.
- Next actions, in order, including working directory, prerequisites, expected result, and validation.
- History coverage: relevant chats inspected, partial/unavailable ranges, and unresolved contradictions.

Separate current observations from historical context. Attach dates and evidence to mutable facts; a timestamp alone is not verification. Update the index after its linked records are saved, preserving concurrent changes.

## Session record: preserve the reasoning and execution

Use this shape, omitting sections with no content. Expand materially different attempts instead of compressing them into a vague summary.

```markdown
# <Task or session topic>
Date/time and timezone:
Project and working directory:
Source chats/turns/files:
Coverage and missing sources:

## User objective and constraints
Requested outcome, exact wording where precision matters, later corrections,
accepted decisions, exclusions, and actual authorization.

## Starting state
Relevant files, branch/commit when observed, environment and versions,
dependencies, prior unfinished work, and known verification limits.

## Decisions
Choice, reason, alternatives considered, why rejected or superseded,
source/date, and whether user-directed or an assistant proposal.

## Attempts and results
For each material attempt: hypothesis, prerequisites, working directory,
sanitized command or code/config change, observed result, evidence pointer,
why it helped or failed, and whether reverted or still present.

## Verification
Check and expected result, actual output, relevant artifact, environment/date,
and remaining coverage gaps. Separate static checks, rendered behavior,
source publication, CI, deployment, and live service health where relevant.

## Handoff
Current file/process state, changes left in place, completed work,
unresolved questions, access/approval dependencies, and next exact action
with its expected observation and validation.
```

Reference large outputs at their original location, or save a necessary sanitized excerpt in the private record. Retain exact error signatures, command arguments, paths, and sequencing needed to reproduce the result. Replace secret values with meaningful placeholders; preserve credential variable names and where they are supplied without their contents.

## Solution record: retrieve by symptom

```markdown
# <Recognizable symptom>
Search terms:
Status: <verified-here | historical-only | failed | partial | proposed>
Source/date and last verification:

## Applies when
Project/component, versions, platform, prerequisites, and trigger.

## Evidence and cause
Observed failure, causal evidence, uncertainties, and original source links.

## Procedure
Working directory, ordered sanitized commands/changes, expected observations,
and reversal steps where relevant and known.

## Validation
Original reproduction, before/after observations, checks and outputs,
and untested assumptions.

## Failed approaches and boundaries
What failed or was rejected, why, and incompatible environments or constraints.

## Current outcome and next action
Whether reproduced here, changes left in place, remaining gaps, and next check.
```

When a solution is corrected, append the new evidence and mark which procedure supersedes the old one. A proposed command is not an executed attempt. An interrupted tool call is not successful validation. Keep timestamps, sources, and status aligned so future agents can distinguish experience from speculation.
