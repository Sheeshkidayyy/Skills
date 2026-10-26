# Windows Baseline

General hardening baseline for Windows 10/11 and Windows Server 2019/2022. Apply settings in the
context the README defines: domain policy on a domain controller, local policy on a workgroup
machine.

This is a checklist, not a script. Verify each setting after you change it.

---

## 1. Account inventory

```powershell
Get-LocalUser | Format-Table Name,Enabled,LastLogon,PasswordRequired,PasswordNeverExpires
Get-LocalGroupMember Administrators
Get-LocalGroupMember "Remote Desktop Users" -ErrorAction SilentlyContinue
Get-LocalGroupMember "Backup Operators" -ErrorAction SilentlyContinue
net accounts
```

Check:

- Accounts that are enabled but not authorized by the README.
- Extra members of Administrators, Remote Desktop Users, Backup Operators, and Hyper-V
  Administrators.
- `PasswordNeverExpires` on accounts that should expire.
- Accounts with `PasswordRequired = False`.
- The built-in Administrator and Guest state.
- Service accounts with interactive logon rights.

Use `Disable-LocalUser` before `Remove-LocalUser` when the README is ambiguous. Record group
membership before removing an account.

---

## 2. Password and lockout policy

```powershell
net accounts
```

Recommended baseline, adjusted to the README:

| Setting | Suggested value |
|---|---|
| Minimum password length | 12 or more |
| Maximum password age | 60-90 days |
| Minimum password age | 1 day |
| Password history | 24 or more |
| Lockout threshold | 5 invalid attempts |
| Lockout duration | 15-30 minutes |
| Reset lockout counter after | 15-30 minutes |
| Complexity | Enabled |
| Reversible encryption | Disabled |

On a domain controller, set these in the Default Domain Policy, not with `net accounts`. Local
policy is ignored once domain policy applies.

Verify the effective source:

```powershell
gpresult /h C:\Temp\gpresult.html
```

---

## 3. Local security options

Common settings worth reviewing. Paths are under `secpol.msc` → Local Policies → Security Options
or the equivalent Group Policy path.

| Setting | Recommended state |
|---|---|
| Interactive logon: Require Ctrl+Alt+Del | Enabled |
| Interactive logon: Do not display last user name | Enabled |
| Interactive logon: Machine inactivity limit | 900 seconds or less |
| Network security: LAN Manager authentication level | Send NTLMv2 only; refuse LM and NTLM |
| Network security: Do not store LAN Manager hash | Enabled |
| Network security: Minimum session security for NTLM | Require 128-bit encryption |
| Network access: Do not allow anonymous enumeration of SAM accounts | Enabled |
| Network access: Do not allow anonymous enumeration of shares | Enabled |
| Network access: Restrict anonymous access to named pipes and shares | Enabled |
| Accounts: Limit local account use of blank passwords to console logon only | Enabled |
| Accounts: Block Microsoft accounts | Enabled where compatible |
| Shutdown: Allow system to be shut down without having to log on | Disabled |
| User Account Control: Run all administrators in Admin Approval Mode | Enabled |
| Device Guard / Credential Guard | Enabled where supported |

Disable autologon:

```powershell
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name AutoAdminLogon -Value 0
```

Disable WDigest credential caching:

```powershell
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest" `
  -Name UseLogonCredential -Value 0
```

Reduce cached logons where policy allows:

```powershell
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" `
  -Name CachedLogonsCount -Value "2"
```

Test policy changes on a domain controller through Group Policy; direct registry edits under
`HKLM\SOFTWARE\Policies` can be overwritten at the next policy refresh.

---

## 4. Audit policy

Use advanced audit subcategories, not the legacy category switches.

```powershell
auditpol /get /category:*
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Security Group Management" /success:enable /failure:enable
auditpol /set /subcategory:"Process Creation" /success:enable
auditpol /set /subcategory:"File Share" /success:enable /failure:enable
auditpol /set /subcategory:"Audit Policy Change" /success:enable /failure:enable
```

Subcategory settings override category settings. If a change does not appear to apply, check which
GPO wins and whether the subcategory is configured.

---

## 5. Event logs

```powershell
wevtutil gl Security
wevtutil gl System
wevtutil gl Application
```

Increase log sizes based on available disk. A reasonable baseline is 128-256 MB for Security and
32-64 MB for System and Application, or the maximum the image can support.

```powershell
wevtutil sl Security /ms:268435456
wevtutil sl System /ms:67108864
wevtutil sl Application /ms:67108864
```

Values are bytes. Verify with `wevtutil gl` after the change.

---

## 6. Services

Inventory:

```powershell
Get-CimInstance Win32_Service |
  Select-Object Name,DisplayName,State,StartMode,StartName,PathName |
  Sort-Object State,Name
```

For each non-Microsoft service, inspect:

- **Binary path** — should be under `System32`, `Program Files`, or a legitimate application
  directory. Flag paths in `Temp`, `AppData`, `ProgramData`, or user-writable locations.
- **Logon account** — `LocalSystem`, `LocalService`, or `NetworkService` are normal; an unexpected
  user account is suspicious.
- **Dependencies** — a service may be required by a role even if it looks unrelated.
- **Display name** — names that imitate Windows components are a common disguise.

Disable only services the README does not require:

```powershell
Stop-Service -Name "<service>" -Force
Set-Service -Name "<service>" -StartupType Disabled
```

Common high-risk services to review when not required: Telnet, Remote Registry, FTP Publishing,
SMTP, SNMP, SSDP Discovery, UPnP Device Host, Windows Remote Management, and unnecessary web
server roles.

---

## 7. Firewall

```powershell
Get-NetFirewallProfile | Select-Object Name,Enabled,DefaultInboundAction,DefaultOutboundAction
Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled True -DefaultInboundAction Block
```

Review inbound allow rules:

```powershell
Get-NetFirewallRule -Enabled True -Direction Inbound |
  Where-Object Action -eq Allow |
  Select-Object DisplayName,Profile,Program,LocalPort
```

Keep only rules required by the README and normal operating-system function. Watch for rules with
`Program = Any` and broad port ranges.

Export the policy before bulk changes:

```powershell
netsh advfirewall export "C:\Temp\firewall-backup.wfw"
```

---

## 8. Updates

```powershell
Get-WindowsUpdate -Install -AcceptAll -IgnoreReboot   # PSWindowsUpdate module
```

or use Settings → Windows Update. For servers, check the update history and pending reboots.
Update the applications named in the README; operating-system updates alone are not enough.

---

## 9. Microsoft Defender

```powershell
Get-MpComputerStatus
Get-MpPreference | Select-Object ExclusionPath,ExclusionProcess,DisableRealtimeMonitoring
```

Check for:

- Exclusion paths or processes added by malware.
- Real-time protection disabled.
- Tamper protection disabled.
- Signature age.
- ASR rules in audit-only mode when they should be enforced.

Remove unjustified exclusions:

```powershell
Remove-MpPreference -ExclusionPath "<path>"
Remove-MpPreference -ExclusionProcess "<process>"
```

If Tamper Protection blocks a change, use Windows Security → Virus & threat protection →
Manage settings.

---

## 10. Remote access

```powershell
# RDP state
Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name fDenyTSConnections
Get-NetFirewallRule -DisplayGroup "Remote Desktop"

# NLA
Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name UserAuthentication

# WinRM
Get-Service WinRM
winrm enumerate winrm/config/listener

# Remote assistance
Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Remote Assistance" -Name fAllowToGetHelp
```

Disable remote access that the README does not require. When RDP is required, keep NLA enabled,
restrict the source scope in the firewall, and use a dedicated account.

---

## 11. SMB and shares

```powershell
Get-SmbServerConfiguration | Select-Object EnableSMB1Protocol,RequireSecuritySignature,EnableSecuritySignature
Get-SmbShare
Get-SmbShareAccess -Name "<share>"
```

Recommended baseline:

- SMBv1 disabled.
- SMB signing required on both server and client.
- No `Everyone` full-control permissions.
- No unnecessary administrative or hidden shares.
- Share permissions aligned with NTFS permissions.

---

## 12. Persistence review

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -ErrorAction SilentlyContinue
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -ErrorAction SilentlyContinue
Get-ChildItem "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"
Get-ScheduledTask | Where-Object { $_.TaskPath -notmatch "\\Microsoft\\" }
Get-CimInstance Win32_Service | Where-Object { $_.PathName -notmatch "System32|Program Files" }
```

Also check:

- WMI event subscriptions: `Get-WmiObject -Namespace root\subscription -Class __EventFilter`.
- `Winlogon\Shell` and `Userinit` values.
- Image File Execution Options debuggers.
- Services with unquoted paths containing spaces.
- Browser extensions and startup pages.

---

## 13. Prohibited content

Use the README's list. A general sweep for media and installer files:

```powershell
Get-ChildItem C:\Users -Recurse -Include *.mp3,*.mp4,*.mov,*.avi,*.flac,*.ogg,*.wav `
  -ErrorAction SilentlyContinue

Get-ChildItem C:\Users -Recurse -Include *.exe,*.msi,*.zip,*.7z -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,LastWriteTime
```

Review before deleting. A file may be a required application installer or user data.

---

## 14. Verification

```powershell
gpresult /h C:\Temp\gpresult.html
auditpol /get /category:*
Get-Service | Where-Object { $_.StartType -eq "Disabled" } | Select-Object Name,Status
Get-NetFirewallProfile | Select-Object Name,Enabled
Get-SmbServerConfiguration
Get-MpComputerStatus
```

Check the effective policy, not just the dialog you changed. On a domain, the winning GPO may be
different from the one you edited.
