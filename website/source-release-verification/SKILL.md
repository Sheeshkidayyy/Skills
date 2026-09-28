---
name: source-release-verification
description: Audit a website's GitHub source or prepare and verify an approved source release. Use for website CI failures, push or release requests, and questions about whether source, CI, and the deployed site match; ordinary feature implementation uses the site's feature workflow.
---

# Source Release Verification

Establish what is in the checkout, what is on the remote branch, and what actually runs. Keep a compact evidence record for each state.

## Establish the change set

Read the repository's contributor and release guidance. Confirm the checkout root, working-tree changes, remote repository, current branch, target branch, and their current commit IDs. Identify the exact intended files and preserve unrelated work. For a remote-only audit, inspect the remote source and checks within that scope.

For a CI failure, read the failing run's branch, head SHA, job log, and exact failing step before choosing a fix. Read the relevant diff and affected call sites. For source publication, reconcile a stale local checkout with the current remote parent without overwriting local changes. Inspect the final file manifest for secrets, generated files, and unintended additions.

## Verify the result

Run checks that exercise the changed behavior, then inspect their complete output. Treat an unhandled rejection or a test that never started as a failure even if another summary says “passed.” For visible changes, show and inspect a rendered local preview at the affected routes and viewports. Check interaction after transitions finish. If the repository requires a preview before every push, show one even for a code-only fix. For CMS or database routes, verify data-backed behavior separately from a static or fallback page; record an unavailable database as a coverage gap.

Follow the repository's preview, branch, and approval gates. Complete the reviewable change set and evidence before seeking a required release approval.

## Publish and trace

When source publication is authorized, advance only the intended branch from its verified parent. Read back the remote commit, branch ref, and changed-file tree. Correlate the resulting CI run's head SHA with that commit, including failed or skipped jobs. A green build establishes only what that build tested.

Report source publication, CI, container build, deployment workflow, and live route health as separate facts. For a live-release claim, establish the deployed SHA from the deployment run or running checkout and check the actual service and relevant data-backed endpoints. Keep a status inquiry read-only unless deployment was separately requested and authorized. End with the remote commit ID, checks and preview evidence, and remaining gaps.
