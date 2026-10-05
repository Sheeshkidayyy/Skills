---
name: nas-release-and-rollback
description: Use when asked to prepare, perform, roll back, or verify an authorized website release on a Docker Compose-hosted NAS. Covers release provenance, backups, database and media migrations, proxy routing, and post-deploy evidence; use payload-compose-diagnosis for diagnosis without a requested release.
---

# NAS Release and Rollback

Manage an explicitly authorized release on a NAS while preserving application data and establishing what is actually running.

## Scope and authorization

- Use this workflow only when the user asks to prepare, perform, roll back, or verify a deployment. A GitHub push, merge, or successful CI run does not authorize a NAS deployment.
- Follow the active repository's deployment, preview, and approval instructions. Treat them as authoritative for hostnames, commands, services, migration steps, and health routes.
- For diagnosis without a requested state change, use the relevant diagnosis workflow. Do not start or alter Compose services as a diagnostic shortcut.
- Do not change DNS, router forwarding, firewall rules, or public exposure unless the user specifically requests that work.
- Keep environment files, credentials, private keys, session tokens, and secret-bearing command output out of chat, logs, commits, and reports. Do not print or copy `.env.production` contents.

## Establish the release target

1. Confirm the repository, checkout, remote, target branch or immutable commit, and exact files or image intended for release. Check the working tree and preserve unrelated changes. Do not switch branches, pull over local work, or sync a dirty checkout without a safe plan.
2. Confirm the intended NAS, application, service names, Compose file, environment-file path, persistent volumes, proxy route, and requested LAN or public URL. Read the current deployment guide rather than relying on remembered host settings.
3. Identify the currently deployed application version and image where evidence allows. Record the previous known-good version so the rollback target is concrete.
4. Inspect available disk space and the current state of the relevant containers, database, and proxy using commands that do not expose environment values or full secret-bearing logs.

If the target host, release ref, Compose configuration, or state-changing authorization is unclear, stop before making changes and report what needs to be resolved.

## Prepare recovery before changing services

1. Determine which persistent data the release can affect. For a Payload site this commonly includes PostgreSQL data and uploaded media; inspect the actual Compose volumes and repository guidance.
2. Make a recoverable backup of each affected data store using the documented procedure. Confirm the backup completed, identify its location and timestamp, and confirm the documented restore procedure. Keep backups in an approved private location; do not upload them to GitHub or attach them to chat.
3. Review migration files and the documented migration command for the exact release. Establish whether the application can return to the previous version after the migration. Do not assume a database migration is reversible.
4. Write down the rollback sequence before deploying. If rollback would replace database or media data written after deployment, explain the data-loss impact and obtain explicit approval before restoring over live data.
5. Check that the NAS has enough space for the new image and required backups. Check actual host port ownership and proxy mappings before changing routes; do not stop an operating-system service to free a port unless that change was specifically requested.

Do not proceed when a required backup is missing or unverified, the migration path is unclear, storage is insufficient, or the rollback target cannot be identified.

## Apply the release

1. Reconfirm the selected source commit or immutable image tag immediately before the state-changing step. Keep source publication, CI, image build, and NAS deployment as separate actions with separate authorization.
2. Use the deployment command and migration sequence documented by the repository. Do not invent Compose files, service names, environment paths, or migration steps.
3. On UGREEN NAS hosts, invoke Compose with `sudo docker compose` and keep the production environment file private. Recheck current UGOS port mappings and proxy configuration; do not assume ports or host services are unchanged.
4. Apply only the changes required for this release. Never use `docker compose down -v`, remove a persistent volume, run development schema push, or restore a database as a speculative fix.
5. Capture sanitized command results and the exact image or commit deployed. If a build, migration, or service update fails, stop and inspect the failure before retrying. Do not repeat a failed migration blindly or chain additional state changes to hide an error.

## Verify the release

Check each layer independently:

- **Source:** the selected branch or commit matches the requested release.
- **Image:** the running image corresponds to that source ref, using a label, digest, deployment record, or other available provenance.
- **Services:** expected containers remain running without a new restart loop; inspect only relevant, sanitized logs.
- **Database and media:** the application connects to the intended persistent stores, migrations are recorded as expected, and an appropriate data-backed route works.
- **Proxy and routes:** test the requested direct backend and LAN or public route as appropriate. Confirm the request reaches this Compose stack, not another host service.
- **Application:** check the repository's documented health endpoint and the affected user-facing route. A container marked healthy or a static page returning HTTP 200 does not prove CMS data is available.

If a check fails, follow the preplanned rollback path. Roll back the application image to the identified previous version only when it is compatible with the current schema. Do not restore or overwrite live database or media data without explicit approval. Verify the restored service and data-backed routes after rollback.

## Report the outcome

State the target host and application, deployed or restored commit/image, backup and migration evidence, services and routes checked, and any remaining gap. Report source publication, CI, image build, NAS runtime, database health, and browser-visible behavior separately. Do not claim a live release from source or build evidence alone.
