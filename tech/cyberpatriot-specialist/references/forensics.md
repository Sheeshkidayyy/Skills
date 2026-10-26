# Forensics Method

A repeatable way to answer forensic questions on a system you are authorized to examine.

The goal is **evidence**, not speed. Record the question, the command, and the output before you
change anything on the system.

---

## Evidence handling

1. **Read the question twice.** Identify exactly what is being asked: a hash, a filename, a port,
   a username, a timestamp, a decoded string.
2. **Work read-only.** Use `cat`, `strings`, `grep`, `Get-Content`, and hash tools. Avoid opening
   files in editors that may rewrite timestamps.
3. **Copy before analysis** when a tool might modify the file. Keep the original path and hash.
4. **Record the command and output.** Screenshots or notes are acceptable; the exact value is what
   matters.
5. **Do not guess.** If two answers are plausible, show your reasoning and the evidence.

---

## Question patterns

### 1. Hash of a file

Windows:

```powershell
Get-FileHash -Algorithm SHA256 "C:\path\to\file"
Get-FileHash -Algorithm MD5 "C:\path\to\file"
```

Linux:

```bash
sha256sum /path/to/file
md5sum /path/to/file
```

Check the algorithm the question asks for. MD5 and SHA-256 produce different values.

### 2. Hidden message in a file

Work from least destructive to most:

```bash
file /path/to/file
strings -n 6 /path/to/file | less
xxd /path/to/file | head -50
binwalk /path/to/file          # embedded files and archives
exiftool /path/to/file         # metadata, comments, GPS
```

Common encodings:

```bash
base64 -d suspect.txt
echo "<string>" | base64 -d
python3 -c "print(bytes.fromhex('<hex>').decode(errors='replace'))"
```

Windows alternate data streams:

```powershell
Get-Item -Path "C:\path\file" -Stream *
Get-Content -Path "C:\path\file" -Stream "hidden"
```

Extract embedded strings without executing the file. Do not run unknown binaries.

### 3. Port or process used by a backdoor

Windows:

```powershell
Get-NetTCPConnection -State Listen |
  Select-Object LocalAddress,LocalPort,OwningProcess,
    @{N="Process";E={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).ProcessName}}

netstat -ano
```

Linux:

```bash
sudo ss -tulnp
sudo lsof -i -P -n
```

Map the PID to the binary:

```powershell
Get-Process -Id <pid> | Select-Object Id,ProcessName,Path
```

```bash
readlink -f /proc/<pid>/exe
ps -fp <pid>
```

### 4. File metadata

```bash
stat /path/to/file
exiftool /path/to/file
```

```powershell
Get-Item "C:\path\to\file" | Select-Object FullName,Length,CreationTime,LastWriteTime,LastAccessTime
```

Timestamps can be changed by an attacker. Compare multiple artifacts: filesystem time, event log
time, and application logs.

### 5. Browser history and downloads

Windows Chrome/Edge:

```text
C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\History
C:\Users\<user>\AppData\Local\Microsoft\Edge\User Data\Default\History
```

Linux Chrome/Chromium:

```text
~/.config/google-chrome/Default/History
~/.config/chromium/Default/History
```

The History file is SQLite. Copy it first, then query:

```bash
sqlite3 History "select datetime(last_visit_time/1000000-11644473600,'unixepoch'), title, url from urls order by last_visit_time desc limit 20;"
```

Firefox uses `places.sqlite` in the profile directory.

### 6. Logs

Windows:

```powershell
Get-WinEvent -LogName Security -MaxEvents 100
Get-WinEvent -FilterHashtable @{LogName='System'; StartTime=(Get-Date).AddHours(-24)}
```

Linux:

```bash
sudo journalctl -n 200
sudo tail -100 /var/log/auth.log
sudo grep -i "failed\|accepted\|sudo" /var/log/auth.log | tail -50
```

Look for logon failures, new accounts, privilege changes, service installs, and cleared logs.

### 7. Plain-text credentials

```bash
grep -Ril "password\|passwd\|pwd" /home /root /var/www 2>/dev/null | head -50
```

```powershell
Get-ChildItem C:\Users -Recurse -Include *.txt,*.csv,*.ps1,*.bat,*.config `
  -ErrorAction SilentlyContinue |
  Select-String -Pattern "password|passwd|pwd" -List | Select-Object Path
```

### 8. Scheduled tasks and startup entries

```powershell
Get-ScheduledTask | Where-Object { $_.TaskPath -notmatch "\\Microsoft\\" } |
  Select-Object TaskName,TaskPath,State,Author

Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```

```bash
cat /etc/crontab /etc/cron.d/* 2>/dev/null
systemctl list-timers --all
```

### 9. Recently modified files

```bash
find / -xdev -type f -mtime -2 2>/dev/null | head -100
```

```powershell
Get-ChildItem C:\ -Recurse -File -ErrorAction SilentlyContinue |
  Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-2) } |
  Select-Object FullName,LastWriteTime
```

### 10. Deleted files

Check the recycle bin or trash first. On Linux:

```bash
ls -la ~/.local/share/Trash/files
```

Recovery beyond the trash requires a forensic tool and a write blocker. Do not install tools on
the evidence disk if you can avoid it.

---

## Documentation template

```text
Question:
Answer:
Evidence:
  Command:
  Output:
  File hash (if applicable):
Timestamp:
Analyst:
```

Paste the exact value into the answer field. Re-read the question to confirm the requested format
(for example, lowercase hex, full path, or port number only).

---

## Boundaries

- Analyze only systems you are authorized to examine.
- Do not execute suspicious binaries to "see what happens".
- Do not upload evidence to third-party services without approval.
- Do not alter timestamps or delete files until the evidence is recorded.

For competition environments, follow the current rules on retaining and sharing materials.
