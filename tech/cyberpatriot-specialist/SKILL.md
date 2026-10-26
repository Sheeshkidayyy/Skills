---
name: cyberpatriot-specialist
description: >
  Defensive study companion for CyberPatriot-style incident-response practice. Use it to build
  hardening checklists, work through README-driven scenarios, audit accounts and persistence,
  review Windows, Linux, and Cisco baselines, practise forensics method, and recover from
  accidental changes. General defensive methodology only: it contains no competition answers,
  image-specific findings, scoring data, or round-specific content.
---

# CyberPatriot Specialist — Defensive Study Companion

## Scope and boundaries

This skill teaches **method**, not answers. It is safe to use for study, practice images you are
authorized to use, and general defensive administration.

It deliberately excludes:

- competition answers, fixes, or vulnerability lists tied to a specific image or round;
- scoring reports, point values, or category weights;
- scripts written for a specific competition image;
- any material that is not publicly available.

CyberPatriot competition materials are the intellectual property of the Air & Space Forces
Association and are treated as examination material. Do not publish, share, or retain them outside
your team. When in doubt, use the official public training resources listed at the end of this
file.

## The right mindset

Treat the system as a network that has already been compromised. You are the incident responder:
what did the attacker change, what did they install, and what did they leave behind?

The README is your case file. It defines what is legitimate. Everything else is a candidate
finding until you classify it.

## Rule #1: README before everything

Extract and write down:

1. **Authorized users** — never delete, disable, or modify them.
2. **Required services** — stopping one can break the scenario and cause penalties.
3. **Required applications and shares** — these must stay available and may need updating.
4. **Scenario context** — web server, domain controller, file server, workstation.
5. **Forensics questions** — collect all of them before changing the system.
6. **Special instructions** — accounts to create, passwords to change, settings to apply.
7. **Update requirements** — which applications the scenario expects you to patch.

Keep this list visible. Every destructive action gets checked against it first.

## Suggested workflow

Adapt the timing to your session. The order matters more than the clock.

```text
1. Verify the system clock and timezone.
2. Read the README completely; record users, services, apps, and questions.
3. Answer forensics questions before modifying evidence.
4. Start operating-system updates in the background.
5. Baseline the system: users, admins, services, listeners, persistence, shares, firewall.
6. Fix in priority order: identity -> exposure -> persistence -> policy -> updates -> apps.
7. Verify each group of changes before moving on.
8. Re-read the README and confirm no authorized user or service was broken.
9. Document what changed, why, and how to undo it.
```

## Change safety

- **Snapshot or export first.** Registry keys, GPO objects, firewall policy, `sshd_config`,
  `sudoers`, and package sources are cheap to back up and expensive to reconstruct.
- **Do not delete until you classify.** Disable or rename first; delete only when the README
  confirms the account, service, or file is unauthorized.
- **One change at a time per category.** Batch changes make verification and rollback harder.
- **Check dependencies.** A service may be required by another role even if it looks unused.
- **Record the undo command** next to every destructive change.

## Priority order

1. **Identity and privilege** — unauthorized users, extra administrators, UID 0 accounts,
   password-never-expires, blank passwords, sudo rules.
2. **Remote access and exposure** — RDP, SSH, FTP, Telnet, VNC, WinRM, SMB, web admin interfaces.
3. **Persistence** — Run keys, startup folders, scheduled tasks, services, WMI subscriptions,
   cron, systemd timers, shell profiles, `rc.local`.
4. **Policy** — password and lockout policy, audit policy, security options, event log sizes.
5. **Updates** — operating system and the applications named in the README.
6. **Application settings** — browser privacy, FTP/SMB hardening, web server configuration.
7. **Malware and prohibited content** — Defender exclusions, suspicious binaries, media files,
   plain-text credentials.

## Category checklist

### Accounts

- Inventory local users, administrators, remote-desktop users, and service accounts.
- Check for enabled accounts that are not in the README.
- Check UID 0 on Linux; check duplicate or hidden accounts.
- Check password age, complexity, lockout, and "password never expires".
- Check for blank passwords and console-only logon exceptions.

### Services and listeners

- List running services and listening ports with owning processes.
- For each service, inspect the display name, binary path, logon account, and dependencies.
- Look for binaries outside standard system directories, unusual logon accounts, and names that
  imitate legitimate services.
- Stop and disable only what the README does not require.

### Persistence

- Registry Run/RunOnce keys, startup folders, scheduled tasks, services, WMI subscriptions.
- Linux: system and user crontabs, systemd units and timers, `rc.local`, profile scripts,
  `authorized_keys`.
- Cisco: startup configuration, boot variable, local users, VTY access.

### Policy and auditing

- Password length, age, history, complexity, and lockout.
- Audit subcategories and log sizes.
- Security options: Ctrl+Alt+Del, last-user display, cached logons, WDigest, anonymous
  enumeration, SMB signing, NLA.
- Verify with the effective policy, not just the local dialog.

### Network

- Default-deny inbound; allow only README-required ports.
- Confirm every firewall profile is enabled.
- Disable unused management protocols: Telnet, CDP/LLDP where not needed, SNMP defaults,
  HTTP management.
- Check shares and SMB permissions.

### Forensics

- Preserve evidence: record the question, the command you ran, and the output.
- Work from copies when a tool would modify the file.
- See `references/forensics.md` for method and question patterns.

## Verification

A change is not complete until you confirm the effective state:

- **Windows policy:** `gpresult /h` or `auditpol /get` — the winning GPO may override local policy.
- **Services:** status must be `Stopped` and startup type `Disabled`.
- **Firewall:** all profiles enabled; rules apply to the right profile.
- **Linux:** re-read the config, run `sshd -t`, `visudo -c`, `ufw status verbose`.
- **Cisco:** `show running-config` and `write memory`; verify with `show` commands.

See `references/recovery-and-verification.md` for undo procedures and common reasons a change
does not take effect.

## Reference files

| File | Load for |
|---|---|
| `references/defense-methodology.md` | Attacker-informed triage, attack-path map, first-15-minute inventory |
| `references/windows-baseline.md` | Windows 10/11 and Server account, policy, service, firewall, persistence baseline |
| `references/linux-baseline.md` | Ubuntu/Mint users, sudo, SSH, PAM, updates, firewall, persistence baseline |
| `references/network-baseline.md` | Listener inventory, port decisions, host firewalls, Cisco IOS hardening |
| `references/forensics.md` | Evidence handling and common question patterns |
| `references/recovery-and-verification.md` | Backups, undo procedures, why a change did not apply |

## Official public resources

- CyberPatriot training materials: <https://www.uscyberpatriot.org/competition/training-materials>
- CyberPatriot competition rules: <https://www.uscyberpatriot.org/competition/rules-book>
- Microsoft Security Baselines: <https://learn.microsoft.com/security/benchmark/>
- CIS Benchmarks: <https://www.cisecurity.org/cis-benchmarks>
- Ubuntu security documentation: <https://ubuntu.com/security>
- NIST SP 800-123, Guide to General Server Security: <https://csrc.nist.gov/pubs/sp/800/123/final>

Use official sources for season-specific rules. This skill does not replace them.
