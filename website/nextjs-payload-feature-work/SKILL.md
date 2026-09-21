---
name: nextjs-payload-feature-work
description: Build or change a feature in a Next.js App Router site backed by Payload CMS. Use for compatible website feature work; do not use for sites without both technologies.
---

# Next.js and Payload Feature Work

Deliver a well-scoped feature with verified behavior and a rendered local preview. Treat the active repository as authoritative; this skill supplies a workflow, not a substitute product brief.

## Establish the site context

1. Find the repository root and read its contributor guidance before editing.
2. Confirm that the project uses Next.js App Router and Payload CMS. If either dependency is absent, explain that this skill's implementation guidance does not apply and continue with the repository's own workflow.
3. Identify whether the request affects frontend UI, Payload schema/content, or both. Read the relevant local framework documentation and obey repository access rules.
4. For a requested data feature, inspect the existing content and media model before adding fields. State whether the existing model is sufficient or a schema migration, generated types, and content backfill are required.
5. Preserve unrelated working-tree changes. Make only edits that serve the requested feature.

## Implement and verify

1. State the intended behavior and affected routes or content before editing.
2. Make the smallest coherent implementation. Reuse shared components and selectors when the behavior belongs in more than one place.
3. Run focused checks appropriate to the risk: formatting or diff checks, type checking, linting, targeted tests, and browser verification for visible work.
4. Start or reuse a local preview for visible changes and inspect the affected routes. Honor repository-defined preview and release gates.
5. Report what passed, what was visually verified, and meaningful gaps. Do not claim a release, deployment, or migration completed without direct evidence.

## Schema and release boundaries

- Keep public Payload reads permission-aware and preserve the project's access-control model.
- Generate types and import maps, and use the project's migration or backfill process when its schema changes.
- Treat a successful build, migration, source update, or container restart as separate from proof that a live endpoint works.
- Before a production deployment or other external release action, obtain the user's approval when the repository does not already define an approval gate.

## Completion

Finish when the requested feature is implemented and verified in proportion to its risk. Stop at any preview or approval gate before release-side actions.
