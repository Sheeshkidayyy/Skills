# Find and validate a previous solution

## Build a useful problem signature

Record the failing action, expected and observed behavior, sanitized exact error, reproducible conditions, affected files/services, versions, and working directory. List attempts already made and their outcomes. Capture one discriminating observation when the symptom is vague.

Search the project index, solution records, and relevant chat history using the signature. Expand from the exact error to the affected component or failure mechanism if needed. Include failed or rejected approaches; their constraints can be as useful as a successful fix.

## Compare before replaying

For each candidate, compare project identity, versions, platform, configuration, data state, permissions, and user constraints. Separate the original trigger, diagnosed cause, attempted fix, and verification. A matching error message alone does not establish a matching cause.

An old record saying a port conflict was fixed by reusing a development server suggests inspecting today's listener and confirming which project it serves. It does not prove the same server still exists. A timeout increase that helped another application suggests checking which layer times out and whether a legitimate operation exceeds that limit. It does not establish today's cause.

Choose the smallest authorized diagnostic that distinguishes the candidate explanation from alternatives. Execute a compatible fix only within the current task's scope. Validate against the original failing case and required surrounding behavior. Record rollback or reversal steps when a change warrants them; do not invent an untested rollback guarantee.

## Break repeated failures

After an identical failure repeats without new evidence, change the diagnostic approach instead of repeating the same command. Do not turn a permissions error into assumed authorization or a retry into a more destructive action. Keep the comparison and failed attempts visible in the handoff.

If no applicable fix exists, proceed with evidence-led diagnosis: reproduce, isolate the failing layer, form a testable hypothesis, perform a focused check, and assess its result. If progress requires unavailable access, authorization, or a user decision, finish independent work and request the specific missing input. A documentation request alone authorizes recording recommended actions, not running operational fixes.

## Save what the next AI needs

Write the symptom and cause separately, original versus current environment, source pointers, exact sanitized commands or changes, why each attempt worked or failed, and before/after verification. Label outcomes `verified-here`, `historical-only`, `failed`, `partial`, or `proposed`. For an unresolved problem, preserve the last known state, useful evidence, eliminated hypotheses, and the next discriminating check. Promote a procedure as reusable only to the extent its evidence supports.
