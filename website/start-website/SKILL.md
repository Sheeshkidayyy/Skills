---
name: start-website
description: Start an existing local website for LAN access, check its source against live GitHub develop, and report commit differences from main. Use when asked to start, open, restart, or refresh a website development preview.
---

# Start Website

Start the existing development site, open its current LAN URL, and report source freshness and runtime health. Default to port 3000 and the development branch `develop`; read project instructions for overrides. This workflow does not authorize source publication, main merges, production builds, Docker/Compose actions, or deployment.

## Establish source freshness

1. Locate the website checkout from the current workspace, configured projects, or the user's path. Read its `AGENTS.md` and inspect the package scripts, lockfile, Git remote, branch, full HEAD SHA, and `git status --short`. Preserve staged, unstaged, and untracked work.
2. Use the connected **GitHub plugin** to read live `develop` and `main` refs in the repository identified by the remote. Discover its tools when necessary. Pin both full SHAs for all comparisons; local `origin/*` refs are not proof of current GitHub state.
3. Obtain these comparisons, with the base on the left:

   | Comparison | Meaning of ahead / behind |
   | --- | --- |
   | `develop...LOCAL_HEAD` | Local-only commits / missing develop commits |
   | `main...develop` | Develop-only commits / main-only commits |
   | `main...LOCAL_HEAD` | Local-only commits / missing main commits |

   Use the plugin's compare tool and its `ahead_by`, `behind_by`, and `status`. When LOCAL_HEAD is unpublished and GitHub returns 404, use fetched, SHA-verified Git objects and `git rev-list --left-right --count BASE...HEAD` (left = behind; right = ahead). Fetch into a temporary repository if the working checkout must remain untouched.

   If Git transport fails, the plugin's paginated commits endpoint can supply each pinned branch's reachable commit SHAs. Compare those sets with `git rev-list HEAD`: local ahead = local minus remote; local behind = remote minus local. Retrieve every page and verify every listed parent is present before calling counts exact. If history is incomplete or access fails, report the count as unavailable, never zero.
4. Check file freshness separately. Equal commit/tree SHAs prove committed content equality, but a dirty working tree still needs inspection. If necessary, compare the local source files' Git blob hashes and modes with the plugin's recursive tree at the pinned develop SHA. Report missing, changed, and locally added source separately; exclude environment secrets, dependencies, caches, and private session notes. History can diverge while working files already contain published changes.
5. Bring a **clean, compatible checkout** to the pinned develop SHA with a fast-forward update. Select an existing develop branch only after checking its own history. If the local branch or files diverge, retain them: use a separate checkout pinned to develop when the request requires the latest published preview, or clearly label a requested current-work preview with its differences. Carry local LAN startup preferences into an isolated preview through process environment/CLI settings. Do not reset, clean, auto-stash, overwrite, or silently preview outdated files as current. A transport or dependency blocker should leave the preserved current-work preview available when possible, with the freshness limitation explicit.

## Start efficiently on the LAN

1. Inspect the listener on port 3000 and its process command/cwd. Reuse a healthy server only when it serves the intended checkout and is accessible at the LAN URL. For a loopback-only server from that checkout, gracefully stop that exact development process and restart it for LAN access. Leave unrelated processes alone and report a port conflict instead of silently choosing 3001.
2. Read the project's startup instructions and installed framework documentation. Reuse installed dependencies when compatible; install from the lockfile only when dependencies are missing or have changed. Avoid reinstalling packages, clearing caches, rebuilding production, or launching duplicate servers as routine startup steps.
3. Discover the current Wi-Fi/Ethernet IPv4 address from the default route and interface list. Prefer a project LAN launcher when present. Otherwise start the framework with the observed LAN interface address (or `0.0.0.0` when required) and explicit port 3000. The browser URL must be `http://<current-LAN-IP>:3000`; `0.0.0.0` is a bind address, not the browser destination. Detect the address each run; use a user-supplied LAN host for ambiguous interfaces.
4. For Next.js, ensure the LAN hostname is accepted by `allowedDevOrigins` using the project's existing development configuration. For Payload, ensure the development server URL/CORS includes the same LAN origin. Set process-only development values when supported, preserving saved secrets and production configuration. Keep any required database available using the existing authorized local setup; repository approval rules still apply to Compose or infrastructure actions.
5. Keep the server in a managed terminal session, save its session/process identity and logs, and wait for readiness. If the sandbox prevents network inspection or binding, use the permitted local-server execution context. State a remaining permission/runtime block accurately.

## Open, verify, and report

Request the homepage and the project's health endpoint through the **LAN address**, then open that same LAN URL with the available browser tools. Inspect the rendered page and browser errors; confirm development assets load without cross-origin 403s. Open an existing initialized admin page only when relevant; starting a preview does not require creating accounts or reseeding content.

Report:

- Clickable LAN URL, port, actual checkout/branch/SHA, and server session identity.
- Whether local source matches live develop; local ahead/behind counts and any uncommitted file differences.
- Develop ahead/behind main, and local ahead/behind main. Interpret “versions off” as commits unless the project defines releases differently. Distinguish a merge-only history difference from changed file contents.
- Rendered-page result and database/CMS health separately. A listener, homepage 200, static fallback, or successful GitHub check alone does not prove database health or access from another LAN device.

Finish with the running LAN preview and these observed results. If synchronization is blocked, identify the preserved checkout and the exact blocker rather than claiming it is up to date.
