---
name: sheesh-cloud-feature-work
description: Build or change a feature in a Next.js App Router site backed by Payload CMS. Use for compatible website feature work; apply sheesh.cloud-specific rules only in that checkout.
---

# Sheesh Cloud Feature Work

Deliver a well-scoped feature with verified behavior and a rendered local preview. Treat the active repository as authoritative; this skill supplies the workflow, not a substitute product brief.

## Establish the site context

1. Find the repository root and read its contributor guidance before editing.
2. Confirm that the project uses Next.js App Router and Payload CMS. If either dependency is absent, explain that this skill's implementation guidance does not apply and continue with the repository's own workflow.
3. Identify whether the request affects frontend UI, Payload schema/content, or both. Read the relevant local Next.js documentation before writing Next code. For Payload work, use any available Payload skill and repository access rules.
4. For a requested data feature, inspect the existing content and media model before adding fields. State whether the existing model is sufficient or a schema migration, generated types, and content backfill are required.
5. Preserve unrelated working-tree changes. Make only edits that serve the requested feature.

## Sheesh.cloud branch

When the repository is `/home/deck/Documents/AI/sheesh-cloud` or identifies itself as the Sheeshkidayyy Website:

- Read `AGENTS.md`, `docs/PRODUCT_REQUIREMENTS.md`, and `docs/DESIGN_SYSTEM.md` before frontend work.
- Keep the site a personal knowledge platform, not a generic developer dashboard. Use its warm editorial design language; reserve cyan for CyberPatriot and security work.
- Keep public Payload reads permission-aware. For schema changes, use generated types/import maps and migrations or backfills appropriate to the environment.
- Before any GitHub push, Docker build, Compose command, or deployment action, show a working local preview and wait for explicit approval. Treat `AGENTS.md` as the release gate.
- For UGREEN NAS diagnosis or maintenance, read `docs/UGREEN-DEPLOYMENT.md` and the repository guidance. Never treat a source change or migration as proof that the LAN site is live.

## Implement and verify

1. State the intended behavior and affected routes or content before editing.
2. Make the smallest coherent implementation. Reuse shared components and selectors when the behavior belongs in more than one place.
3. Run focused checks appropriate to the risk: formatting/diff checks, type checking, linting, targeted tests, and browser verification for visible work.
4. Report what passed, what was visually verified, and meaningful gaps. Do not claim a release, deployment, or migration completed without direct evidence.

## Completion

Finish when the requested feature is implemented and verified in proportion to its risk. Stop at any preview or approval gate before release-side actions.
