# Linux Baseline

General hardening baseline for Ubuntu 22.04/24.04 and Linux Mint 21. Commands assume a
Debian-based system and `sudo`.

This is a checklist, not a script. Verify each change against the README and the running service.

---

## 1. Account inventory

```bash
# Users with a login shell
awk -F: '($7 ~ /sh$/) {print $1 ":" $3 ":" $7}' /etc/passwd

# UID 0 accounts (only root should appear)
awk -F: '($3==0) {print $1 ":" $3}' /etc/passwd

# Privileged groups
getent group sudo adm admin wheel 2>/dev/null

# Password state
sudo passwd -S -a

# Sudo rules
sudo grep -R "NOPASSWD\|ALL=(ALL" /etc/sudoers /etc/sudoers.d 2>/dev/null
```

Check for:

- Accounts not listed in the README.
- Extra UID 0 accounts.
- Unexpected members of `sudo`, `adm`, `admin`, or `wheel`.
- `NOPASSWD` rules and wildcard commands.
- Accounts with empty password fields in `/etc/shadow`.
- Service accounts with interactive shells.

Fix a UID 0 backdoor by changing the UID, not just reporting it:

```bash
sudo usermod -u 1100 <user>
sudo groupmod -g 1100 <user>
```

Do not delete an account until you have recorded its groups, home directory, and any running
processes.

---

## 2. Password and account policy

```bash
grep -E "PASS_MAX_DAYS|PASS_MIN_DAYS|PASS_MIN_LEN|PASS_WARN_AGE" /etc/login.defs
```

Suggested baseline, adjusted to the README:

| Setting | Suggested value |
|---|---|
| `PASS_MAX_DAYS` | 90 |
| `PASS_MIN_DAYS` | 7 |
| `PASS_MIN_LEN` | 12 |
| `PASS_WARN_AGE` | 7 |

Install and configure `libpam-pwquality`:

```bash
sudo apt-get install -y libpam-pwquality
```

`/etc/security/pwquality.conf`:

```ini
minlen = 12
minclass = 3
maxrepeat = 3
maxclassrepeat = 4
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
```

Remove `nullok` from PAM so blank passwords are rejected:

```bash
sudo sed -i 's/ nullok//g; s/nullok //g' /etc/pam.d/common-auth /etc/pam.d/common-password
```

Verify PAM syntax before closing your session:

```bash
sudo pam-auth-update --package
```

Keep a root shell open until you have tested a new login.

---

## 3. SSH

```bash
sudo sshd -T | grep -Ei "permitrootlogin|passwordauthentication|pubkeyauthentication|permitemptypasswords|x11forwarding|maxauthtries"
```

Recommended baseline in `/etc/ssh/sshd_config` or a drop-in under `/etc/ssh/sshd_config.d/`:

```text
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no
X11Forwarding no
MaxAuthTries 4
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2
AllowTcpForwarding no
```

Set `PasswordAuthentication no` only when key-based access is already working. Test the config
before restarting:

```bash
sudo sshd -t
sudo systemctl restart ssh
```

Check `~/.ssh/authorized_keys` for unknown keys.

---

## 4. Updates

```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get autoremove -y
```

Enable unattended security updates:

```bash
sudo apt-get install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

Verify:

```bash
systemctl status unattended-upgrades
apt-config dump APT::Periodic::Unattended-Upgrade
```

Check that `/etc/apt/sources.list` and `/etc/apt/sources.list.d/` point only to official
distribution repositories. Investigate unexpected PPAs, mirrors, or `[trusted=yes]` entries.

---

## 5. Firewall

Using UFW:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw status verbose
```

Allow only the ports the README requires. Delete stale rules:

```bash
sudo ufw status numbered
sudo ufw delete <number>
```

If the system uses nftables or iptables directly, inspect before adding rules:

```bash
sudo nft list ruleset
sudo iptables -S
```

Do not run UFW and a hand-written iptables ruleset at the same time without checking which one
owns the active policy.

---

## 6. Services

```bash
systemctl list-unit-files --type=service --state=enabled
systemctl --type=service --state=running
ss -tulnp
```

For each listening service, confirm:

- It is required by the README or by a normal system function.
- The binary path is under `/usr`, `/bin`, `/sbin`, or `/opt` — not `/tmp`, `/home`, or `/var/tmp`.
- The unit file is owned by root and not world-writable.
- The service account is appropriate.

Disable only what is not required:

```bash
sudo systemctl stop <service>
sudo systemctl disable <service>
sudo systemctl mask <service>    # prevents manual start as well
```

Check unit file contents before masking:

```bash
systemctl cat <service>
```

Common services to review when unused: `telnet`, `rsh`, `rlogin`, `vsftpd`, `postfix`, `smtp`,
`snmpd`, `avahi-daemon`, `cups`, `rpcbind`, and unnecessary web or database servers.

---

## 7. File permissions

```bash
# Setuid and setgid binaries
sudo find / -xdev \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null | sort

# World-writable files and directories
sudo find / -xdev -type f -perm -0002 2>/dev/null
sudo find / -xdev -type d -perm -0002 2>/dev/null

# Files owned by nobody
sudo find / -xdev -nouser -o -nogroup 2>/dev/null | head -50
```

Investigate unusual setuid binaries. Standard ones include `sudo`, `su`, `passwd`, `mount`,
`umount`, `ping`, `chsh`, `gpasswd`, `newgrp`, and `pkexec`.

Check home directory permissions:

```bash
ls -ld /home/*
```

Home directories should not be world-readable or world-writable.

---

## 8. Persistence review

```bash
# System and user cron
cat /etc/crontab /etc/cron.d/* 2>/dev/null
for u in $(cut -d: -f1 /etc/passwd); do
  crontab -l -u "$u" 2>/dev/null | sed "s/^/[$u] /"
done

# Systemd units and timers
systemctl list-timers --all
systemctl list-unit-files --type=service --state=enabled
ls -la /etc/systemd/system

# Legacy and profile autostarts
cat /etc/rc.local 2>/dev/null
ls -la /etc/profile.d
grep -R "curl\|wget\|base64\|nc " /etc/cron* /etc/profile.d /etc/systemd/system 2>/dev/null

# SSH keys
find /root /home -name authorized_keys -exec ls -l {} \; 2>/dev/null
```

Also check:

- `/etc/init.d/` for short or unusual scripts.
- `~/.bashrc`, `~/.bash_profile`, `~/.profile` for unexpected commands.
- `/etc/ld.so.preload` for injected libraries.
- Environment variables in `/etc/environment`.

---

## 9. Logging and auditing

```bash
systemctl status rsyslog 2>/dev/null
journalctl --disk-usage
ls -la /var/log/auth.log /var/log/syslog 2>/dev/null
```

Install auditd when the scenario requires detailed process and file auditing:

```bash
sudo apt-get install -y auditd
sudo systemctl enable --now auditd
```

Verify the service is running and logs are being written.

---

## 10. Prohibited content

Use the README's list. A general sweep:

```bash
find /home /root /tmp -type f \
  \( -iname "*.mp3" -o -iname "*.mp4" -o -iname "*.mov" -o -iname "*.avi" \
     -o -iname "*.flac" -o -iname "*.ogg" -o -iname "*.wav" \) 2>/dev/null
```

Also look for plain-text credential files:

```bash
grep -Ril "password\|passwd\|pwd" /home /root 2>/dev/null | head -50
```

Review before deleting. A file may be required by the README.

---

## 11. Verification

```bash
# Accounts and policy
sudo passwd -S -a
sudo visudo -c
grep -E "PASS_MAX_DAYS|PASS_MIN_DAYS|PASS_MIN_LEN" /etc/login.defs

# SSH
sudo sshd -t
sudo sshd -T | grep -Ei "permitrootlogin|passwordauthentication"

# Firewall and services
sudo ufw status verbose
ss -tulnp

# Package integrity
sudo apt-get update
sudo dpkg --verify 2>/dev/null | head -50
```

Re-check the README after each group of changes. Do not disable a service that the scenario
requires.
