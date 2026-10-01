---
name: github-commit-push
description: Use when preparing a Git commit, writing a commit message, or pushing selected source changes to GitHub, including publication from a plain folder through a connected GitHub app. A request to write or review changes alone does not authorize pushing them.
---

# GitHub Commit and Push

Publish the intended change set to the intended repository and branch, preserving unrelated work. Automatic skill selection supplies a workflow; authorization comes from the user and the active repository's rules.

## Establish scope and current state

Read contributor and release instructions and applicable user preferences. Determine whether the folder is a Git checkout or a collection of source files. Confirm the repository, target branch, and authorization from the conversation and live remote state; clarify only material ambiguity. Inspect the remote default/release branch and target branch when they differ. Apply the repository's working-branch convention.

For a checkout, inspect the working tree, existing index, upstream, and outgoing commits. Local refs may be stale. Refresh the remote view without overwriting local changes. Every outgoing commit must be within the authorized scope; selecting files in a new commit does not exclude older unpublished commits.

Build a manifest of intended additions, edits, deletions, and file modes. Preserve unrelated staged and unstaged changes, collaborator files, and private material. Inspect selected content for secrets and accidental artifacts without printing sensitive values. If an authorized file contains unrelated edits, select only the intended hunks.

## Prepare a reviewable commit

Run repository-required checks and verification appropriate to the changed behavior. Inspect complete output; unhandled failures or checks that never ran are coverage gaps. For visible website changes, render and inspect the affected local routes and follow existing preview requirements.

Finish the diff, manifest, checks, and any required preview before seeking release approval. Honor approval already given for this scope; ask only for a genuinely outstanding gate. A changed manifest or target needs renewed scope assessment.

Write a short commit subject naming the concrete change, following repository conventions. Add a body only when the reason, tradeoff, or validation needs it. Describe the final diff, not the conversation. Keep credentials and private infrastructure details out of messages.

## Commit and publish

In a checkout, stage explicit paths or hunks and review the staged diff and whitespace checks. Isolate the commit with a separate index or worktree when the existing index contains unrelated work; preserve that work. Verify the candidate commit and complete outgoing range. Push explicitly to the verified remote and target ref using a normal fast-forward update.

When earlier unpublished commits fall outside scope, build the selected patch on the current remote target in an isolated checkout. A separate index alone will not exclude those commits.

For a plain folder, unavailable Git authentication, or a repository that requires the connected GitHub app, read [references/connector-publication.md](references/connector-publication.md) before remote writes. Follow existing app preferences. Never use a force push, history rewrite, or destructive cleanup as a routine synchronization fix.

## Verify and report

Read back the remote ref, commit, and changed tree; compare approved paths, bytes or blob hashes, and modes. Establish that unrelated remote paths survived. Check CI for the resulting commit SHA, including pending, failed, and skipped jobs; an empty or incomplete lookup is not success.

Report the repository/branch, linked commit, published scope, verification, and remaining gaps. Distinguish source publication, CI, deployment, and live service health. Creating a commit locally is not pushing it; pushing it does not establish deployment. Create, merge, or deploy only within the requested scope. Attach a created PR when the environment provides an artifact tool.
