# Reusable Agent Skills

Small, focused skills organized by the kind of work they support rather than by a single personal website.

This repository mirrors the local master copy at `~/Documents/AI playground/AI/skills`
(Steam Deck). OpenCode discovers that folder globally through the `skills` entries in
`~/.config/opencode/opencode.json`, so editing a skill locally and syncing it here keeps
the collection recoverable off-machine.

## Collections

| Folder | Skills | What it is |
| ------ | -----: | ---------- |
| `general/` | 6 | Cross-project skills: documentation, GitHub commits, portfolio evidence, CyberPatriot publishing safety. |
| `website/` | 5 | Website release, diagnosis, and feature workflows (Next.js/Payload, NAS Compose, source verification). |
| `tech/` | 1 | Public-safe CyberPatriot defensive study companion. |
| `agents/` | 41 | Matt Pocock engineering suite plus local variants (review, TDD, triage, writing, planning). |
| `codex/` | 16 | Codex-side installs and sheesh.cloud-specific variants. |
| `hermes/` | 120 | Hermes Agent's library, grouped by domain (apple, github, mlops, red-teaming, …). Many need external CLIs or accounts. |

189 `SKILL.md` files in total. Each skill is independently installable from its own folder.

## General

- [`documenter`](general/documenter/) — recovers relevant past chats, preserves decisions and attempts, and records verified solutions and next steps for future AI sessions.
- [`marketplace-offer-evaluator`](general/marketplace-offer-evaluator/) — assesses second-hand listings for fair value, condition evidence, offer guidance, and risks.
- [`portfolio-evidence-curator`](general/portfolio-evidence-curator/) — turns real work into credible portfolio case studies without invented claims.
- [`case-study-humanizer`](general/case-study-humanizer/) — drafts and edits natural case-study prose while preserving the author's voice, facts, and project status.
- [`github-commit-push`](general/github-commit-push/) — prepares focused commits and verifies approved GitHub publication through Git or a connected app.
- [`cyberpatriot-safe-publishing`](general/cyberpatriot-safe-publishing/) — prepares public CyberPatriot learning material while protecting restricted content.

## Website

- [`start-website`](website/start-website/) — starts a LAN development preview, checks live GitHub develop, and reports commit differences from main.
- [`nextjs-payload-feature-work`](website/nextjs-payload-feature-work/) — implements and verifies features in compatible Next.js App Router and Payload CMS sites.
- [`source-release-verification`](website/source-release-verification/) — traces website source changes, CI, and deployment evidence through an approved release.
- [`payload-compose-diagnosis`](website/payload-compose-diagnosis/) — diagnoses Payload, PostgreSQL, Compose, and proxy failures from runtime evidence.
- [`nas-release-and-rollback`](website/nas-release-and-rollback/) — prepares and verifies authorized website releases on Docker Compose-hosted NAS systems, including backup and rollback planning.

## Tech

- [`cyberpatriot-specialist`](tech/cyberpatriot-specialist/) — defensive study companion for CyberPatriot-style incident response: account and persistence auditing, Windows/Linux/Cisco baselines, forensics method, and recovery. General methodology only — no competition answers, image-specific findings, or scoring data.

## Agents and Codex collections

`agents/` is the Matt Pocock engineering suite plus local variants: `code-review`,
`diagnosing-bugs`, `domain-modeling`, `grilling`, `implement`, `tdd`, `triage`,
`wayfinder`, `writing-*`, and the rest of the flow. `codex/` holds Codex installs
(`case-study-humanizer`, `documenter`, `frontend-design`, `github-commit-push`, …)
and the sheesh.cloud-specific variants that override the general copies on ID
conflicts.

## Hermes collection

`hermes/` is the library shipped with Hermes Agent, kept here so a fresh machine
can restore it. It is grouped by domain (`apple/`, `github/`, `mlops/`,
`red-teaming/`, `smart-home/`, …) and includes skills that need external CLIs,
accounts, or local services. Treat it as a vendored library: upstream is the
source of truth, and local edits should be called out in the commit message.

## Provenance and safety

- `agents/` was copied from `~/.agents/skills`; `hermes/` from `~/.hermes/skills`;
  `codex/` from `~/.codex/skills`. The local master copy is canonical on this machine.
- The public `tech/cyberpatriot-specialist` is deliberately the **sanitized**
  variant. The full competition study skill stays local and is not published here:
  it contains image-specific findings and answer-key-derived material that must
  not leave the team.
- No credentials, tokens, or `.env` contents are stored in this repository.
- Review a skill's instructions before using it in a new environment, especially
  where content publication or deployment is involved.
