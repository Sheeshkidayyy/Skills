---
name: payload-compose-diagnosis
description: Diagnose an existing Payload CMS and PostgreSQL Docker Compose deployment when the web service restarts, tables appear missing, admin or API routes fail, or a reverse proxy LAN route is wrong. Use for evidence-first troubleshooting before a repair.
---

# Payload Compose Diagnosis

Locate the failing layer and preserve the deployed data. A healthy database container, a successful build, and a page returning 200 each prove different things.

## Establish the target

1. Confirm the checkout, remote, branch, current commit, and intended deployed revision. Read the repository's contributor and deployment guidance before runtime commands. Preserve unrelated local changes.
2. Identify the actual Compose file, environment file, service names, data volumes, and reverse proxy. Check that the environment file exists without printing its contents or exposing secrets from logs.
3. Honor the user's existing approval and preview gates before any Compose, build, source sync, or deployment command. Read the current repository's deployment guide for host-specific commands and routing.

## Collect the smallest useful evidence

Collect available source, deployment-record, and read-only HTTP evidence immediately. After any approval gate that applies to Compose commands, inspect service state and recent logs for the failing service, then test the affected route and a database-backed health route. When a proxy is involved, compare the backend response with the response through the proxy and inspect the mapped host port. Record the exact error and which layer produced it; avoid dumping full environment or unfiltered logs.

Use the error to narrow the next check:

- **Restart or permission error:** stabilize the web service first, then retest failed routes before pursuing a separate schema cause. Compare the running image, Dockerfile ownership and read permissions, and mounts over the failing path. Establish an immutable link from running image to source commit through a build label, digest record, or deployment attestation; a current checkout alone does not prove image provenance.
- **Missing Payload table or failing admin/API:** identify the database actually selected by the app. Check whether application tables and `payload_migrations` exist there, then inspect candidate databases, persistent storage, and backups before assuming an empty deployment. PostgreSQL readiness only establishes that it accepts connections.
- **LAN or proxy failure:** inspect the proxy's route matcher, backend target, published port, and direct backend response. Confirm the tested address reaches this Compose stack rather than another host service.

## Repair and verify

Explain the evidence and proposed correction before a state-changing step. For schema or storage changes, confirm recoverable database and media backups and follow the repository's migration procedure. Do not use a fresh migration, delete a volume, restore over data, or use development schema push as a speculative production fix. Use a historical baseline only when the selected database and backups establish that there is no content to recover.

After a repair, verify that services stay up, the original error is absent from fresh logs, a database-backed health check succeeds, and the affected route works through the requested LAN or public path. Report source/CI status, running-image provenance, container status, database health, and browser behavior separately when any of them remain unverified.
