# Defense Methodology

Attacker-informed triage for an authorized practice system. Use it to decide **what to inspect
first**, not as a list of fixes for a specific image.

This is a blue-team reference. Do not use it to attack systems outside an environment you are
authorized to administer.

---

## Defensive method

1. **Read the README.** Record authorized users, required services, required applications,
   required shares, forensics questions, and special instructions.
2. **Baseline before changing.** Inventory users, services, listeners, persistence, firewall
   rules, shares, logs, and security settings.
3. **Classify every finding:**
   - required by the README;
   - a normal operating-system dependency;
   - authorized but poorly configured;
   - unauthorized or suspicious;
   - unknown — research before deleting.
4. **Fix in order:** identity abuse, credential exposure, persistence, open remote access,
   application settings, logging/auditing, updates.
5. **Verify after each group:** service health, listener list, firewall state, policy result,
   and the README requirements.

---

## Attack path to defense map

| Attacker goal | What to look for | Defensive action |
|---|---|---|
| Initial access | RDP, SSH, FTP, Telnet, VNC, WinRM, web apps, default accounts, outdated apps | Disable unauthorized services; patch apps; require strong authentication; allow only README ports |
| Persistence | Run keys, startup folders, scheduled tasks, services, WMI, cron, systemd timers, shell profiles, `rc.local` | Remove unauthorized autostarts; disable malicious services; document before deleting |
| Privilege escalation | Extra administrators/sudoers, UID 0 users, setuid binaries, weak service paths, writable application directories | Demote unauthorized accounts; fix ownership and ACLs; remove UID 0 backdoors |
| Credential theft | Plain-text password files, saved browser passwords, SSH keys, WDigest, LSASS exposure, cached logons | Delete or protect secrets; disable WDigest; reduce cached logons where the README allows |
| Discovery | `net view`, shares, ARP, DNS, LDAP, SNMP, CDP/LLDP, route tables | Restrict enumeration; disable CDP/LLDP/SNMP unless required; audit shares |
| Lateral movement | SMB admin shares, WinRM, RDP, SSH trust, weak service accounts, domain-admin sprawl | Restrict firewall; remove broad admin rights; enforce SMB signing; disable unused remote administration |
| Defense evasion | Disabled antivirus, Defender exclusions, cleared logs, audit policy off, hidden files, tampered package repositories | Remove exclusions; enable logging and auditing; restore trusted package sources |
| Command and control | Unknown listeners, outbound beacons, unusual DNS, proxy settings, scheduled download jobs | Identify the owning process; remove persistence; block unauthorized ports; keep required traffic working |

---

## First 15 minutes: inventory

Do this after collecting the README facts and answering forensics questions. It is an inventory,
not a hardening pass.

### Windows

```powershell
# Identity and privilege
Get-LocalUser | Select Name,Enabled,LastLogon,PasswordRequired,PasswordNeverExpires
Get-LocalGroupMember Administrators
Get-LocalGroupMember "Remote Desktop Users" -ErrorAction SilentlyContinue

# Network exposure
Get-NetTCPConnection -State Listen | Sort-Object LocalPort |
  Select-Object LocalAddress,LocalPort,OwningProcess,
    @{N="Process";E={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).ProcessName}}

Get-NetFirewallRule -Enabled True -Direction Inbound |
  Where-Object Action -eq Allow | Select-Object DisplayName,Profile,Action

# Persistence
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -ErrorAction SilentlyContinue
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -ErrorAction SilentlyContinue
Get-ScheduledTask | Where-Object { $_.State -ne "Disabled" -and $_.TaskPath -notmatch "\\Microsoft\\" }
Get-CimInstance Win32_Service | Select-Object Name,DisplayName,StartName,PathName,State

# Defender and logs
Get-MpPreference | Select-Object ExclusionPath,ExclusionProcess,DisableRealtimeMonitoring
auditpol /get /category:*
wevtutil gl Security
```

### Linux

```bash
# Identity and privilege
awk -F: '($7 ~ /sh$/) {print $1 ":" $3 ":" $7}' /etc/passwd
awk -F: '($3==0 && $1!="root") {print "UID0 ACCOUNT: " $1}' /etc/passwd
getent group sudo adm admin wheel 2>/dev/null
grep -R "NOPASSWD\|ALL=(ALL" /etc/sudoers /etc/sudoers.d 2>/dev/null

# Network exposure
ss -tulnp
ip addr
ip route
ufw status verbose 2>/dev/null || true
iptables -S 2>/dev/null | head -100 || true
nft list ruleset 2>/dev/null | head -100 || true

# Persistence
systemctl list-unit-files --type=service --state=enabled
systemctl --type=service --state=running
for u in $(cut -d: -f1 /etc/passwd); do
  crontab -l -u "$u" 2>/dev/null | sed "s/^/[$u] /"
done
cat /etc/crontab /etc/cron.d/* 2>/dev/null
ls -la /etc/systemd/system /etc/init.d /etc/rc.local 2>/dev/null

# Logs and package sources
tail -100 /var/log/auth.log 2>/dev/null || journalctl -n 100
grep -R "^deb " /etc/apt/sources.list /etc/apt/sources.list.d 2>/dev/null
```

### Cisco IOS

```text
show running-config
show ip interface brief
show interfaces status
show vlan brief
show access-lists
show line
show cdp neighbors detail
show lldp neighbors detail
show snmp community
show ip http server status
show port-security
```

---

## Windows priorities

### 1. Remove unauthorized remote access

```powershell
# Disable RDP unless the README requires it.
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name fDenyTSConnections -Value 1

# If RDP is required, require Network Level Authentication.
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name UserAuthentication -Value 1

# Restrict remote management services that are not required.
Get-Service WinRM,RemoteRegistry,Telnet | Select-Object Name,Status,StartType
```

### 2. Remove persistence

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -ErrorAction SilentlyContinue
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -ErrorAction SilentlyContinue
Get-ChildItem "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"
Get-ScheduledTask | Where-Object { $_.TaskPath -notmatch "\\Microsoft\\" } |
  Select-Object TaskName,TaskPath,State,Author
Get-CimInstance Win32_Service |
  Select-Object Name,DisplayName,StartName,PathName,State,StartMode
```

Inspect each non-Microsoft task, service, and Run entry. Check the binary path, signer, and
whether the README expects it.

### 3. Protect credentials

```powershell
# Disable WDigest plain-text credential caching.
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" `
  -Name UseLogonCredential -Value 0

# Reduce cached domain logons where policy allows (0-2).
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" `
  -Name CachedLogonsCount -Value "2"

# Find likely plain-text secrets.
Get-ChildItem C:\Users -Recurse -Include *.txt,*.csv,*.ps1,*.bat -ErrorAction SilentlyContinue |
  Select-String -Pattern "password|passwd|pwd" -List | Select-Object Path
```

### 4. Restore defensive visibility

```powershell
Get-MpPreference | Select-Object ExclusionPath,ExclusionProcess,DisableRealtimeMonitoring
Set-MpPreference -DisableRealtimeMonitoring $false
auditpol /get /category:*
```

Remove Defender exclusions that the README does not require. Re-enable real-time protection,
behavior monitoring, and the relevant attack-surface-reduction rules.

---

## Linux priorities

### 1. Remove privilege backdoors

```bash
# Only root should have UID 0.
awk -F: '($3==0) {print $1 ":" $3}' /etc/passwd

# Sudo rights and passwordless rules.
grep -R "NOPASSWD" /etc/sudoers /etc/sudoers.d 2>/dev/null

# Setuid and setgid binaries outside normal system paths.
find / -xdev -perm -4000 -o -perm -2000 2>/dev/null | sort
```

### 2. Remove persistence

```bash
systemctl list-unit-files --type=service --state=enabled
systemctl list-timers --all
for u in $(cut -d: -f1 /etc/passwd); do
  crontab -l -u "$u" 2>/dev/null | sed "s/^/[$u] /"
done
cat /etc/crontab /etc/cron.d/* 2>/dev/null
grep -R "curl\|wget\|base64" /etc/cron* /etc/systemd/system 2>/dev/null
ls -la /etc/rc.local /etc/profile.d 2>/dev/null
```

### 3. Check package integrity

```bash
grep -R "^deb " /etc/apt/sources.list /etc/apt/sources.list.d 2>/dev/null
apt-get update
dpkg --verify 2>/dev/null | head -50
```

Package sources should point only to official distribution repositories. Unexpected PPAs,
mirrors, or `deb [trusted=yes]` entries deserve investigation.

### 4. Restore logging

```bash
systemctl status rsyslog 2>/dev/null
journalctl --disk-usage
ls -la /var/log/auth.log /var/log/syslog 2>/dev/null
```

---

## Cisco priorities

1. Remove default credentials and unused local users.
2. Restrict VTY lines with an ACL and SSH-only access.
3. Disable CDP/LLDP on untrusted interfaces.
4. Remove default SNMP communities; use SNMPv3 if required.
5. Disable the HTTP/HTTPS management server unless required.
6. Shut down unused ports and place them in an unused VLAN.
7. Enable port security, BPDU Guard, and root guard on access ports.
8. Send logs to a local buffer or syslog server.
9. Save the configuration and verify with `show running-config`.

See `references/network-baseline.md` for commands.

---

## Documentation template

For every change, record:

```text
Time:
Category: account | service | policy | firewall | persistence | update | application
Finding:
Evidence (command + output):
Action:
Undo command:
Verified by:
```

This makes penalties recoverable and prevents two responders from undoing each other's work.
