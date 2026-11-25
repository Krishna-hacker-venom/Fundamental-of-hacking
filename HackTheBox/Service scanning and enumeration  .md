# Basic Tools & Service Scanning — Attacker Perspective

> **Source:** HTB Academy — *Getting Started* (Basic Tools, Service Scanning, Attacking Network Services)
> **Scope:** Authorized lab / CTF / pentest environments only. Never run these techniques against systems you do not own or have written permission to test.

---

## Table of Contents

1. [Methodology Overview](#1-methodology-overview)
2. [Core Tools: SSH, Netcat, Socat, Tmux](#2-core-tools-ssh-netcat-socat-tmux)
3. [Nmap — Service Scanning](#3-nmap--service-scanning)
4. [Nmap Scripting Engine (NSE)](#4-nmap-scripting-engine-nse)
5. [Banner Grabbing](#5-banner-grabbing)
6. [FTP Enumeration](#6-ftp-enumeration)
7. [SMB Enumeration](#7-smb-enumeration)
8. [SNMP Enumeration](#8-snmp-enumeration)
9. [OS Fingerprinting via Version Strings](#9-os-fingerprinting-via-version-strings)
10. [Attack Chain From the Lab Example](#10-attack-chain-from-the-lab-example)
11. [Defender Takeaways (Why These Work)](#11-defender-takeaways-why-these-work)
12. [Cheat Sheet](#12-cheat-sheet)
13. [Practice Checklist](#13-practice-checklist)

---

## 1. Methodology Overview

**Definition — Service:** An application running on a host that performs a function for other users or computers. Hosts dedicated to this are called **servers**.

**Attacker goal:** Find services that are **misconfigured** or **vulnerable**, then coerce them into performing an unintended action (e.g., executing a command of our choosing, disclosing credentials).

| Phase | Question Being Answered | Primary Tools |
|---|---|---|
| 1. Discovery | Which ports are open? | `nmap` |
| 2. Fingerprinting | What service and version is on each port? | `nmap -sV`, `nc`, banner scripts |
| 3. Enumeration | What data/config does each service expose? | `ftp`, `smbclient`, `snmpwalk`, NSE |
| 4. Credential hunting | Are there passwords/keys in exposed data? | Manual review of downloaded files |
| 5. Access | Can found credentials/vulns give a foothold? | `ssh`, Metasploit, reused creds |

**Key principle:** Port numbers are *conventions*, not guarantees. A service on port 80 may not be HTTP. Always verify with version detection.

---

## 2. Core Tools: SSH, Netcat, Socat, Tmux

These are not "pentest tools" by design, but are used daily in every engagement.

### 2.1 SSH (Secure Shell)

| Property | Detail |
|---|---|
| Default port | 22/TCP |
| Model | Client–server (e.g., OpenSSH client → SSH server) |
| Auth methods | Password, or public/private key pair (passwordless) |
| Capabilities | Remote shell, file transfer, port forwarding, proxying/pivoting |

**Attacker relevance**
- Port 22 open usually suggests **Linux/Unix**, but SSH can also run on Windows.
- Port forwarding/proxying lets us reach **internal networks** from a compromised host (pivoting).
- Reused or leaked credentials frequently work on SSH.

```bash
ssh user@10.10.10.10
ssh -L 8080:127.0.0.1:80 user@10.10.10.10     # local port forward
ssh -D 9050 user@10.10.10.10                   # dynamic SOCKS proxy
```

### 2.2 Netcat (`nc` / `ncat` / `netcat`)

**Definition:** A utility for reading/writing to TCP/UDP connections.

| Use Case | Description |
|---|---|
| Banner grabbing | Connect to a port and read what the service announces |
| Shell handling | Catch reverse shells / connect to bind shells |
| File transfer | Move files between machines |
| Port interaction | Manually talk to any listening service |

```bash
netcat 10.10.10.10 22
# SSH-2.0-OpenSSH_8.4p1 Debian-3
```

- Pre-installed on most Linux distributions.
- Windows options: downloadable `nc.exe`, or **PowerCat** (PowerShell implementation).

### 2.3 Socat

**Definition:** A more capable relative of netcat.

| Feature | netcat | socat |
|---|---|---|
| Basic TCP/UDP connect/listen | Yes | Yes |
| Port forwarding | Limited | Yes |
| Serial device connection | No | Yes |
| Upgrade to fully interactive TTY | No | Yes |

**Attacker relevance:** After obtaining RCE, a **standalone socat binary** can be transferred to the target to produce a more **stable reverse shell** than a raw netcat shell.

### 2.4 Tmux (Terminal Multiplexers)

**Definition:** Tools like **tmux** and **Screen** provide multiple windows/panes inside one terminal session.

**Why it matters:** Pentests run many long-lived tasks (listeners, scans, shells). Tmux keeps them organized and persistent if the connection drops.

```bash
tmux new -s pentest
# Ctrl+b  c    new window
# Ctrl+b  %    vertical split
# Ctrl+b  "    horizontal split
# Ctrl+b  d    detach
tmux attach -t pentest
```

---

## 3. Nmap — Service Scanning

**Definition — Nmap (Network Mapper):** The most widely used port scanner and service fingerprinting tool.

### 3.1 Port Fundamentals

| Concept | Detail |
|---|---|
| Valid port range | 1 – 65,535 |
| Well-known (privileged) ports | 1 – 1,023 |
| Port 0 | Reserved; a service binding to port 0 is assigned the next available port above 1,024 (wildcard behavior) |
| Default Nmap scan | TCP, **top 1,000 most common ports** only |

### 3.2 Basic Scan

```bash
nmap 10.129.42.253
```

Result from the lab: ports **21, 22, 80, 139, 445** open. Completed in ~2 seconds.

### 3.3 Understanding Output Columns

| Column | Meaning | Attacker Note |
|---|---|---|
| **PORT** | Port number and protocol | Default scan is TCP; UDP must be requested (`-sU`) |
| **STATE** | `open`, `closed`, `filtered` | `filtered` usually means a firewall is restricting access, possibly only to certain source IPs |
| **SERVICE** | Name mapped from port number | Only a *guess* based on the port; not verified without `-sV` |
| **VERSION** | Product and version (with `-sV`) | Feeds directly into CVE/exploit research |

### 3.4 Advanced Scan

```bash
nmap -sV -sC -p- 10.129.42.253
```

| Flag | Function |
|---|---|
| `-sV` | Version detection; fingerprints service protocol, application name, and version using a database of 1,000+ signatures |
| `-sC` | Runs default NSE scripts for extra information |
| `-p-` | Scans all 65,535 TCP ports |

**Trade-off:** Much slower (233 s vs. 2 s in the lab) because it performs more than a simple TCP handshake. Hidden services on non-standard ports are only found with `-p-`.

### 3.5 Lab Scan Results Breakdown

| Port | Service | Version | Finding |
|---|---|---|---|
| 21/tcp | FTP | vsftpd 3.0.3 | **Anonymous login allowed**; `pub` directory exposed |
| 22/tcp | SSH | OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 | Reveals Ubuntu (Focal 20.04) |
| 80/tcp | HTTP | Apache 2.4.41 (Ubuntu) | Title shows `phpinfo()` page, discloses PHP 7.4.3 |
| 139/tcp | NetBIOS-SSN | Samba smbd 4.6.2 | SMB over NetBIOS |
| 445/tcp | SMB | Samba smbd 4.6.2 | SMB signing enabled but **not required** |

**Extra findings from scripts**
- `ftp-syst`: control and data connections are **plain text**.
- `smb2-security-mode`: message signing **enabled but not required**.
- `nbstat`: NetBIOS name `GS-SVCSCAN`.

### 3.6 Other Useful Scan Options

```bash
nmap -A -p445 <target>               # aggressive: OS detect, version, scripts, traceroute
nmap -sU --top-ports 20 <target>     # UDP scan (needed for SNMP on 161/udp)
nmap -p- --min-rate 5000 <target>    # faster full-port sweep
nmap -oA scan_results <target>       # save in all formats
```

> **Tip:** Run a fast `-p-` sweep first, then a targeted `-sV -sC -p<ports>` on only the open ports to save time.

---

## 4. Nmap Scripting Engine (NSE)

**Definition:** NSE lets Nmap run Lua scripts for enumeration, vulnerability detection, and brute forcing. Scripts live under `/usr/share/nmap/scripts/`.

```bash
locate scripts/citrix
# citrix-brute-xml.nse
# citrix-enum-apps-xml.nse
# citrix-enum-apps.nse
# citrix-enum-servers-xml.nse
# citrix-enum-servers.nse
```

**Syntax**

```bash
nmap --script <script-name> -p<port> <host>
```

| Example | Purpose |
|---|---|
| `nmap --script smb-os-discovery.nse -p445 <host>` | Extract OS, hostname, workgroup from SMB |
| `nmap -sV --script=banner <host>` | Grab service banners |
| `nmap --script "vuln" <host>` | Run vulnerability-category scripts |

**Real-world example:** During an assessment scoped to a large Citrix installation, an auditor can run Citrix-specific NSE scripts to check for the NetScaler vulnerability **CVE-2019-19781**.

**Further study:** HTB module *Network Enumeration with Nmap*.

---

## 5. Banner Grabbing

**Definition:** Reading the identification message a service sends when a connection is opened, to determine the service and version quickly.

### 5.1 Manual (Netcat)

```bash
nc -nv 10.129.42.253 21
# (UNKNOWN) [10.129.42.253] 21 (ftp) open
# 220 (vsFTPd 3.0.3)
```

| Flag | Meaning |
|---|---|
| `-n` | No DNS resolution (faster, quieter) |
| `-v` | Verbose output |

### 5.2 Automated (Nmap)

```bash
nmap -sV --script=banner -p21 10.10.10.0/24
```

Scales the same technique across an entire subnet.

**Attacker use:** A banner such as `vsFTPd 3.0.3` is searched against exploit databases (Exploit-DB, CVE lists, `searchsploit`) to find known weaknesses.

---

## 6. FTP Enumeration

**Definition — FTP (File Transfer Protocol):** Standard file transfer protocol, default port **21**. Traffic (including credentials) is **plain text**.

### 6.1 Why It Is Valuable

- Frequently contains interesting files.
- **Anonymous login** is a common misconfiguration.
- Files may contain credentials.

### 6.2 Detection

```bash
nmap -sC -sV -p21 10.129.42.253
# ftp-anon: Anonymous FTP login allowed (FTP code 230)
# drwxr-xr-x 2 ftp ftp 4096 Dec 19 23:50 pub
```

### 6.3 Interaction

```bash
ftp -p 10.129.42.253
Name: anonymous
Password: (blank / any)

ftp> ls
ftp> cd pub
ftp> ls
ftp> get login.txt
ftp> exit
```

| FTP Command | Function |
|---|---|
| `ls` | List directory |
| `cd <dir>` | Change directory |
| `get <file>` | Download file |
| `put <file>` | Upload file (check write permissions) |
| `-p` (client flag) | Passive mode, helps through firewalls/NAT |

### 6.4 Loot

```bash
cat login.txt
# admin:ftp@dmin123
```

**Result:** Credentials obtained via anonymous access (lab data).

> **Next steps after finding credentials:** try them against SSH, SMB, web admin panels, and any other discovered service (**credential reuse**).

---

## 7. SMB Enumeration

**Definition — SMB (Server Message Block):** Protocol for file/printer sharing and inter-process communication, prevalent on Windows (also implemented on Linux by **Samba**). Ports **139** and **445**.

### 7.1 Why It Is Valuable

| Reason | Detail |
|---|---|
| Large attack surface | Many enumeration and exploitation vectors |
| Lateral/vertical movement | Shares and credentials enable pivoting |
| Sensitive data in shares | Passwords, configs, backups |
| Vulnerable versions | e.g., **EternalBlue** (RCE on legacy Windows) |

### 7.2 OS Discovery

```bash
nmap --script smb-os-discovery.nse -p445 10.10.10.40
```

Lab output:

| Field | Value |
|---|---|
| OS | Windows 7 Professional 7601 Service Pack 1 |
| Computer name | CEO-PC |
| Workgroup | WORKGROUP |

**Interpretation:** Legacy Windows 7 → candidate for **EternalBlue**. Validate with Metasploit modules (covered in a later section).

### 7.3 Aggressive Scan on Samba

```bash
nmap -A -p445 10.129.42.253
```

Findings: Linux kernel host, **Samba 4.6.2**, hostname **GS-SVCSCAN**, SMB signing not required, 2 network hops.

> **Note:** OS guess reliability drops when Nmap cannot find at least one open and one closed port.

### 7.4 Share Enumeration with smbclient

```bash
smbclient -N -L \\\\10.129.42.253
```

| Flag | Meaning |
|---|---|
| `-L` | List shares on the host |
| `-N` | Null session: suppress password prompt |

| Share | Type | Comment | Interest |
|---|---|---|---|
| `print$` | Disk | Printer drivers | Default |
| `users` | Disk | (none) | **Non-default, investigate** |
| `IPC$` | IPC | IPC service | Default |

### 7.5 Access Attempts

**Guest / anonymous:**

```bash
smbclient \\\\10.129.42.253\\users
smb: \> ls
NT_STATUS_ACCESS_DENIED listing \*
```

Guest access is not permitted.

**With credentials (`bob:Welcome1`):**

```bash
smbclient -U bob \\\\10.129.42.253\\users
smb: \> ls
smb: \> cd bob
smb: \bob\> ls
smb: \bob\> get passwords.txt
```

**Result:** `passwords.txt` downloaded from the user's folder.

| smbclient Command | Function |
|---|---|
| `ls` | List files |
| `cd` | Change directory |
| `get <file>` | Download |
| `put <file>` | Upload |
| `exit` | Disconnect |

### 7.6 Additional SMB Tools (Beyond the Module)

| Tool | Use |
|---|---|
| `smbmap` | Share permissions overview |
| `enum4linux` / `enum4linux-ng` | Users, groups, shares, policies |
| `crackmapexec` / `netexec` | Credential spraying and enumeration |
| `rpcclient` | RPC-based user/group enumeration |

---

## 8. SNMP Enumeration

**Definition — SNMP (Simple Network Management Protocol):** Protocol used to collect information and statistics from routers and devices. Typically **161/UDP**.

### 8.1 Versions and Security

| Version | Authentication | Encryption |
|---|---|---|
| v1 | Plaintext **community string** | None |
| v2c | Plaintext **community string** | None |
| v3 | Yes | Yes |

**Default community strings:** `public` (read) and `private` (write) are often left unchanged.

### 8.2 Information Obtainable

- System name and description (OS, kernel build)
- Running **process parameters** (may leak credentials passed on command line)
- Routing information
- Services bound to additional interfaces
- Installed software versions

**Impact:** Leaked credentials can be reused on externally accessible services (password reuse is common in enterprises).

### 8.3 Queries

```bash
snmpwalk -v 2c -c public 10.129.42.253 1.3.6.1.2.1.1.5.0
# iso.3.6.1.2.1.1.5.0 = STRING: "gs-svcscan"

snmpwalk -v 2c -c private 10.129.42.253
# Timeout: No Response
```

`public` worked, `private` did not, so the read community is valid.

### 8.4 Brute Forcing Community Strings

```bash
onesixtyone -c dict.txt 10.129.42.254
# [public] Linux gs-svcscan 5.4.0-66-generic #74-Ubuntu SMP ... x86_64
```

`onesixtyone` quickly tests many community strings from a wordlist.

---

## 9. OS Fingerprinting via Version Strings

**Technique:** Convert an application's reported version to its distribution package format, then search it online.

| Step | Value |
|---|---|
| Reported SSH version | `OpenSSH 8.2p1 Ubuntu 4ubuntu0.1` |
| Ubuntu package format | `1:7.3p1-1ubuntu0.1` |
| Converted | `1:8.2p1-4ubuntu0.1` |
| Search result | Ubuntu **Focal Fossa 20.04** |
| Release date | April 23, 2020 |

**Other indicators**

| Indicator | Suggests |
|---|---|
| Port 3389 (RDP) | Windows |
| Port 22 (SSH) | Linux/Unix (but not certain) |
| `Service Info: OSs: Unix, Linux` | Linux/Unix family |
| Page title `PHP 7.4.3 - phpinfo()` | PHP version disclosure |

> **Limitation:** This is **not fully reliable**. Newer packages can be installed on an older OS, so confirm with additional evidence.

---

## 10. Attack Chain From the Lab Example

```
Nmap full scan
   ├── 21/FTP  → anonymous login → pub/login.txt → admin:ftp@dmin123
   ├── 22/SSH  → OpenSSH 8.2p1 → Ubuntu 20.04 identified
   ├── 80/HTTP → phpinfo() exposed → PHP 7.4.3 disclosed
   ├── 139,445/SMB → share "users" → bob:Welcome1 → passwords.txt
   └── 161/SNMP → community "public" → system info leaked
```

| Step | Action | Outcome |
|---|---|---|
| 1 | Full port + version scan | Service and OS map |
| 2 | Anonymous FTP | First credential set |
| 3 | SMB share listing | Non-default `users` share found |
| 4 | SMB login with known creds | `passwords.txt` retrieved |
| 5 | SNMP `public` | Hostname and kernel build leaked |
| 6 | Credential reuse (next step) | Try creds on SSH / web / other services |

---

## 11. Defender Takeaways (Why These Work)

| Weakness Exploited | Mitigation |
|---|---|
| Anonymous FTP enabled | Disable anonymous login; use SFTP/FTPS |
| Plaintext protocols (FTP, SNMP v1/v2c) | Use encrypted alternatives; SNMPv3 |
| Default SNMP community strings | Change or disable; restrict by source IP |
| SMB signing not required | Enforce SMB signing; disable SMBv1 |
| Credentials stored in files/shares | Secrets management; remove plaintext password files |
| Verbose banners and `phpinfo()` pages | Suppress version disclosure; delete diagnostic pages |
| Legacy OS (Windows 7) | Patch or retire (EternalBlue exposure) |
| Password reuse | Unique credentials; MFA; credential rotation |

---

## 12. Cheat Sheet

| Goal | Command |
|---|---|
| Quick scan | `nmap <ip>` |
| Full TCP scan | `nmap -p- <ip>` |
| Version + default scripts | `nmap -sV -sC -p<ports> <ip>` |
| Aggressive scan | `nmap -A <ip>` |
| UDP top ports | `nmap -sU --top-ports 20 <ip>` |
| Run one script | `nmap --script <name> -p<port> <ip>` |
| Banner grab | `nc -nv <ip> <port>` |
| Anonymous FTP | `ftp -p <ip>` → `anonymous` |
| List SMB shares | `smbclient -N -L \\\\<ip>` |
| Connect SMB share | `smbclient -U <user> \\\\<ip>\\<share>` |
| SNMP walk | `snmpwalk -v 2c -c public <ip>` |
| SNMP brute force | `onesixtyone -c dict.txt <ip>` |

---

## 13. Practice Checklist

- [ ] Run default scan and a full `-p-` scan; compare timing and results
- [ ] Identify a non-standard-port service using `-sV`
- [ ] Grab banners with both `nc` and Nmap's `banner` script
- [ ] Locate an anonymous FTP server and retrieve files
- [ ] Enumerate SMB shares with and without credentials
- [ ] Query SNMP with `public` and brute force community strings
- [ ] Write a short report: finding, evidence, impact, remediation
- [ ] Reproduce the chain on an HTB / TryHackMe machine and publish a writeup

---

## References

- HTB Academy: *Getting Started* — Service Scanning
- HTB Academy: *Network Enumeration with Nmap*
- Nmap documentation: https://nmap.org/book/
- CVE-2019-19781 (Citrix NetScaler)
- MS17-010 / EternalBlue

> **Disclaimer:** Notes are for education and authorized security testing only.
