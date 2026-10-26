# Recovery and Verification

How to undo a bad change and how to prove a good one actually applied.

---

## Back up before destructive work

### Windows

```powershell
# Registry key
reg export "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" "C:\Temp\lsa-backup.reg" /y

# Firewall policy
netsh advfirewall export "C:\Temp\firewall-backup.wfw"

# Local security policy
secedit /export /cfg "C:\Temp\secpol-backup.inf"

# Group Policy (domain)
Backup-GPO -All -Path "C:\Temp\GPO-Backup"

# Installed software and services
Get-CimInstance Win32_Product | Select-Object Name,Version | Export-Csv "C:\Temp\software.csv" -NoTypeInformation
Get-CimInstance Win32_Service | Select-Object Name,StartMode,State,PathName | Export-Csv "C:\Temp\services.csv" -NoTypeInformation
```

### Linux

```bash
sudo cp -a /etc/ssh/sshd_config /root/sshd_config.bak
sudo cp -a /etc/sudoers /root/sudoers.bak
sudo cp -a /etc/pam.d /root/pam.d.bak
sudo cp -a /etc/login.defs /root/login.defs.bak
sudo ufw status verbose > /root/ufw.bak
sudo iptables-save > /root/iptables.bak
sudo nft list ruleset > /root/nftables.bak
```

Keep a root shell open while changing authentication or firewall rules. If you lock yourself out,
the backup is worthless.

---

## Windows recovery

### Deleted a required user

```powershell
New-LocalUser -Name "<user>" -Password (Read-Host -AsSecureString "Password") -PasswordNeverExpires:$false
Add-LocalGroupMember -Group "Users" -Member "<user>"
Add-LocalGroupMember -Group "Remote Desktop Users" -Member "<user>"   # only if required
```

If the account was a domain account, restore it in Active Directory Users and Computers or
recycle it from the AD Recycle Bin.

### Disabled a required service

```powershell
Set-Service -Name "<service>" -StartupType Automatic
Start-Service -Name "<service>"
Get-Service -Name "<service>" | Select-Object Name,Status,StartType
```

If the service was set to Manual, restore Manual rather than Automatic unless the vendor
documentation says otherwise.

### Removed required software

```powershell
winget install --id <package-id>
```

or reinstall from the original installer. Check whether the README asked for an **update** rather
than a removal.

### Broke a registry value

```powershell
reg import "C:\Temp\lsa-backup.reg"
```

If there is no backup, check the default value in Microsoft documentation. Do not guess security
settings that affect boot or logon.

### Broke Group Policy

```powershell
gpupdate /force
gpresult /h C:\Temp\gpresult.html
```

Restore a GPO backup:

```powershell
Restore-GPO -Name "<GPO name>" -Path "C:\Temp\GPO-Backup"
```

If the GPO was deleted and the recycle bin is enabled, restore it from Group Policy Management.

### Locked out of RDP

From the console or an administrative session:

```powershell
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name fDenyTSConnections -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name UserAuthentication -Value 1
Restart-Service TermService -Force
```

Check that your account is in Remote Desktop Users and that the firewall rule applies to the
current network profile.

### Broke Defender

```powershell
Set-MpPreference -DisableRealtimeMonitoring $false
Set-MpPreference -DisableBehaviorMonitoring $false
Update-MpSignature
Get-MpComputerStatus
```

If Tamper Protection blocks the change, use Windows Security → Virus & threat protection →
Manage settings.

---

## Linux recovery

### Deleted a required user

```bash
sudo useradd -m -s /bin/bash <user>
sudo passwd <user>
sudo usermod -aG <group> <user>
```

Restore the home directory from backup if it existed. Reapply file ownership:

```bash
sudo chown -R <user>:<user> /home/<user>
```

### Disabled a required service

```bash
sudo systemctl unmask <service>
sudo systemctl enable --now <service>
systemctl status <service>
```

### Broke SSH access

From the console:

```bash
sudo cp /root/sshd_config.bak /etc/ssh/sshd_config
sudo sshd -t
sudo systemctl restart ssh
```

If you removed `nullok` and locked out a password-based account, restore the PAM file:

```bash
sudo cp /root/pam.d.bak/common-auth /etc/pam.d/common-auth
```

### Broke sudo

```bash
sudo visudo -c
```

If sudoers is invalid, boot into recovery mode or use a root shell and restore the backup:

```bash
cp /root/sudoers.bak /etc/sudoers
chmod 440 /etc/sudoers
visudo -c
```

### Broke the firewall

```bash
sudo ufw disable
sudo ufw reset
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw enable
```

For nftables/iptables, restore the saved ruleset:

```bash
sudo nft -f /root/nftables.bak
sudo iptables-restore < /root/iptables.bak
```

### Broke package sources

Restore the original files:

```bash
sudo cp /root/sources.list.bak /etc/apt/sources.list
sudo apt-get update
```

---

## Why a change did not apply

### Windows

| Symptom | Likely cause | Check |
|---|---|---|
| Password policy unchanged | Domain GPO overrides local policy | `gpresult /h`, edit the winning GPO |
| Audit setting unchanged | Subcategory overrides category | `auditpol /get /subcategory:"<name>"` |
| Service still runs | Startup type not disabled, or dependency restarted it | `Get-Service`, check dependencies |
| Firewall rule ignored | Wrong profile or a block rule with higher precedence | `Get-NetFirewallRule`, check all profiles |
| Registry value reverts | Policy key managed by GPO | Check `HKLM\SOFTWARE\Policies`, use GPO |
| SMB signing not applied | Server and client both need configuration | `Get-SmbServerConfiguration`, `Get-SmbClientConfiguration` |
| RDP still refuses | NLA, firewall, or user not in Remote Desktop Users | Check all three |

### Linux

| Symptom | Likely cause | Check |
|---|---|---|
| SSH setting ignored | A drop-in file overrides `sshd_config` | `sshd -T`, inspect `/etc/ssh/sshd_config.d/` |
| Password policy ignored | PAM module order or missing `pwquality` | `grep pam_pwquality /etc/pam.d/common-password` |
| Service restarts | Socket activation or a timer | `systemctl status`, `systemctl list-timers` |
| Firewall rule ignored | UFW and nftables/iptables both active | `ufw status`, `nft list ruleset`, `iptables -S` |
| Sudo rule ignored | File not included or syntax error | `visudo -c`, check `#includedir` |
| Package change reverted | Unattended upgrade or pinned version | `apt-cache policy`, check apt logs |

---

## Verification discipline

After every destructive change:

1. Re-read the README requirement.
2. Query the **effective** state, not the dialog.
3. Test the user-facing function: log in, open the share, start the service.
4. Record the result and the undo command.
5. Re-check that no authorized user, service, or application was broken.

If the scenario tracks penalties, report an accidental change immediately. A fast, documented undo
is better than a second change made to hide the first.
