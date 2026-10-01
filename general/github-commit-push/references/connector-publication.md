# Publication through a connected GitHub app

Use live tool schemas rather than assuming names, payloads, or return shapes. Discover the app's reads and writes, and confirm access to the intended repository. Read access does not prove write access. A local folder needs no Git initialization to supply reviewed source bytes.

## Publish a partial change set

1. Read the target branch's current ref, commit, and tree. Pin the parent SHA and base tree SHA from that read. Compare selected paths with the current remote contents; reconcile overlapping changes instead of overwriting newer collaborator work.
2. Create blobs from the exact approved bytes. Preserve executable bits, symlinks, and other supported modes. Compute or read back blob hashes when needed; do not silently normalize binary data or line endings.
3. Create a tree on the verified remote base tree, applying only manifest entries. Represent deletions explicitly using the API's supported form. For a partial publication, never omit the base tree or set it to null: a tree built from scratch can remove unrelated paths.
4. Read the candidate tree and compare it with the base. Verify the complete changed-path set, expected hashes and modes, and preserved unrelated entries. An API's truncated recursive tree is incomplete evidence; obtain missing subtrees or another complete comparison before advancing the branch.
5. Create one commit with the verified parent and candidate tree. Re-read the branch before updating its ref, then update with force disabled. If it advanced, compare overlapping edits and rebuild on the new parent/base. Limit race retries to two; persistent races need a fresh scope assessment, not a force update.
6. Read back the ref and published commit/tree. If the branch subsequently advanced, verify the intended commit is an ancestor and its changes remain present. Report the actual published SHA.

A rejected or uncertain write requires reading current remote state before any retry. Reuse successful objects where appropriate; check whether the branch already contains the intended result. Do not switch tools, encodings, or destinations to bypass an approval rejection. Report permission failures plainly and retain the prepared change set.

## When only Contents writes are available

Read each existing file's current blob SHA and pass it to update/delete operations; use create only for a missing path. Set the intended branch explicitly. Write sequentially and verify each result. This creates multiple commits and can leave a partial publication if a later operation fails. Report every completed commit and the remaining paths; do not claim atomic completion. Preserve unrelated files and reconcile a changed path before retrying.

## Trace checks accurately

Read the actual endpoint's filters and pagination. Some commit-workflow wrappers expose only PR-triggered runs or the first page. For push CI, use a capable workflow-runs endpoint filtered by the published head SHA, and inspect relevant conclusions and skipped jobs. Fetch required pages where possible. If lookup coverage is incomplete or no matching run is found, report CI as unverified or pending, according to the evidence.

Connector publication may leave the local checkout dirty or behind the remote. Record that state and reconcile it separately while preserving local edits; never reset the checkout merely to make it match.
