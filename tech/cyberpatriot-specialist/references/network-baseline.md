# Network Baseline

Listener inventory, port decisions, host firewalls, and Cisco IOS hardening for an authorized
practice environment.

---

## Core rule

Start from **default deny inbound**. Allow only the ports the README requires, plus the traffic
needed for normal operating-system function and updates. Every open port should have a named
owner and a reason.

---

## Listener inventory

### Windows

```powershell
# TCP listeners with owning process
Get-NetTCPConnection -State Listen | Sort-Object LocalPort |
  Select-Object LocalAddress,LocalPort,OwningProcess,
    @{N="Process";E={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).ProcessName}}

# UDP endpoints
Get-NetUDPEndpoint | Sort-Object LocalPort |
  Select-Object LocalAddress,LocalPort,OwningProcess

# Services and binary paths
Get-CimInstance Win32_Service |
  Select-Object Name,DisplayName,State,StartName,PathName
```

### Linux

```bash
sudo ss -tulnp
sudo lsof -i -P -n | grep LISTEN
systemctl --type=service --state=running
```

### Cisco IOS

```text
show ip interface brief
show interfaces status
show ip sockets
show control-plane host open-ports
```

For each listener, confirm the process, the binary path, and whether the README expects it.

---

## Port decision matrix

| Port | Protocol | Typical use | Decision |
|---|---|---|---|
| 20/21 | TCP | FTP control/data | Disable unless required; prefer SFTP |
| 22 | TCP | SSH | Allow only from trusted sources |
| 23 | TCP | Telnet | Disable |
| 25 | TCP | SMTP | Disable unless the scenario is a mail server |
| 53 | TCP/UDP | DNS | Allow only on DNS servers |
| 67/68 | UDP | DHCP | Allow only where required |
| 80/443 | TCP | HTTP/HTTPS | Allow only for required web services |
| 88 | TCP/UDP | Kerberos | Domain controllers only |
| 110/143 | TCP | POP3/IMAP | Disable unless required |
| 135 | TCP | RPC endpoint mapper | Restrict; never expose broadly |
| 137-139 | TCP/UDP | NetBIOS | Disable unless a legacy application requires it |
| 161/162 | UDP | SNMP | Disable or restrict to management hosts |
| 389/636 | TCP | LDAP/LDAPS | Domain controllers only |
| 445 | TCP | SMB | Restrict to required hosts; enforce signing |
| 3389 | TCP | RDP | Disable unless required; require NLA and restrict source |
| 5985/5986 | TCP | WinRM | Disable unless required |
| 3306 | TCP | MySQL | Local or required hosts only |
| 5432 | TCP | PostgreSQL | Local or required hosts only |
| 8080/8443 | TCP | Alternate web/admin | Verify the application; restrict management access |

This table is a starting point. The README overrides it.

---

## Windows Defender Firewall

### Backup and inspect

```powershell
netsh advfirewall export "C:\Temp\firewall-backup.wfw"

Get-NetFirewallProfile |
  Select-Object Name,Enabled,DefaultInboundAction,DefaultOutboundAction,LogAllowed,LogBlocked

Get-NetFirewallRule -Enabled True -Direction Inbound |
  Where-Object Action -eq Allow |
  Select-Object DisplayName,Profile,Program,LocalPort,RemoteAddress
```

### Baseline

```powershell
Set-NetFirewallProfile -Profile Domain,Private,Public `
  -Enabled True -DefaultInboundAction Block -DefaultOutboundAction Allow

Set-NetFirewallProfile -Profile Domain,Private,Public `
  -LogBlocked True -LogAllowed False
```

Enable logging only where it is useful; verbose allow logging can fill the disk.

### Allow only required services

```powershell
New-NetFirewallRule -DisplayName "Allow HTTPS" -Direction Inbound -Protocol TCP `
  -LocalPort 443 -Action Allow -Profile Domain,Private

New-NetFirewallRule -DisplayName "Allow RDP from management subnet" -Direction Inbound `
  -Protocol TCP -LocalPort 3389 -RemoteAddress 10.0.0.0/24 -Action Allow
```

### Find overbroad rules

```powershell
Get-NetFirewallRule -Enabled True -Direction Inbound |
  Where-Object {
    $_.Action -eq "Allow" -and
    ($_.RemoteAddress -eq "Any" -or $_.LocalPort -eq "Any")
  } | Select-Object DisplayName,Profile,Program,LocalPort,RemoteAddress
```

Disable a rule instead of deleting it when you may need to restore it:

```powershell
Disable-NetFirewallRule -DisplayName "<rule>"
```

---

## Linux firewall

### UFW

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 10.0.0.0/24 to any port 22 proto tcp
sudo ufw enable
sudo ufw status verbose
```

Delete stale rules by number:

```bash
sudo ufw status numbered
sudo ufw delete <number>
```

### nftables / iptables

Inspect first:

```bash
sudo nft list ruleset
sudo iptables -S
sudo iptables -L -n -v
```

Do not assume UFW owns the active policy. Check:

```bash
sudo systemctl status ufw
sudo systemctl status nftables
```

### Allow required services

```bash
sudo ufw allow from 10.0.0.0/24 to any port 22 proto tcp
sudo ufw allow 443/tcp
sudo ufw deny 23/tcp
```

Pair the firewall with the service configuration. Closing a port does not disable the service,
and disabling a service does not remove an allow rule.

---

## Cisco IOS hardening

### Inventory

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

### 1. Disable CDP and LLDP on untrusted interfaces

```text
conf t
 no cdp run
 no lldp run
end
write memory
```

If discovery is required, enable it only on trusted uplinks:

```text
conf t
 interface GigabitEthernet0/1
  cdp enable
  lldp transmit
  lldp receive
end
```

### 2. Remove default SNMP communities

```text
conf t
 no snmp-server community public RO
 no snmp-server community private RW
 snmp-server community <random-string> RO <management-acl>
end
```

Prefer SNMPv3 with authentication and privacy when SNMP is required.

### 3. Restrict VTY lines

```text
conf t
 ip access-list standard MGMT-HOSTS
  permit 10.0.0.0 0.0.0.255
  deny any log
 exit

 line vty 0 4
  access-class MGMT-HOSTS in
  transport input ssh
  exec-timeout 10 0
  logging synchronous
 exit
end
```

Generate a local user with a strong secret:

```text
conf t
 username admin privilege 15 secret <strong-secret>
 enable secret <strong-secret>
end
```

### 4. Disable the HTTP management server

```text
conf t
 no ip http server
 no ip http secure-server
end
```

### 5. Shut down unused ports

```text
conf t
 interface range GigabitEthernet0/10 - 24
  shutdown
  switchport mode access
  switchport access vlan 999
 exit
end
```

Create the unused VLAN if it does not exist and keep it off all trunks.

### 6. Port security

```text
conf t
 interface GigabitEthernet0/5
  switchport mode access
  switchport port-security
  switchport port-security maximum 2
  switchport port-security mac-address sticky
  switchport port-security violation restrict
 exit
end
```

### 7. Spanning Tree hardening

```text
conf t
 interface range GigabitEthernet0/1 - 24
  spanning-tree portfast
  spanning-tree bpduguard enable
 exit

 interface GigabitEthernet0/1
  spanning-tree guard root
 exit
end
```

Do not enable BPDU Guard on uplinks or trunk ports to other switches.

### 8. Logging

```text
conf t
 service timestamps log datetime msec
 logging buffered 16384
 logging trap informational
 logging host 10.0.0.10
end
```

### 9. Save and verify

```text
write memory
show running-config
show ip interface brief
show access-lists
show line
show port-security
```

---

## Verification checklist

- [ ] Every listener has a named process and a reason.
- [ ] Inbound default is deny on every host firewall profile.
- [ ] Only README-required ports are allowed.
- [ ] Management access is restricted by source address.
- [ ] Telnet, FTP, and unneeded HTTP administration are disabled.
- [ ] SMB and RDP are restricted and hardened.
- [ ] Cisco discovery, SNMP, and HTTP management are disabled or restricted.
- [ ] Unused switch ports are shut down.
- [ ] Logging is enabled and reaching a destination.
- [ ] Configuration is saved and verified.
